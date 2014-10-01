---
layout: default
title: Configuring the Server
---

The SVMP Server and Overseer are configured via their respective `config/config-local.yaml` files and via the [command line configuration tool](./server-install.html#configuration-tool).

Each server component includes an example configuration file. Copy this file to get started

    $ cp config/_config.local.template.yaml config/config-local.yaml

The available configuration options are documented extensively by comments in each template.

Several of the more important tasks are explained below.

## Managing Users

User management is done primarily via the command line configuration tool. A full listing of its commands can be obtained by running:

    $ svmp-config -h

### Creating

Create new users with the following command:

    $ svmp-config add <username> <password> <device_type>

The password field is required, but its contents are only used if the server is configured to use password authentication. The device type field is also required, and must match one of the configured types shown by

    $ svmp-config devices

See below in the Automating Lifecycle section for more details on configuring the list of device types.

### Listing User Details

To print a summary of users currently configured in the system:

    $ svmp-config list

This will output a table of all the users, the device type currently configured for that user, the ID of their persistent data volume in the cloud infrastructure, and ID and IP address of their virtual device instance if one currently exists.

### Deleting

To delete a user, run:

    $ svmp-config delete <username>

This will remove the user account from the SVMP server's database. It will leave their data volume and any instantiated virtual device intact.

## Session Management

There are several variables available that govern the length of user sessions.

`max_session_length`: This sets the maximum life time (in seconds) of a user's authentication token after logging in. If a user stays connected to their VM past the expiration of this token, the svmp-server will forcibly disconnect them. A new login is required for the user to reconnect.

## Managing Virtual Devices

### Manual Assignment

For testing, development, and small-scale deployments, users' virtual devices may be created manually and statically assigned.

1. Create a new user account as detailed above
2. First [launch a virtual device instance](vm-install.html)
3. Obtain the VM's DHCP-assigned IP address. This is printed to the boot log output to the VM's serial port console.
4. Register the VM to the user
        $ svmp-config vm-add <username> <ip address>

Example:

    $ svmp-config devices
    Supported device types (settings.new_vm_defaults.images):
    ┌─────────────┐
    │ Device Type │
    ├─────────────┤
    │ device-x    │
    └─────────────┘

    $ svmp-config add alice password email device-x
    Adding a new User...
        Created user:  alice  Password:  password  Device:  device-x

    $ svmp-config vm-add alice 10.0.0.42
    Adding existing VM for  alice
    Success: Assigned IP to an existing VM

    $ svmp-config list
    Proxy users:
    ┌───────┬───────────┬───────┬────────┬──────────┐
    │ User  │ VM_IP     │ VM_ID │ VOL_ID │ Device   │
    ├───────┼───────────┼───────┼────────┼──────────┤
    │ alice │ 10.0.0.42 │       │        │ device-x │
    └───────┴───────────┴───────┴────────┴──────────┘

When using manual VM assignment, the VM ID and data volume ID fields are ignored. All management of the virtual device, the user's persistent data volume, and binding between them is left up to the operator to configure manually.

### Automated Lifecycle with Openstack

To run at scale, the SVMP Overseer is designed to orchestrate the life cycle of virtual devices using a cloud API. In this mode, users' virtual devices will be created and destroyed dynamically as users connect and disconnect.

The Overseer relies on the [pkgcloud](https://github.com/pkgcloud/pkgcloud) library to manipulate various cloud infrastructures. Currently, the Openstack and Amazon AWS APIs are supported. As pkgcloud adds support for additional cloud targets, the SVMP server can be easily extended to take advantage of this.

**Connecting to OpenStack**

To connect the SVMP Overseer with Openstack, the `openstack:` section of the `config/config-local.yaml` file must be filled in with the data for your cloud provider. The appropriate values can be obtained from your cloud provider or administrator or from the `openrc` file obtainable through the OpenStack Horizon web UI.

Test the configuration with the command line client:

    $ svmp-config images

If the Overseer is configured correctly, a table of VM image templates and flavors
will be display. If the expected output isn't display, check the OpenStack configuration
values and try again.

**Setting Lengths and Life Spans**

If a client connects and there is not already a VM running for them, the server will create one on the fly. When a user disconnects, their VM becomes idle. After being idle for too long, a VM becomes expired and is destroyed by the server. The following configuration option controls this process:

`vm_idle_ttl`: The length of time a VM is allowed to exist after its user disconnects. If the user does not reconnect again before this timer expires, the VM is destroyed and a new one must be created the next time we see this user.

**Device Types and VM Parameters**

To allow for users that require different variants of the SVMP virtual device, the server maintains a list of mappings from a *device type* name to the unique ID of a virtual device base image stored in the cloud (e.g., an Amazon EC2 AMI name or an OpenStack Glance image UUID).

To configure the SVMP server properly you will need to obtain some information from OpenStack using the command line tool's 'images' command. Make a note of the flavor and image IDs it returns.

    $ svmp-config images

    Image flavors available
    ┌─────────────────────────┬────┐
    │ Name                    │ ID │
    ├─────────────────────────┼────┤
    │ 512MB Standard Instance │ 1  │
    ├─────────────────────────┼────┤
    │ svmp-flavor             │ 2  │
    └─────────────────────────┴────┘

    NOTE: Use the ID value when choosing a Flavor

    Images available:
    ┌───────────────────┬──────────────────────────────────────┐
    │ Name              │ ID                                   │
    ├───────────────────┼──────────────────────────────────────┤
    │ phone-image       │ 7d532f18-88ed-489b-8b7a-9dfb72f0c8f4 │
    ├───────────────────┼──────────────────────────────────────┤
    │ phablet-image     │ 63e6c5f0-e061-11e3-8df0-005056835866 │
    ├───────────────────┼──────────────────────────────────────┤
    │ tablet-image      │ 4c33ef5c-6e78-47f9-bec3-243837c16469 │
    └───────────────────┴──────────────────────────────────────┘

For example a deployment might offer different base images configured to offer different UI experiences. This could be achieved by setting the `images` variable under the `new_vm_defaults` section of the config file.

    new_vm_defaults:
        images:
            phone: "7d532f18-88ed-489b-8b7a-9dfb72f0c8f4"
            phablet: "63e6c5f0-e061-11e3-8df0-005056835866"
            tablet: "4c33ef5c-6e78-47f9-bec3-243837c16469"
        vmflavor: "2",
        ...

Pick image names ("phone", "phablet", "tablet") of your choosing and set the following values to match an Image ID returned in the images command output. These are the device type names you will use when creating new user accounts with the `user-add` command.

Set `vmflavor` equal to one of the Image Flavor IDs returned by the 'images' command. This will pick which VM resource template to use (RAM, CPU, etc.). In this example, we defined the custom template "svmp-flavor" within OpenStack for use by SVMP virtual device instances.

In the case of public clouds that force you to choose one of a pre-existing set of templates, use one that has at least 1 CPU, 1GB RAM, and 1GB of root disk.

**Other Cloud Settings**

Depending on the network architecture of your chosen cloud it may be necessary to assign floating IPs to VMs in order for them be be accessible to the outside network.

If this is the case for your installation, set `use_floating_ips` to true and set `floating_ip_pool` to the IP pool to obtain addresses from. If this feature is not needed, leave `use_floating_ips` false, and the VMs will only receive private tenant network IPs.

    new_vm_defaults:
        ...
        use_floating_ips: true,
        floating_ip_pool: "nova",
        ...

The last variable, `pollintervalforstartup` tells the svmp-server how long it should wait after ordering a VM to be created before attempting to query the cloud API for the VM's existence. Different cloud installations will take different amounts of time to create new instances. Adjust this value until it is just slightly greater than the typical new VM startup time of your cloud.

    new_vm_defaults:
        ...
        pollintervalforstartup: 2000
        ...

**Data Volumes**

If you wish to use the the SVMP server's command line tool to create users' persistent data volumes, set the following variables:

    new_vm_defaults:
        ...
        goldsnapshotId: "xxxx-xxxx-xxxx-xxxx"
        goldsnapshotSize: 6
        ...

Where `goldsnapshotId` is the UUID of a master Cinder volume snapshot to use when cloning the newly created user's volume. The `goldsnapshotSize` should be set no less than the size in GB of the master volume. **Important note**: this must be set to a *snapshot*, not a normal volume, otherwise volume creation will fail.

With these variables set correctly, the command line tool can then create user volumes like so:

    $ svmp-config volume-create <username>

Which will create a new volume cloned from the master volume, and link it to that user account.

Alternatively, if the data volumes are created some other way external to SVMP, they can be manually assigned to user accounts within SVMP.

    $ svmp-config volume-assign <username> <cinder-volume-uuid>

## Logging

The SVMP server can output detailed logs of user sessions and connection attempts. The log file and detail are controlled by two variables:

1. `log_file`: Full path to the output log file. Log messages are written in a machine readable JSON format with an attached timestamp value. Corresponding human readable messages are printed to stdout.

2. `log_level`: The verbosity of log output. For production deployments, 'info' is the recommended output level. The higher levels ("verbose", "debug", and "silly") may be useful to debug configuration and runtime errors.

   **Warning**: The highest levels ("silly" and "debug") will output potentially sensitive data to the log file including session tokens. Use these levels only if you know what you are doing, and **never** in production.

## SSL / TLS

To configure the server to use TLS encryption between it and the client set the following fields within `config/config-local.js` to enable use of TLS and provide the full paths of the key and certificate files.

    enable_ssl: true
    server_certificate: '/path/to/server-cert.pem'
    private_key: '/path/to/server-key.pem'
    private_key_pass: 'password for the private key'

If the private key file is not password protected, use an empty string value.

    tls_private_key_pass: '',

## Configuring Authentication

The SVMP Overseer can use one of several methods to authenticate users.

### Password

Unless one of the other methods is specifically enabled, the server will default to password authentication. Passwords are stored in the server's MongoDB database. The user model requires that passwords be at least 8 characters long.

    authentication_type: 'pam'

### PAM

The SVMP server can use the underlying system's Pluggable Authentication Modules service for performing user authentication. This is the preferred method for production environments. By leveraging PAM, the SVMP server can take advantage of any of the variety of authentication methods already implemented as PAM modules, including RADIUS, LDAP, /etc/passwd, and many more.

To enable PAM authentication, set the following fields in the config file:

    authentication_type: 'pam'
    pam_service: 'service-name'

Where 'service-name' corresponds with a file `/etc/pam.d/service-name`.

Configuring PAM is outside the scope of this document, but there are many good resources available online. Only the "auth" facility is necessary.

An example using RADIUS:

    $ cat /etc/pam.d/svmp
    auth    required    pam_radius_auth.so


### PKI Certificates

To use x.509 PKI certificate based authentication:

1. Ensure the server is configured to use TLS encryption. See above.
2. Set `authentication_type` to 'certificate'.
3. Set `ca_cert` to the path of the certificate authority public cert used to sign the client certificates.
4. Install client certs signed by that CA on the client devices.

The default authentication logic when using certificates is to match the email address field from the user certificate offered against the server's username database. The correct logic and fields may differ from one deployment to another, so customize the `lib/authentication/plugins/tls.js` file to match the needs of your environment.

## Configuring the Server

Each svmp-server instance must be configured to communicate with the Overseer.

`overseer_url`: The base URL for REST calls to the Overseer.

`overseer_cert`: Path to a PEM file containing the Overseer's public certificate. Required to verify authentication tokens granted to users.

`auth_token`: The server's authentication token created by the Overseer. Grants this svmp-server instance access to the Overseer's REST API for managing VM lifecycle.

To generate authentication tokens for the svmp-server use the `create-token` script included with the Overseer.

    $ node create-token.js -a [-e expiration_time] svmp-server-N

Use this tool also to create tokens for admin users for use with the svmp-config tool.
