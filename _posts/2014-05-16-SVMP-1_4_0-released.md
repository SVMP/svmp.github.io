---
layout: default
title: SVMP 1.4.0 Released
---

SVMP 1.4.0 was released on Friday, May 16 2014. This release brings some major changes to the the Android client and SVMP server. The Android client was substantially rewritten to move logged in server connections into a persistent background service. The SVMP server receiving a thorough refactoring for better modularity, maintainability, and scalability. The clients and VM image were updated to a newer upstream release of the WebRTC library, improving video performance and stability in the process. Various other minor features and bug fixes were implemented too. Read on for more details.

## Detailed Change Log

Android Client

* Moved client socket handling into a persistent background service
* Substantial code refactoring to support the background service
* Easier, more automated method for enabling server SSL cert pinning
* Updated to WebRTC 3.52 and ported over useful fixes from AppRTCDemo
* Adding "debouncing" filter for screen rotation updates
* Fixed some bugs with displaying notifications from remote apps
* Support wider variety of ICE server JSON syntaxes from the server
* Cleanly shut down the socket connection to the server in more error cases
* Wipe stored session tokens for a connection if the connection is edited
* Do not attempt session token-based re-authentication if token is known to be expired. Fixes extraneous auth fail on first login.

iOS Client

* Updated WebRTC library to v3.52
* Fixed build for iOS 7.1 SDK
* Added universal build support for both iPhone and iPad
* Remove internet connection test that doesn't work on private networks
* UI tweaks

SVMP Server

* Refactored and better modularized the code
* Switched protocol buffer library used to ProtoBuf.js
* More robust processing of protocol message framing
* No more message fragmentation or desync problems with debug and silly log levels
* Send session token expiration time to clients along with the token value
* Ignore session tokens when using TLS client cert authentication
* Added command line tool option to wipe all session token entries for a user
* Added a 60 second timeout to client socket connections to [work around problem with svmpd](https://github.com/SVMP/svmp-server/commit/1c8e416b1c608b04600ae65daa5ffa135af4eda7)
* Improved config file documentation

VM

* Updated WebRTC to 3.52
* Can now stop and restart video without restarting or crashing svmpd
* Support wider variety of ICE server JSON syntaxes from the server
* Init waits for the /data partition to become available before continuing boot
* Made the VM think it has a fully charged battery
* Made the VM think wifi is connected so apps that check won't give 'no connection' errors
* Fixed fbset so that the refresh rate is set accurately at all screen resolutions
* Added an optional 64-bit kernel config
