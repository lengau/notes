This file is for notes on the AMD Ryzen 7040-based Framework 13 and my use of it.

# Setup

I've tried both KDE Neon and Kubuntu on this laptop. For KDE Neon, the [Official Ubuntu notes](https://github.com/FrameworkComputer/linux-docs/blob/main/ubuntu-22.04-amd-fw13.md) were useful. 
For Kubuntu, I'm running 23.10 (Mantic Minotaur) and plan to keep my notes here.

I installed Kubuntu with encrypted LVM.

## TPM-backed disk encryption

I used the standard encrypted LVM install, so my drive contains an unencrypted `/boot` partition and a luks2 encrypted LVM volume. The latter is in `/dev/nvme0n1p3` and is the device I care about.

```
sudo apt install clevis clevis-tpm2 clevis-luks clevis-initramfs tss2
sudo clevis luks bind -d /dev/nvme0n1p3 '{"pcr_bank":"sha256"}'
sudo update-initramfs -u
```

This will request a password on boot, but if it successfully retrieves the password from the TPM it'll continue booting. Otherwise, you can enter your password then. So... it works.

### Non-working version

Currently, using systemd-cryptenroll directly doesn't work. Here are the steps though:

```
sudo apt install tpm2-tools tpm2-initramfs-tool
sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=7+8 /dev/nvme0n1p3
```

Adding PCR 8 was suggested [by /u/gordonmessmer on /r/Fedora](https://www.reddit.com/r/Fedora/comments/szlvwd/comment/hy5ec79/) to include measured boot.

At this point we're stuck with: https://bugs.launchpad.net/ubuntu/+source/cryptsetup/+bug/1980018
However, you can use clevis to get the rest of the way, as shown in [this askubuntu.com answer](https://askubuntu.com/a/1475182):

## Fingerprint reader

Follow the instructions for resetting the firmware on the fingerprint reader: https://github.com/FrameworkComputer/framework-linux-docs/blob/main/13th-gen-fingerprint-reader-firmware.md

Then install fprintd and its pam plugin: 

```
sudo apt install fprintd libpam-fprintd
```

If you want to use the default settings, run `sudo pam-auth-update`. Otherwise, edit `/etc/pam.d/common-auth`. (I actually first ran pam-auth-update to add fingerprint, then moved the fprintd outside of its controlled block so I could change the settings.)

The settings I changed were to set max-tries to 20 and removed the timeout altogether. One max try is a bit overly sensitive (even the pam_fprintd default of 3 is, IMO). Removing the timeout makes KDE's lock screen behave better.
