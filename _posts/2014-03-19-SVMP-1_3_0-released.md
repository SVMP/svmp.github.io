---
layout: default
title: SVMP 1.3.0 Released
date: 2014-03-19 16:41:04 -0400
---

SVMP now has an iOS client! It's fairly basic at the moment compared to the Android client application, but the two will converge over time. The SVMP server received an overhaul and now includes more logging options and fully automated lifecycle management of the Android virtual machines when used with an OpenStack cloud. Some remaining issues with the VM's virtual SD card were fixed. Another focus of this release is on improving security. PKI certificate based user authentication was added as an option. The Android client now supports server certificate pinning. And we had a security code review performed that resulted in a number of smaller improvements.

## Detailed Change Log

Android Client

* Added PKI certificate based authentication using the Android keystore
* Build-time option to create a server certificate pinning database
* SSL/TLS cipher suite list hardening
* Locked down intent receiver filters
* Removed unused "Domain" field from connection settings
* More graceful error handling if the server cannot connect to the user's VM

iOS Client

* All new client application for iOS including:
    * Multi-touch gesture forwarding
    * Remote desktop video playback
    * Username / password authentication
    * No sensor, notification, or intent forwarding currently

SVMP Server

* Added support for SSL certificate-based user authentication
* Command line configuration utility can now create and associate user data volumes
* Full VM lifecycle management
    * Create VMs on demand and attach data volume when users login
    * Destroy idle VMs after a configurable period of time
    * Optionally configure floating IPs if needed
* Update to the latest upstream pkgcloud library for better Openstack support
* Add detailed debug output logging mode
* SSL/TLS cipher suite list hardening (as much as Node.js 0.10.x allows)
* Stronger session token generation
* Update Puppet module to account for all the configuration file changes

VM

* Fully fixed the emulated SD card
* Optional forwarding of logcat to a UDP syslog server
* Fixed problem with location forwarding
* Locked down intent forwarding filter to just dialer intents
* Locked down framebuffer read permissions
* Fixed intermittent bootloop related to audioflinger and init.rc ordering
