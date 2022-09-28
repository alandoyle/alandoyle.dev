---
title: "Swap vs ZSwap vs zRAM"
description: "... for all the times you need Swap (which I think is always)."
author:
email:
date: 2022-09-28T11:38:55+01:00
publishDate: 2022-09-28T11:38:55+01:00
images: []
draft: false
tags: ["ubuntu","swap","tips","performance"]
showToc: true
cover:
    image: "/images/covers/possessed-photography-nuc3NFB_6po-unsplash-cropped.jpg"
    alt: "Image Description"
    relative: false
---

# What is Swap and why do we need it?

Some backstory, Linux divides its RAM into chunks of memory called Pages. Swapping is the process whereby a page of memory is copied to the preconfigured space, usually on the hard disk, called swap space, to free up that page of memory. Freeing up memory pages allows more applications to allocate more memory when needed which can help the application to run smoother and perform better. The bigest caveat is that there can be a performance hit when the swapped out memory is needed again hence there are multiple types of swap depending on the requirements.

On Linux there are 3 popular types of swap: **Swap**, **ZSwap** and **zRAM**.

I'll be discussing these in the context of how to configure on [Ubuntu](https://www.ubuntu.com) 22.04 which is my current preferred Linux distribution. I've tested these instructions on Ubuntu Server and Ubuntu Desktop. The commands used should also work on Ubuntu derivatives like Kubuntu, Xubuntu, etc. but I've not tested them and your mileage may vary.

# So what's the difference between them?

- Swap is the most basic verion and simply consists of a file, files, partition or partitions on a hard drive or SSD marked as swap space which Linux can use to store swapped out memory pages.
- ZSwap operates as a compressed RAM cache and works in conjunction with an uncompressed standard swap device described above.
- zRAM is a compressed swap device that resides solely in RAM and does not require a backing swap device (i.e. uncompressed swap space on a hard drive or SSD).

 Both zRAM and ZSwap use compression to fit more pages into memory thus allowing more memory to be swapped into a smaller space and reduce the number of read/writes to a backing swap device (if configured) which is especially useful if the backing swap device is flash based e.g. an SSD or eMMC drive. That's the theory, however in practice it's the kind of data in the memory page which affects how compressable the memory page is, for example: a memory page containing JPEG data is much less compressable than a memory page containing text data.

# Why do we need any swap nowadays when RAM is cheap and plentiful?

It's common knowledge that using swap space instead of RAM can severely slow down performance, especialy when the swap space is located on a regular hard drive. So, you may ask yourself, since I have plenty of RAM available, wouldn’t it better to disable swap? The short answer is, No. There are actually performance benefits when swap is enabled, even when you have more than enough RAM.

The 2 main benefits are:

1. Swapping out unused Memory pages means there is more actual memory available to be allocated to running applications which can improve responsiveness on desktops and laptops making for a better overall user experience.
1. For server admins swap gives them time to react to low memory issues, i.e. the server slows down but doesn't crash.

The main disadvantage of swap over more RAM is that swap usage becomes a performance bottleneck when the Linux Kernel is pressured to continuously move memory pages in and out of memory and swap space i.e. there isn't enough RAM to hold all the currently active applications in memory so pages of actively used applications are swapped out when another application needs memory and vice versa when the original application requests the active memory back from the swap device. This makes the entire system run very slowly and usually involves a lot of hard drive active. This is commonly called **Thrashing** and is very bad. If this happens the best solution is to either close some of the active applications/services or invest in more RAM to alleviate the problem (if possible).

# How much do you need?

Some recommend no swap (not usually advised) or swap size slightly larger than the total RAM if the swap space is located on a hard drive or SSD. However, this is hardly the case on production servers, and you should instead balance your decision with the effects swap will have on your specific applications. Swap does *not* change the amount of RAM required for a healthy server, or desktop for that matter. It’s designed to be complementary to the performance of healthy systems.

## Check swap space

By default an Ubuntu install will set up a small swap file /swap.img
To check swap space run the following command from a terminal window...

