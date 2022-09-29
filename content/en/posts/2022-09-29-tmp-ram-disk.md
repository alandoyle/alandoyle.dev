---
title: "/tmp RAM disk"
description: ""
author: "Alan Doyle"
date: 2022-09-29T00:13:08+01:00
publishDate: 2022-09-29T00:13:08+01:00
images: []
draft: false
tags: ["ubuntu","tips","performance"]
cover:
    image: "/images/blog/harrison-broadbent-5tLfQGURzHM-unsplash-cropped.jpg"
    alt: "Image Description"
    relative: false
---

The purpose of **/tmp** directory on Ubuntu is to provide programs a directory for temporary files. A system reboot automatically deletes the file in /tmp directory. In this short entry, I'll discuss setting up /tmp as a RAM disk and why I do it.

# What /tmp as a RAM disk provides?

## 1. Performance

The main feature of mounting /tmp as a RAM disk is the improved Read and Write performance. It is worth mentioning here that, RAM’s Read and Write speeds are always faster when compared to that of a traditional Hard Drive or Solid State Drive, even when these drives are included in a RAID array. Hence, applications can access data on /tmp quicker than usual.

## 2. Disk Optimization

Temporary files when placed in a /tmp RAM disk frees up disk I/O which allows disk more important disk I/O operations to run quicker. If /tmp was originally mounted on a flash based storage device (SSD, eMMC, etc.) mounting /tmp in RAM will result in fewer Read and Write operations thus reducing the rwear on NAND based disks.

Also, it results in fewer Disk wake-ups.

# What are limitations of /tmp on RAM disk?

## 1. RAM Limit

A /tmp RAM disk is limited to amount of RAM available in the system. If a program uses /tmp directory for huge amounts of data then system will resort back to swap space or slow down the whole system, leaving you with no RAM for other processes. So setting up /tmp as a RAM disk will be greatly influenced by the type of application being run.

# Setup

On Ubuntu 22.04 :

## Check

To check if /tmp RAM disk is already enabled simply run the following command...

```bash
sudo systemctl is-enabled tmp.mount
```

## Enable

To enable the /tmp RAM disk we need to enable the Systemd service

```bash
sudo cp /usr/share/systemd/tmp.mount /etc/systemd/system/
sudo systemctl enable tmp.mount
```

## Disable

To disable the /tmp RAM disk simply run the following command...

```bash
sudo systemctl disable tmp.mount
```

# Personal view

As the vast majority of my systems have Flash based storage (SSD, eMMC, etc.) I prefer to enable the /tmp RAM disk for systems with 8Gb or more RAM and only enable the /tmp RAM disk on systems with less memory if I'm confident that nothing on those systems will place a lot of data in /tmp.
