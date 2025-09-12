# Boot Alternative OSes on QNAP NASes

This document serves as help when running alternative OSes on QNAP NASes.

## QNAP TS-473A
Boot alternative OSes on the QNAP TS-473A.

### Serial Console Setup

<img width="709" height="349" alt="image" src="https://github.com/user-attachments/assets/4ccd775a-9d9c-474e-ad36-1c5d390e2ed2" />

**Serial Console Connector Pinout:**

| 1    | 2     | 3        | 4    | 5     |
| ---- | ----- | -------- | ---- | ----- |
| `TX` | `VCC` | `(3.3V)` | `RX` | `GND` |

*All IO is 3.3V reference.*
*Make sure you connect motherboard TX to the RX on the USB serial adapter and motherboard RX to the TX on the USB serial adapter.*

**Parts:**

* [Motherboard Connector - JPH-2.0-4P-F-M-2-20pc](https://a.co/d/exj0P0y) - [Wayback Machine Archive](https://web.archive.org/web/20220325051225/https://www.amazon.com/dp/B08RHGT3W3)
* [USB Type-C Serial Board](https://a.co/d/2YyofCJ) - [Wayback Machine Archive](https://web.archive.org/web/20250912225229/https://www.amazon.com/dp/B07MLXS877)
* [5 Minute Epoxy](https://a.co/d/5qFtlhH)
* [2x Kingston 500GB Boot SSDs](https://a.co/d/1E6CDp7) - [Wayback Machine Archive](https://web.archive.org/web/20240426041709/https://www.amazon.com/dp/B0BBWJH1P8)
  - These are cheap SSDs with high-er write endurance.
  - We will never even come close to the maxing out the transfer speed of this SSD, since QNAP TS-473A's two onboard PCIe slots only support Gen3 x1.
  - I do not expect to save personal data on these drives.
* [Pinecil V2 Solder Iron](https://www.pine64.org/pinecil/)
* [Solder](https://a.co/d/aOSKhpQ)
* Drill bits up to 1/2" -- Incrementally enlarge the hole from 1/8" to 1/2", to avoid issues with not having a sturdy drilling platform.

**References:**

* [Build a permanent usb-to-serial interface](https://www.youtube.com/shorts/-r4dkya10EQ)
* See my full post about this [Serial console and TrueNAS on QNAP TS-473A Reddit post](https://www.reddit.com/r/qnap/comments/11lfqgn/serial_console_and_truenas_on_qnap_ts473a/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button).
* https://www.cyrius.com/debian/kirkwood/qnap/ts-219/serial/
* [https://www.desgehtfei.net/en/qnap-serial-console-setup-with-arduino/](https://web.archive.org/web/20241114040425/https://www.desgehtfei.net/en/qnap-serial-console-setup-with-arduino/)
* https://www.rigacci.org/wiki/doku.php/doc/appunti/hardware/qnap_ts-120
* https://www.reddit.com/r/qnap/comments/ttm5db/gaining_access_to_the_ts451deu/)

### OS Selection

If you want to use the TPM2 to unlock LUKS volumes with dracut, you need to use the stable (bookworm) release of Debian (Aug 1, 2024).
There seems to be an issue on testing.

### Install from Serial Console

I have since moved on from TrueNAS to bare Debian. Here are some notes about running other distros and how to force them to use the serial console.

* **Temporary Boot Preference** \- You can boot off flash drive or other temporary boot devices, without changing permanent boot order.
   * Just press Del on boot to open the BIOS/FW settings.
   * Navigate right using the arrow keys (in screen) to the `Save & Exit` tab of the settings menu.
   * At the bottom of the page is `Boot Override`, which allows you to select a boot device/mode to temporarily boot from.
* **Debian Live** \- When booting Debian Live, the grub bootloader will present on the serial console at 115200 baud, but you need to modify the grub linux boot command line to force linux to use the serial console, too.
   * At the grub menu, press `e` on the keyboard to modify the first grub live boot option.
   * Add `console=ttyS0,115200n8` to the linux command line and press `Ctrl-X` to boot.
   * If you have enough RAM, you might want to add `toram`, also, which will copy and run the rootfs from RAM.
   * Login as `user` and password `live`.
   * Tested on Debian bookworm.
* **Debian Net Installer** \- Like Debian live, you need to modify the linux command line.
   * At the grub menu, highlight the desired installer mode/entry.
   * Press `e` to edit the entry and add `console=ttyS0,115200n8` to the linux command line.
   * When navigating the installer, you can do `Ctrl-A`, `A`, and then `2` to switch to the shell tab, when using screen. See [https://unix.stackexchange.com/a/533545/162557](https://unix.stackexchange.com/a/533545/162557) for clarification.
   * After you finish the install and reboot, you may need to temporarily edit the grub boot entry and add `console=ttyS0,115200n8`, again. I don't believe the Debian installer configures the grub kernel command line by default.
   * Once you have booted into the fresh Debian install, be sure to manually amend the `/etc/default/grub` file to include `GRUB_CMDLINE_LINUX="console=ttyS0,115200n8"`. Then, run `update-grub`. You may see that GRUB itself was configured to use serial, but the linux kernel itself might not be configured.
   * If you need to pre-configure the LUKS volume, before debian install, check out https://www.blakecarpenter.dev/installing-debian-on-existing-encrypted-lvm/. Note that you will need to manually setup the crypttab, after install.
      * Make sure to select dm-crypt, parted, fdisk, and other necessary extra tools during the Debian installer, if you intend to manually partition and setup LUKS in the installer shell.
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
* Entering BIOS, changing startup beep setting, and saving changes did NOT change PCRs.
* PCRs 1 and 10 changed by simply connecting a USB flash drive during normal boot. I have seen that PCR 1 and 10 can return back to the original values, once the USB mass storage device is disconnected, even if you had booted from the USB device once. I was testing with the PiKVM virtual USB flash drive.
* PCR 14 seems to stay the same between Debian testing live boot and Debian stable (bookworm) normal boot.
