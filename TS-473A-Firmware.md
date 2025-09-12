# QNAP TS-473A BIOS/EC Updates

I just booted the PMAP USB to do a QNAP software update, in hopes of it doing some kind of BIOS/EC firmware update.
It looks like it did bump the EC version.
The cool thing is that you can actually just temporarily boot from the PMAP USB disk, connect to the web interface, and have it self upgrade (or using firmware file), without it "installing" the QNAP software to your other disks. Do note that booting the UEFI PMAP target seems to be broken, or non-functional before QNAP install.

* Original QNAP Verision - **QuTS hero h5.0.1.2277**
   * BIOS Vendor - American Megatrends
   * Core Version - 5.14
   * Compliancy - UEFI 2.7; PI 1.6
   * Project Version - Q07DAR15
   * Build Date and Time - 12/20/2021 11:58:14
   * **EC Version - Q07DE012**
   * Bottom Advertised Version: 2.20.1274
* Updated QNAP Version - **QuTS hero h5.2.0.2737 build 20240417 public beta** (2024-04-24)
   * BIOS Vendor - American Megatrends
   * Core Version - 5.14
   * Compliancy - UEFI 2.7; PI 1.6
   * Project Version - Q07DAR15
   * Build Date and Time - 12/20/2021 11:58:14
   * **EC Version - Q07DE014**
   * Bottom Advertised Version: 2.20.1274
* Updated QNAP Version - **QuTS hero h5.2.0.2782 build 20240601 Release Candidate** (2024-06-03)
   * **NO CHANGE FROM PREV VERSION**
   * BIOS Vendor - American Megatrends
   * Core Version - 5.14
   * Compliancy - UEFI 2.7; PI 1.6
   * Project Version - Q07DAR15
   * Build Date and Time - 12/20/2021 11:58:14
   * **EC Version - Q07DE014**
   * Bottom Advertised Version: 2.20.1274

[https://www.qnap.com/en/download?model=ts-473a&category=firmware](https://www.qnap.com/en/download?model=ts-473a&category=firmware)


*References and Tools:*
* https://github.com/max-boehm/qnap-utils
* https://www.securefirmware.de/posts/qnap_decryptor/
