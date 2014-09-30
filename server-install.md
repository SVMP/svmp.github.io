---
layout: default
title: Installing and Running the SVMP Server
---

## Prerequisites

The SVMP Server can be installed on most Linux distributions. It has been tested on Ubuntu 10.04+, CentOS 6, and RHEL 6. For development, it can also be installed and run on other platforms Nodejs supports, such as OS X and Windows. See the [README](https://github.com/SVMP/svmp-server/blob/master/README.md) for more options.

* Install [Node.js](http://nodejs.org) 0.10.24 or newer as well as the npm tool. See [here](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager) to install from packages.
* Install [MongoDB](http://docs.mongodb.org/manual/installation/) 2.2 or newer

## Overseer

### Installation

1. Install pre-requisites from npm

        $ npm install -g grunt-cli
        $ npm install -g bower

    **Optional Dependencies.** Install libpam development packages if you want to enable PAM-based user authentication with the server.

    On Debian and Ubuntu.

        # apt-get install libpam-dev

    On RHEL and CentOS. Requires EPEL.

        # yum install pam-devel

2. Download the code from git and install dependencies

        $ git clone https://github.com/SVMP/svmp-overseer.git
        $ cd svmp-overseer
        $ npm install

3. Create the configuration file from the provided template

        $ cp config/_config.local.template.yaml config/config-local.yaml

4. [Edit the configuration](./server-config.html#overseer) to suit your environment.

### Running

Start the daemon by running:

    $ node server.js [--config <path/to/config.yaml>]

See the [Puppet](#puppet) module for an example of running the overseer as a
service using [supervisord](http://supervisord.org/).

To create authentication tokens for use with svmp-server instances and the command
line configuration tool:

    $ node create-token.js [-h] <token username>

## Server

### Installation

1. Download the code from git

        $ git clone https://github.com/SVMP/svmp-server.git

2. Change to the root directory of the project and install its dependencies:

        $ cd svmp-server
        $ npm install

3. Create the configuration file from the provided template

        $ cp config/_config.local.template.yaml config/config-local.yaml

4. [Edit the configuration](./server-config.html#server) to suit your environment.

### Running

Starting up the server daemon itself:

    $ node bin/server.js

See the [Puppet](#puppet) module for an example of running svmp-server as a
service using [supervisord](http://supervisord.org/).

## Configuration Tool

### Installation

Install globally using npm:

    $ npm install -g git+https://github.com/SVMP/svmp-server-cli.git

### Running

    $ export overseer_url='https://<overseer_host>:<overseer_port>'
    $ export auth_token='<admin role authorization token>'
    $ svmp-config -h

Alternatively, set the url and token in the YAML file `~/.svmprc` like so:

    overseer_url: '<url>'
    auth_token: '<token>'

## Puppet

A puppet module is available for automated install and configuration of the SVMP server on RHEL and CentOS systems. This module can be found in the [puppet-svmp-server](https://github.com/SVMP/puppet-svmp-server) project.

In addition to installing and configuring the server, the module can also manage the init script, Node.js, and MongoDB as well.
