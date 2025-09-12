# Boot Alternative OSes on QNAP NASes

This document serves as help when running alternative OSes on QNAP NASes.

## QNAP TS-473A
Boot alternative OSes on the QNAP TS-473A.

### Serial Console Setup

See my [Serial console and TrueNAS on QNAP TS-473A Reddit post](https://www.reddit.com/r/qnap/comments/11lfqgn/serial_console_and_truenas_on_qnap_ts473a/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button).

### OS Selection

If you want to use the TPM2 to unlock LUKS volumes with dracut, you need to use the stable (bookworm) release of Debian (Aug 1, 2024).
There seems to be an inssue on testing.

### Install from Serial Console

I have since moved on from TrueNAS to bare Debian. Here are some notes about running other distros and how to force them to use the serial console.

* **Temporary Boot Preference** \- You can boot off flash drive or other temporary boot devices, without changing permanent boot order.
   * Just press Del on boot to open the BIOS/FW settings.
   * Navigate right using the arrow keys (in screen) to the `Save & Exit` tab of the settings menu.
   * At the bottom of the page is `Boot Override`, which allows you to select a boot device/mode to temporarily boot from.
* **Debian Live** \- When booting Debian Live, the grub bootloader will present on the serial console at 115200 baud, but you need to modify the grub linux boot command line to force linux to use the serila console, too.
   * At the grub menu, press `e` on the keyboard to modify the first grub live boot option.
   * Add `console=ttyS0,115200n8` to the linux command line and press `Ctrl-X` to boot.
   * If you have enough RAM, you might want to add `toram`, also, which will copy and run the rootfs from RAM.
   * Login as `user` and password `live`.
   * Tested on Debian bookworm.
* **Debian Net Installer** \- Like Debian live, add you need to modify the linux command line.
   * At the grub menu, highlight the desired installer mode/entry.
   * Press `e` to edit the entry and add `console=ttyS0,115200n8` to the linux command line.
   * When navigating the installer, you can do `Ctrl-A`, `A`, and then `2` to switch to the shell tab, when using screen. See [https://unix.stackexchange.com/a/533545/162557](https://unix.stackexchange.com/a/533545/162557) for clarification.
   * Be sure to manually ammend the `/etc/default/grub` file to include `GRUB_CMDLINE_LINUX="console=ttyS0,115200n8"`. Then, run `update-grub`. You may see that GRUB itself was configured to use serial, but the linux kernel itself might not be configured.
   * If you need to pre-configure the LUKS volume, before debian install, checkout https://www.blakecarpenter.dev/installing-debian-on-existing-encrypted-lvm/. Note that you will need to manually setup the crypttab, after install.
      * Make sure to select dm-crypt, parted, fdisk, and other neccesary extra tools during the Debian installer, if you intend to manually partiiton and setup LUKS in the installer shell.
   * Tested on Debian bookworm.
* **TrueNAS Scale** \- It has a dedicated console install mode grub item name `Start TrueNAS SCALE Installation (115200 baud)`.

### TPM2 PCR Info

This helps with planning for which PCRs can be used for automatic LUKS disk encryption unlocking on boot.

See the following for typical PCR usages/assignments:
* https://uapi-group.org/specifications/specs/linux_tpm_pcr_registry/
* https://man.archlinux.org/man/systemd-cryptenroll.1#TPM2_PCRs_and_policies

Experiments using the firmwares included in **QuTS hero h5.2.0.2782 build 20240601 Release Candidate (2024-06-03)**:

* PCR 9 and 10 changed after a `sudo dracut --regenerate-all -f`.
* Entering BIOS and discarding setting did NOT change any PCRs.
* Entering BIOS, making no changes, but saving changes anyways did NOT change and PCRs.
* Entering BIOS, changing startup beep setting, and saving changes did NOT PCRs.
* PCRs 1 and 10 changed by simply connecting a USB flash drive during normal boot. I have seen that PCR 1 and 10 can return back to the original values, once the USB mass storage device is disconnect, even if you had booted from the USB device once. I was testing with the PiKVM virtual USB flash drive.
* PCR 14 seems to stay the same between Debian testing live boot and Debian stable (bookworm) normal boot.
