---
layout: default
title: SVMP 2.0.0 Released
---

SVMP 2.0.0 brings some major changes across many parts of the system. First and
foremost is an upgrade of the virtual phone image to the latest Android 4.4.4.
New user-facing features include sound streaming from the VM to the client, and
the addition of an HTML5 based client app in addition to the native Android app.

Behind the scenes the server architecture and network protocol have changed
significantly. The client to server connection now uses a new transport based on
websockets instead of the custom TCP protocol previously used. The server has
been split apart into separate components completing the refactoring work begun
back in SVMP v1.4. User and VM management is now handled by a central controller,
svmp-overseer, leaving svmp-server the sole role of forwarding data from
clients to their personalized virtual device. This new architecture makes it
much easier to scale to larger numbers of users through load balancers and
reverse proxying.

## Detailed Change Log

Network Protocol

* Client -> Server protocol significantly altered to use websockets as the
  underlying transport for protobuf messages instead of raw TCP.
* User login moved to a separate HTTP-based REST API within the new svmp-overseer
  server component, which yields a time-limited login token. Login now works more
  like a traditional web app as a result.
* Extraneous and obsolete messages removed from the protocol buffers definition.

Android Client

* Network code swapped out for new websocket-based protocol

Web-based Client

* Updated to work with the new websocket-based protocol
* Code merged into the web-based user management console within the svmp-overseer

SVMP Server

* Split apart and component-ized:
    * [*Overseer*](https://github.com/SVMP/svmp-overseer): Login service, session
      management API server, web-based user registration console, and serves up
      the HTML client. Centralizes all user database and cloud API interactions.
    * *svmp-server*: Handles websocket connections from clients, interacts with
      the overseer to setup VMs, and forwards user data to and from those VMs.
    * *svmp-server-cli*: Command line user management utility. Formerly included
      within the svmp-server package.
* Added support for Amazon AWS cloud API in addition to Openstack. However, the
  Android VM does not yet run correctly on EC2.
* Added user self-registration option to the Overseer's web console.
* Switched configuration file format over to YAML from JSON.
* Updated Puppet module for installing and configuring server and overseer instances.

VM

* Rebased to Android 4.4.4. And made it in before L was released!
* Sound streaming enabled. Audio from the VM is now forwarded and replayed on
  the client.
