---
layout: default
title: Installing and Running the SVMP Server Components
---

## Overview

The SVMP server components consist of three separate pieces:

1. The SVMP Overseer, which handles authentication and cloud services, and also includes the web console
2. The SVMP Server, which proxies messages from clients to their virtual machines
3. The SVMP Configuration Tool, which provides a command-line interface to manage users and virtual devices

The SVMP server components can be installed on most Linux distributions. It has been tested on Ubuntu 10.04+, CentOS 6, and RHEL 6. For development, it can also be installed and run on other platforms Nodejs supports, such as OS X and Windows. See the [SVMP Overseer](https://github.com/SVMP/svmp-overseer/blob/master/README.md), [SVMP Server](https://github.com/SVMP/svmp-server/blob/master/README.md), and [SVMP Configuration Tool](https://github.com/SVMP/svmp-server-cli/blob/master/README.md) READMEs for more options.

## Prerequisites

* Install [Node.js](http://nodejs.org) 0.10.24 or newer as well as the npm tool. See [here](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager) to install from packages.
* Install [MongoDB](http://docs.mongodb.org/manual/installation/) 2.0 or newer

## Overseer

### Installation

1. Install pre-requisites from the Node Package Manager:

        $ sudo npm install -g grunt-cli
        $ sudo npm install -g bower

    > **Optional Dependencies.** Install libpam development packages if you want to enable PAM-based user authentication with the server.
    >
    > On Debian and Ubuntu:
    >
    >     $ sudo apt-get install libpam-dev
    >
    > On RHEL and CentOS (requires EPEL):
    >
    >     $ sudo yum install pam-devel

2. Download the code from GitHub:

        $ git clone https://github.com/SVMP/svmp-overseer.git

3. Change to the root directory of the project and install its dependencies:

        $ cd svmp-overseer
        $ npm install

4. Create the configuration file from the provided template

        $ cp config/_config.local.template.yaml config/config-local.yaml

5. [Edit the configuration](./server-config.html#overseer) to suit your environment.

### Running

Start the daemon by running:

    $ export NODE_ENV=production
    $ node server.js [--config <path/to/config.yaml>]

See the [Puppet](#puppet) module for an example of running the overseer as a
service using [supervisord](http://supervisord.org/).

### Authenticating

For the Server and Configuration Tool to be able to communicate with the Overseer, they need to authenticate using a pre-generated *administrator role* JSON Web Token. You should generate separate tokens for each component you have, and store them inside the appropriate config files. To find out how to generate one of these tokens, use the following command:

    $ node create-token.js -h

Example:

    $ node create-token.js -a svmp-server-1
    $ node create-token.js -a svmp-config-tool-1

## Server

### Installation

1. Download the code from GitHub:

        $ git clone https://github.com/SVMP/svmp-server.git

2. Change to the root directory of the project and install its dependencies:

        $ cd svmp-server
        $ npm install

3. Create the configuration file from the provided template

        $ cp config/_config.local.template.yaml config/config-local.yaml

4. [Edit the configuration](./server-config.html#server) to suit your environment. Before you do this, you will need to configure the Overseer with its certificates, and you also need to generate a token for the Server to use.

### Running

Starting up the server daemon itself:

    $ export NODE_ENV=production
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
