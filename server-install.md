---
layout: default
title: Installing and Running the SVMP Server
---

## Prerequisites

The SVMP Server can be installed on most Linux distributions. It has been tested on Ubuntu 10.04+, CentOS 6, and RHEL 6. For development, it can also be installed and run OS X and Windows. See the [README](https://github.com/SVMP/svmp-server/blob/master/README.md) for more options.

* Install [Node.js](http://nodejs.org) 0.10.24 or newer as well as the npm tool. See [here](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager) to install from packages.
* Install [MongoDB](http://docs.mongodb.org/manual/installation/) 2.2 or newer

* *Optional.* Install libpam development packages if you want to enable PAM-based user authentication with the server.

    On Debian and Ubuntu.
    
        # apt-get install libpam-dev

    On RHEL and CentOS. Requires EPEL.

        # yum install pam-devel

## Installation

1. Download the code from git

        $ git clone https://github.com/SVMP/svmp-server.git

2. Change to the root directory of the project and install its dependencies:

        $ cd svmp-server
        $ npm install

3. Create the configuration file from the provided template

        $ cp config/config-local-example.js config/config-local.js

4. [Edit the configuration](./server-config.html) to suit your environment.

## Running

The command line configuration tool for managing users and VMs:

    $ node bin/cli.js -h
    $ node bin/cli.js <command> <options>

Starting up the server daemon itself:

    $ node bin/server.js

A sysv init script template can be found [here](https://github.com/SVMP/puppet-svmp-server/blob/master/templates/svmp-init.erb). Edit the template to set the `APPLICATION_DIRECTORY` and `LOGFILE` to your liking, then copy it to `/etc/init.d/svmp-server`.

The script depends on the `forever` tool that can be installed by running 

    # npm install -g forever

## Puppet

A puppet module is available for automated install and configuration of the SVMP server on RHEL and CentOS systems. This module can be found in the [puppet-svmp-server](https://github.com/SVMP/puppet-svmp-server) project.

In addition to installing and configuring the server, the module can also manage the init script, Node.js, and MongoDB as well.