```no-highlight
swapon
```
```no-highlight
NAME      TYPE SIZE USED PRIO
/swap.img file   4G   4M   -2
```

Alternatively run the following command from a terminal window...

```no-highlight
free --mega
```
```no-highlight
               total        used        free      shared  buff/cache   available
Mem:           16617        2317         370          12       13929       13942
Swap:           4294           3        4291
```

Notice that even on this 16Gb system with more than 12Gb available some swap is still used.
 
---

# Basic swap space

![Hard drive](/images/blog/benjamin-lehman-GNyjCePVRs8-unsplash-cropped.jpg)

To add more swap space run the following set of commands...

```no-highlight
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
```
```no-highlight
Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)
no label, UUID=cba13513-5ba0-480a-a0f9-d3e22807dc1a
```

To activate the swap run the following command...

```no-highlight
sudo swapon /swapfile
```

To check if it's enabled run the following command...

```no-highlight
sudo swapon --show
```
```no-highlight
NAME      TYPE SIZE USED PRIO
/swap.img file   4G   4M   -2
/swapfile file   2G   0B   -3
```

## Enable at boot

To enable the swap at the next boot run the following command...

```no-highlight
echo "/swapfile none swap sw 0 0" | sudo tee -a /etc/fstab
```

Now the swap will be active every time the computer boots.

---

# ZSwap

![RAM on Hard drive](/images/blog/heliberto-arias-raNIrcd39jY-unsplash-cropped.jpg)

ZSwap makes very efficient use of swap. It will minimizes Disk I/O by both reducing the number of writes and reads required (data is compressed and held in RAM) and by reducing the bandwidth of these I/O operations as the data is in a compressed form. Also as the compressed data is held in RAM you will not face sudden slowdowns when your system runs out of memory and tries to read/write the swap drive, even if that drive is an SSD.

## Caveats

- ZSwap _needs_ a Swap partition/file. If you don't want/need a Swap partition or file but do want to use a compressed memory swap device then use zRAM.
- With a CPU speed less than 1ghz there will be a performance hit as compressing/decompressing data requires a fast, preferrably a multi-core, CPU.
- ZSwap _should not_ be used with ZSwap. They both provide a compressed cache and would wind up using more system memory than each individually. It makes no sense.

## Check Kernel supports ZSwap

For ZSwap to work it must be built into the Linux Kernel being run.

To check if your OS Kernel comes with ZSwap by running the command below.

```no-highlight
cat /boot/config-`uname -r` | grep -i zswap
```
```no-highlight
CONFIG_ZSWAP=y
# CONFIG_ZSWAP_COMPRESSOR_DEFAULT_DEFLATE is not set
CONFIG_ZSWAP_COMPRESSOR_DEFAULT_LZO=y
# CONFIG_ZSWAP_COMPRESSOR_DEFAULT_842 is not set
# CONFIG_ZSWAP_COMPRESSOR_DEFAULT_LZ4 is not set
# CONFIG_ZSWAP_COMPRESSOR_DEFAULT_LZ4HC is not set
# CONFIG_ZSWAP_COMPRESSOR_DEFAULT_ZSTD is not set
CONFIG_ZSWAP_COMPRESSOR_DEFAULT="lzo"
CONFIG_ZSWAP_ZPOOL_DEFAULT_ZBUD=y
# CONFIG_ZSWAP_ZPOOL_DEFAULT_Z3FOLD is not set
# CONFIG_ZSWAP_ZPOOL_DEFAULT_ZSMALLOC is not set
CONFIG_ZSWAP_ZPOOL_DEFAULT="zbud"
# CONFIG_ZSWAP_DEFAULT_ON is not set
```
If the response is **CONFIG_ZSWAP=y**, you are OK to proceed.

