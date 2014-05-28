---
layout: default
title: SVMP 1.2.0 Released
date: 2014-01-29 11:12:35 -0500
---

The 1.2 release of SVMP saw changes mostly to the gateway server component. The Java-based test proxy is retired with this release and will not be receiving any further updates. Accordingly, the Node.js based svmp-server received better support for use in dev and testing environments. Notable improvements include making more options customizable through the configuration file and support in the command line tool for manually mapping VM IP addresses to user accounts for testing and small-scale deployments. There is also now a Puppet module available for installing and configuring the server. The VM now contains an SSH server as a supplement to ADB. See the VM build script for instructions on how to embed an authorized keys file and enable the server.

 including a Puppet module for  

## Detailed Change Log

Android Client

* Fix the application display name string

SVMP Server

* Implemented a maximum length timer for user sessions
* Deprecated the test proxy in favor of the single svmp-server
* Command line tools can now manually associate VM IP addresses to user accounts
* Added a configuration file schema validator that produces good feedback on config errors
* Made certain configuration variables optional with default values appropriate to most installations
* Made many hardcoded settings customizable via the config file
    * Session management TTLs
    * TLS cert and key files
    * Log file location
    * PAM authentication enablement
* Added a Puppet module for configuring and installing the server, complete with an init script

VM

* Enabled openssh server in the VM giving PKI-based logins to a root shell
* Added missing iptables support to the kernel config
* Repaired stock Email application building, also fixing Exchange account-related crashes showing up in logcat
* Removed busybox and the unused volume mounting changes to init that needed it
