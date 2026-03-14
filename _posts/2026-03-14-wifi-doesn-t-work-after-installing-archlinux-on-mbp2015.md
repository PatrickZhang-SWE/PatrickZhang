---
layout: post
title: wifi and speaker don't work after installing archlinux on MBP2015
date: 2026-03-14 02:20 +0000
categories: archlinux troubleshooting
---

## WIFI

I have an old MBP(2015, 15') and I installed Archlinux on it recently. My main purpose is to use the Omarchy linux. I didn't install the Omarchy by the quick way since I want to maintain the old MacOS. Unfortunately, I didn't successfully install the dual-boot system on my laptop. I erased the whole disk and used it to install Archlinux.

> Before this installation I have already installed Linux Mint. When I try to allocated a new partition, I damaged the booter accidentally. I can not get into the MacOS anymore.

After the installation, I can not use the built-in WIFI. Here is how I fix this problem.

First, check the wifi device:

```shell
~ ❯ lspci -vnn -d 14e4:

03:00.0 Network controller [0280]: Broadcom Inc. and subsidiaries BCM43602 802.11ac Wireless LAN SoC [14e4:43ba] (rev 01)
        Subsystem: Apple Inc. Device [106b:0133]
        Flags: bus master, fast devsel, latency 0, IRQ 82
        Memory at c1400000 (64-bit, non-prefetchable) [size=32K]
        Memory at c1000000 (64-bit, non-prefetchable) [size=4M]
        Capabilities: <access denied>
        Kernel driver in use: brcmfmac
        Kernel modules: brcmfmac
```

Pay attention to **brcmfmac** , this is the module which is need for the wifi to work.

Second, check if this module exists:

```shell
~ ❯ lsmod | grep brcmfmac
brcmfmac_wcc           12288  0
brcmfmac              610304  1 brcmfmac_wcc
brcmutil               20480  1 brcmfmac
mmc_core              303104  1 brcmfmac
cfg80211             1470464  1 brcmfmac
```

If nothing output, then you need to add this module:

```shell
~ ❯ modprobe brcmfmac
```

By doing so we temporarily added this module but after reboot this setting will not persistent. We need to add this module permanently by doing:

```shell
vim /etc/modprobe.d/brcmfmac.conf
add one line: brcmfmac
```

Until now, we haven't done yet. We need to add a kernel parameter: brcmfmac.feature_disable=0x82000.

In order to add this kernel parameter, you need to know the bootloader of your OS. In my case it's **systemd-boot**.

```shell
~ ❯ bootctl status
System:
      Firmware: UEFI 1.10 (Apple 1.10)
 Firmware Arch: x64
   Secure Boot: disabled (unsupported)
  TPM2 Support: no
  Measured UKI: no
  Boot into FW: not supported

Current Boot Loader:
       Product: systemd-boot 259.1-1-arch
     Features: ✓ Boot counting
               ✓ Menu timeout control
               ✓ One-shot menu timeout control
               ✓ Default entry control
...
```

Before you change the kernel parameter you can check what's current used parameters:

```shell
~ ❯ cat /proc/cmdline
```

In my case, I changed this file: **/boot/loader/entries/2026-02-10_16-43-14_linux.conf**, the file name doesn't matter, maybe you have a different file name. You need to append this kernel parameter by append it to options. **Don't forget add one space before this new parameter**.

If you are using a different bootloader, please check [this](https://wiki.archlinux.org/title/Kernel_parameters)

Reboot your MPB, then it should work.

## Sound and Speaker

In my case, the sound and speaker also don't work. After some try and research, I found I need software **pipewire** to get things work.

Here is how to install it and required dependencies:

```shell
sudo pacman -S pipewire pipewire-pulse pipewire-alsa pipewire-jack wireplumber
```

and the setup:

```shell
systemctl --user enable --now pipewire.service pipewire-pulse.service wireplumber.service
```

## Reference

1. [Broadcom wireless](https://wiki.archlinux.org/title/Broadcom_wireless)
2. [kernal parameters](https://wiki.archlinux.org/title/Kernel_parameters)
3. Google Gemini
