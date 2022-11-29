---
title: "Running PowerTOP on boot"
description: ""
author:
email:
date: 2022-09-29T00:57:34+01:00
publishDate: 2022-09-29T00:57:34+01:00
images: []
draft: false
tags: ["ubuntu","tips","powersaving"]
cover:
    image: "/images/blog/roberto-sorin-ZZ3qxWFZNRg-unsplash-cropped.jpg"
    alt: "Image Description"
    relative: false
---

# Introduction

So in February 2020 I bought a [Teclast F5](/hardware/#teclast-f5) on Amazon to run Ubuntu on. It's a superlight 11.6" 2-in-1 laptop with a FullHD (1920x1080) touchscreen, 8Gb RAM and a 256Gb SSD coupled with an Intel Celeron N4100 1.1GHz (up to 2.4 GHz) Quad Core. It's perfect for carrying around, while being powerful enough to use for most of my mobile computing needs and running Ubuntu get's up to 4 hours from the original 29Wh which suits me fine. I did run Windows 10 and 11 on it too as well as ChromeOS via the [Brunch Framework](https://github.com/sebanc/brunch) and although both ChromeOS and Windows worked well enough the laptop got quite hot, especially Windows, so I returned to Ubuntu and PowerTOP. I'll delve further into Ubuntu on the Teclast F5 in another post but for this post I'll concentrate on PowerTOP.

# What is PowerTOP?

From [Wikipedia](https://en.wikipedia.org/wiki/PowerTOP)... 

> PowerTOP is a software utility designed to measure, explain and minimise a computer's electrical power consumption.

That sums it up quite nicely. 

Nowadays the Linux kernel is much better at power saving but I found in some scenarios, particularly Intel-based laptops, PowerTOP can help lower power consumption and heat. For example, on the Teclast F5 I found that enabling "SATA link power management" in PowerTOP had a very profound and positive effect on the temperature of the 256Gb M.2 SSD installed. It went from being a burning hotspot on the bottom of the laptop to being the same temperature as the rest of the all-metal aluminum chassis. Even under Windows, were unfortunately powersaving and ACPI handling can be better, it was burning hot so very pleased with the outcome. An added bonus is that enabling it incurs almost no noticeable drawbacks (no performance impact, no latency) but it will save you 0.5W to 1W and a _lot_ of heat!

# Installation

Installation on Ubuntu is incredibly simple. Run the following command...

> sudo apt install powertop -y

PowerTOP will now be installed ready to configure and run at startup.

# Enabling PowerTOP on boot

To enable PowerTOP powersaving on boot we simply need to create a simple systemd service file and enable it. Run the following commands to enable PowerTOP on boot...

```bash
cat << EOF | sudo tee /etc/systemd/system/powertop.service
[Unit]
Description=PowerTOP Auto Tune

[Service]
Type=oneshot
Environment="TERM=dumb"
RemainAfterExit=true
ExecStart=/usr/sbin/powertop --auto-tune

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable powertop
sudo systemctl start powertop
```

Alternatively, run my more flexible Powersaver systemd service by running the following command...

```bash
sudo bash -c "$(curl -sSfL https://raw.githubusercontent.com/alandoyle/helper-scripts/main/installers/powersaver-installer)"
```

THis installs a Powersaver service which call **/usr/bin/powertop --auto-tune**, it also includes a file called _/etc/powersaver.d/custom-rules.conf_ in which additional rules can be added depending on the hardware.

For example on my Teclast F5 my _/etc/powersaver.d/custom-rules.conf_ looks like this:

```bash
################################################################################
# Disable audio powersaving to stop loud "popping" noise coming out of speakers
# when the audio hardware goes in and out of sleep.
#
echo 0 > /sys/module/snd_hda_intel/parameters/power_save
echo N > /sys/module/snd_hda_intel/parameters/power_save_controller

################################################################################
#
# NOTE: These options will only take effect after a reboot
#

################################################################################
# Enable iwlwifi powersaving
#
if [ ! -f /etc/modprobe.d/iwlwifi-powersave.conf ] ; then
    cat << EOF | sudo tee /etc/modprobe.d/iwlwifi-powersave.conf
#
# Enable powersaving and disable LED activity for an LED that doesn't exist
#
options iwlwifi power_save=1 led_mode=3
EOF
fi

################################################################################
# Enable video card powersaving
#
if [ ! -f /etc/modprobe.d/i915-powersave.conf ] ; then
    cat << EOF | sudo tee /etc/modprobe.d/i915-powersave.conf
#
# Enable frame buffer compression which lowers power usage.
#
options i915 enable_fbc=1 enable_guc=0
EOF
fi

################################################################################
# Disable sound card powersaving
#
if [ ! -f /etc/modprobe.d/audio_disable_powersave.conf ] ; then
    cat << EOF | sudo tee /etc/modprobe.d/audio_disable_powersave.conf
#
# Disable audio powersaving
#
options snd_hda_intel power_save=0 power_save_controller=N
EOF
fi

```

# Conclusion

In particular, for my Teclast F5 I've found the PowerTOP auto tuned settings combined with my custom rules maximize the battery life of this little laptop. 

Hopefully this information will help someone reduce the power usage of their laptop.
