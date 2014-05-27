---
layout: default
title: SVMP 1.3.1 Released
---

The 1.3.1 release of SVMP fixes some outstanding issues found while deploying the 1.3 version for testing. There are no tags in the git repositories corresponding to this release, although there are branches in the Android client and SVMP server. Most are backported fixes from the development branches of the forthcoming 1.4 release. Changes to the Android VM image went directly into the master or svmp-1.1 branches, and will be picked up in future tagged releases. Major changes include better support for VMs with tablet-sized screen dimentions and orientation, more resilient /data volume mounting, and a pair of critical Android client bugfixes.

## Detailed Change Log

Android Client

* Fixes for running on landscape orientation tablets
* Hide client-side navigation bar buttons
* Remove code that altered client-side audio settings preventing the phone from going into silent or vibrate mode.
* Remove an unnecessary wakelock.
* Updated WebRTC library to v3.52
* More accepting ICE candidate JSON parsing

SVMP Server

* Add a timeout on client sockets to prevent deadlocks if clients do not cleanly close their connection to the server

VM

* Disable VM-side UI animations that slow down rendering
* Improved mount options for the /data partition to reduce chances of it being corrupted by a hard reset
* Let launcher rotate to landscape orientation if using a low resolution VM with a tablet client
* Dead code removal and other housekeeping in smvpd JNI code
