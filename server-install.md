---
layout: default
title: Installing and Running the SVMP Server
---

## Prerequisites

* Install [Node.js](http://nodejs.org) 0.10.24 or newer with npm
* Install [MongoDB](http://docs.mongodb.org/manual/installation/) 2.2 or newer

### Windows only

* Install [Git for Windows](http://msysgit.github.io/)
* Install [Python 2.7.6](https://www.python.org/download/releases/2.7.6/)

Additionally, on Windows, do the following *in order:*

* Install [Visual Studio 2010](http://www.microsoft.com/visualstudio/eng/downloads#d-2010-express)
* Install [Windows SDK 7.1](http://www.microsoft.com/en-us/download/details.aspx?id=8279)
* Install [Visual Studio 2010 SP1](http://www.microsoft.com/en-us/download/details.aspx?id=23691)
* Install [Visual C++ 2010 SP1 Compiler Update for the Windows SDK 7.1](http://www.microsoft.com/en-us/download/details.aspx?id=4422)

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

## Puppet

A puppet module is available for automated install and configuration of the SVMP server on RHEL and CentOS systems. This module can be found in the [puppet-svmp-server](https://github.com/SVMP/puppet-svmp-server) project.

In addition to installing and configuring the server. This module can also install an init script for automatically launching the server at boot. It  also has options for managing the install of Node.js and MongoDB as well.