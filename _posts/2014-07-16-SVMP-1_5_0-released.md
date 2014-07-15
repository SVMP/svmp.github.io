---
layout: default
title: SVMP 1.5.0 Released
---

New in SVMP 1.5.0 is the long awaited single-app mode for the Android client app.
Other major additions include an all-new more flexible build system for the
Android VM image, and an early preview of a new HTML5-based client application.

## Detailed Change Log

Android Client

* [Single-app mode]({% post_url 2014-07-15-single-app-mode %}). Now individual remote
  applications can be mapped to shortcuts on the client's homescreen instead of
  having a whole 2nd "desktop" to manage.
* New icons and graphics
* Sends current default timezone to the VM
* New connections now default to SSL encrypted

Web-based Client

* Early preview of [HTML5-based client application](https://github.com/SVMP/svmp-html-client)
  (requires a WebRTC-enabled browser, such as Chrome or Firefox)

SVMP Server

* Supports in-client password changes
* Pinned all the NPM dependency versions. No more random breakage due to unexpected module changes.
* <code>npm install -g</code> will now create executable links for the server and configuration tool

VM

* All new build system with lots of new features
    * Menu-based configuration
    * New targets for multiple virtual disk and VM platforms types
    * OVF/OVA appliance building for VirtualBox and VMware
    * Disk sizes configurable from BoardConfig.mk (including /data)
    * New "all-in-one" build target combining /system and /data into a single disk image
* Now bootable on hypervisors using [pygrub](http://wiki.xen.org/wiki/PyGrub) and
 [pvgrub](http://wiki.xen.org/wiki/PvGrub) (e.g., Xen PV-mode)
* Kernel updates
    * Updated kernel to linux-stable 3.4.98
    * Revamped the default configuration to add more hypervisor support (i.e. Xen) and remove unnecessary features
* svmpd can now handle multiple connections without blocking (prior connections are kicked)
* VM timezone settable by client
* Fixed notification forwarding again
* Moved /cache partition to the system disk. The contents of /cache are tied more closely to the ROM version and should not be persisted long-term like /data.