NOTE: Ubuntu Kernels have ZSwap configured, as do the [Xanmod Kernels](https://xanmod.org/).

## Enable ZSwap

To enable ZSwap on the next reboot run the following commands...

```no-highlight
echo GRUB_CMDLINE_LINUX_DEFAULT="$GRUB_CMDLINE_LINUX_DEFAULT zswap.enabled=1 zswap.compressor=lz4" | sudo tee /etc/default/grub.d/enable-zswap.cfg
update-grub2
EOF
```

To make things easier I've created a Zswap Configuration script which can be installed by running the following command...

```no-highlight
sudo bash -c "$(curl -sSfL https://raw.githubusercontent.com/alandoyle/helper-scripts/main/installers/zswap-installer)"
```
This will download and install the new configurable Zswap configuration for Ubuntu. Please feel free to read the full [zswap-installer source](https://github.com/alandoyle/helper-scripts/blob/main/installers/zswap-installer) to make sure you are happy with what the scripts is doing. 

Reboot to enable Zswap.

## Check if ZSwap is enabled

To check if ZSwap has been enabled simply run the following command...

```no-highlight
cat /sys/module/zswap/parameters/enabled
```

If ZSwap is enabled, you should see `Y` in return.

Or if Zswap is enabled via my installation script simply run the following command for all the information required...

```no-highlight
sudo zswap-stats
```
```no-highlight
Zswap Configuration

        same_filled_pages_enabled       Y
        enabled                         Y
        max_pool_percent                20
        compressor                      lzo-rle
        zpool                           zsmalloc
        accept_threshold_percent        90

Zswap Statistics

        same_filled_pages               12
        stored_pages                    623
        pool_total_size                 1597440
        duplicate_entry                 0
        written_back_pages              0
        reject_compress_poor            0
        reject_kmemcache_fail           0
        reject_alloc_fail               147
        reject_reclaim_fail             0
        pool_limit_hit                  0
```

### Edit Zswap Configuration

If Zswap is installed via my installation script above then Zswap can be easily re-configured.
Simply edit **/etc/default/zswap** as root

```no-highlight
#
# Zswap Configuration Options
#
# See https://www.kernel.org/doc/Documentation/vm/zswap.txt
#
################################################################################
#
# Enabled flag 0/1
#
ENABLED=1
#
# Compression method
#   lzo lzo-rle lz4 zstd
#
COMPRESSOR=lz4
#
# If pool utilisation is over this percentage, refuse to take new pages
# (prevents swap thrashing which can kill performance)
#
ACCEPT_THRESHOLD_PERCENT=90
#
# The maximum percentage of RAM that the pool can occupy
#
MAX_POOL_PERCENT=20
#
# Should same-value handling be used to reduce pool storage use where
# possible?
#
SAME_FILLED_PAGES=1
#
# Zswap makes use of zpool for the managing the compressed memory pool.
#   zbud z3fold zsmalloc
#
ZPOOL=zbud
```

Once the settings have been changed it's best to reboot rather than trying to switch settings on-the-fly.

---

# zRAM

![RAM](/images/blog/harrison-broadbent-5tLfQGURzHM-unsplash-cropped.jpg)

zRAM is a fantastic solution to trade some CPU horsepower to gain more RAM. It is also the most configurable.

## Enable zRAM

zRAM is probably the simplest to set up on Ubuntu unlike Swap and ZSwap. Simply run the following command...

```no-highlight
sudo apt install zram-config -y
```

...and zRAM is installed.

NOTE: By default zRAM is set to use half the computer’s actual RAM. This does *NOT* mean that zRAM is using half the available RAM but that it will expand, if necessary, _up to_ this value if the free memory is available.

## Check current zRAM State

Once installed zRAM can be easily checked using the following command...

```no-highlight
zramctl 
```
```no-highlight
NAME       ALGORITHM DISKSIZE DATA COMPR TOTAL STREAMS MOUNTPOINT
/dev/zram0 lz4           3.9G   4K   64B    4K       4 [SWAP]
```

However, this method of enabling is very limited as it is not possible to configure easily. To this end I've created a much more flexible alternative to the `zram-config` package provided by Ubuntu in my [Helper Scripts](https://github.com/alandoyle/helper-scripts) Github repository.

My updated zRAM Configuration script can be installed by running the following command...

```no-highlight
sudo bash -c "$(curl -sSfL https://raw.githubusercontent.com/alandoyle/helper-scripts/main/installers/zram-installer)"
```
This will download and install the new configurable zRAM configuration for Ubuntu. Please feel free to read the full [zram-installer source](https://github.com/alandoyle/helper-scripts/blob/main/installers/zram-installer) to make sure you are happy with what the scripts is doing. 

By default zRAM uses the `lz4` compression algorithm. This algorithm is fast but at the expense of some compression, a good compromise for most systems.

The supported compression algorithms can be seen using the following command...

```no-highlight
cat /sys/block/zram0/comp_algorithm
```
```no-highlight
lzo lzo-rle [lz4] lz4hc 842 zstd
```

The currently used compression algorithm is enclosed in square brackets, in this case **[lz4]**

Configuration is simple
 
As before, once installed zRAM can be easily checked using the following command...

```no-highlight
zramctl 
```
```no-highlight
NAME       ALGORITHM DISKSIZE DATA COMPR TOTAL STREAMS MOUNTPOINT
/dev/zram0 lz4           3.9G   4K   64B    4K       4 [SWAP]
```
### Edit zRAM Configuration

If zRAM is installed via my installation script above then zRAM can be easily re-configured.
Simply edit **/etc/default/zram** as root

```no-highlight
#
# zRAM Configuration Options
#
# See https://www.kernel.org/doc/Documentation/vm/zswap.txt
#
################################################################################
#
# Number of split the zRAM swap into.
#   If set to 0 the CPU Core count will be used to split the zRAM swap.
#
NUM_DEVICES=1
#
# Compression method
#   lzo lzo-rle lz4 lz4hc 842 zstd
#
COMP_MODE=lz4
#
# Fixed memory to allocate
#   If 0 then 1/2 system RAM is used
#   Format sizes like: 100M 250M 1.5G 2G
#
FIXED_MEM=0
```

Once the settings have been changed it's best to reboot rather than trying to switch settings on-the-fly.

 ---

# Let's talk about swappiness...

![Hard disk](/images/blog/art-wall-kittenprint-9Wq1HpghQ4A-unsplash-cropped.jpg)

Swappiness is a Linux kernel property that defines how often the system will use the swap space. It can have a value between 0 and 100. A low value will make the kernel to try to avoid swapping whenever possible, while a higher value will make the kernel to use the swap space more aggressively.

On Ubuntu 22.04, the default swappiness value is set to **60**. You can check the current value by typing the following command...

```no-highlight
cat /proc/sys/vm/swappiness
```
```no-highlight
60
```

While the swappiness value of **60** is OK for most Linux desktop systems, for production servers, you may need to set a lower value.

For example, to set the swappiness value to **10**, run:

```no-highlight
sudo sysctl vm.swappiness=10
```

To make this parameter persistent across reboots, append the following line to the **/etc/sysctl.d/99-swappiness.conf** file:

```no-highlight
echo vm.swappiness=10 | sudo tee /etc/sysctl.d/99-swappiness.conf
```

As with most server tweaks the optimal swappiness value depends entirely on your system workload and how the memory is being used. You should adjust this parameter in small increments to find the optimal value.

# What's my preference?

![Which direction?](/images/blog/jon-tyson-PXB7yEM5LVs-unsplash-cropped.jpg)

The answer is very simple: **ALL** of the above, depending on the scenario.

I use a basic swap file with a low swappiness on my production servers, I use a small basic swap as a backup swapping device for Zswap on all my systems with more than 8Gb RAM, and for systems with less than 8Gb, especially those with flash storage (SSD, eMMC, etc.) I use a single zRAM device @ 50% of system memory.

Hopefully, you'll now have a better appreciation and understanding of the 3 main types of swap on Linux systems and while each has strengths and weakness they are all extremely useful and when anyone suggests removing swap altogether you'll know why having some swap is better than none.