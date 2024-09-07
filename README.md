# QNAP TS-472A
Boot alternative OSes on the QNAP TS-472A.

## Serial Console Setup

See my [Serial console and TrueNAS on QNAP TS-473A Reddit post](https://www.reddit.com/r/qnap/comments/11lfqgn/serial_console_and_truenas_on_qnap_ts473a/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button).

## OS Selection

If you want to use the TPM2 to unlock LUKS volumes with dracut, you need to use the stable release of Debian (Aug 1, 2024).
There seems to be an inssue on testing.

## TPM2 PCR Info

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
