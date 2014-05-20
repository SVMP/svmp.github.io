---
layout: default
title: Configuring the Server
---

The SVMP server is configured via the `config/config-local.js` file and via the command line application included with the server.

An example configuration file is provided. Copy this file to get started

    $ cp config/config-local-example.js config/config-local.js

The available configuration options are documented extensively by comments in the [example file](https://github.com/SVMP/svmp-server/blob/master/config/config-local-example.js) as well as in the [`config/README.txt`](https://raw.githubusercontent.com/SVMP/svmp-server/master/config/README.txt).

Several of the more important tasks are explained below.

## Managing Users

User management is done via the server's command line tool. A full listing of its commands can be obtained by running:

    $ node bin/cli.js -h

### Creating

Create new users with the following command:

    $ node bin/cli.js add <username> <password> <device_type>

The password field is required, but its contents are only used if the server is configured to use password authentication. The device type field is also required, and must match one of the configured types shown by

    $ node bin/cli.js devices

See below in the Automating Lifecycle section for more details on configuring the list of device types.

### Listing User Details

To print a summary of users currently configured in the system:

    $ node bin/cli.js list

This will output a table of all the users, the device type currently configured for that user, the ID of their persistent data volume in the cloud infrastructure, and ID and IP address of their virtual device instance if one currently exists.

### Deleting

To delete a user, run:

    $ node bin/cli.js delete <username>

This will remove the user account from the SVMP server's database. It will leave their data volume and any instantiated virtual device intact.

## Session Management

There are several variables available that govern the length of user sessions.

All of thse variables are denominated in seconds.

1. `max_session_length`: This sets the maximum life time of a connected user session. If a user stays attempts to keep an active connection open longer than this time, the server will forcibly disconnect it.

2. `session_token_ttl`: This field sets the lifetime of the session tokens the server generates and passes to a user upon successful login. If a user disconnects from their session, and reconnects again within the number of seconds specified by this field, the token will be used to reactivate the user's connection without requiring full re-authentication.

  The purpose of this is to allow users to briefly switch between local apps on their phone and then switch back to SVMP and login again without being burdened by constant password checks. It also helps mitigate dropped connections over low-quality cellular links.  

3. `session_check_interval`: How often to run the task that checks if sessions have surpassed either of the previous two limits. The default value for this field should be sufficient for most deployments.

## Managing Virtual Devices

### Manual Assignment

For testing, development, and small-scale deployments, users' virtual devices may be created manually and statically assigned.

1. Create a new user account as detailed above
2. First [launch a virtual device instance](vm-install.html)
3. Obtain the VM's DHCP-assigned IP address. This is printed to the boot log output to the VM's serial port console.
4. Register the VM to the user
        $ node bin/cli.js vm-add <username> <ip address>

Example:

    $ node bin/cli.js devices
    Supported device types (settings.new_vm_defaults.images):
    ┌─────────────┐
    │ Device Type │
    ├─────────────┤
    │ device-x    │
    └─────────────┘

    $ node bin/cli.js add alice password device-x
    Adding a new User...
        Created user:  alice  Password:  password  Device:  device-x

    $ node bin/cli.js vm-add alice 10.0.0.42
    Adding existing VM for  alice
    Success: Assigned IP to an existing VM

    $ node bin/cli.js list
    Proxy users:
    ┌───────┬───────────┬───────┬────────┬──────────┐
    │ User  │ VM_IP     │ VM_ID │ VOL_ID │ Device   │
    ├───────┼───────────┼───────┼────────┼──────────┤
    │ alice │ 10.0.0.42 │       │        │ device-x │
    └───────┴───────────┴───────┴────────┴──────────┘

When using manual VM assignment, the VM ID and data volume ID fields are ignored. All management of the virtual device, the user's persistent data volume, and binding between them is left up to the operator to configure manually.

### Automated Lifecycle with Openstack

To run at scale, the SVMP server is designed to orchestrate the life cycle of virtual devices using a cloud API. In this mode, users' virtual devices will be created and destroyed dynamically as users connect and disconnect.

The SVMP server relies on the [pkgcloud](https://github.com/pkgcloud/pkgcloud) library to manipulate various cloud infrastructures. Currently, only the Openstack API (and compatible) is fully supported. As pkgcloud adds support for additional cloud targets, the SVMP server can be easily extended to take advantage of this.

**Connecting to OpenStack**

To connect the SVMP server with Openstack, the `openstack: {}` section of the `config/config-local.js` file must be filled in with the data for your cloud provider. The appropriate values can be obtained from your cloud provider or administrator as well as from the `openrc` file obtainable through the OpenStack Horizon web UI.

Test the configuration with the command line client:

    $ node bin/cli.js images

If there is an error in your configuration, the command will output an error similar to:

    Image flavors available
    ERROR:  { [Error: connect ECONNREFUSED]
      code: 'ECONNREFUSED',
      errno: 'ECONNREFUSED',
      syscall: 'connect' }

Check the OpenStack configuration values and try again.

**Setting Lengths and Life Spans**

If a client connects and there is not already a VM running for them, the server will create one on the fly. When a user disconnects, their VM becomes idle. After being idle for too long, a VM becomes expired and is destroyed by the server. The following configuration options control this process:

1. `vm_idle_ttl`: The length of time a VM is allowed to exist after its user disconnects. If the user does not reconnect again before this timer expires, the VM is destroyed and a new one must be created the next time we see this user.

2. `vm_check_interval`: How often the server checks idle VMs to see if they are expired. The default value for this field should be sufficient for most deployments.

**Device Types and VM Parameters**

To allow for users that require different variants of the SVMP virtual device, the server maintains a list of mappings from a *device type* name to the unique ID of a virtual device base image stored in the cloud (e.g., an Amazon EC2 AMI name or an OpenStack Glance image UUID).

To configure the SVMP server properly you will need to obtain some information from OpenStack using the command line tool's 'images' command. Make a note of the flavor and image IDs it returns.

    $ node bin/cli.js images

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

    new_vm_defaults: {
        "images": {
            "phone": "7d532f18-88ed-489b-8b7a-9dfb72f0c8f4",
            "phablet": "63e6c5f0-e061-11e3-8df0-005056835866",
            "tablet": "4c33ef5c-6e78-47f9-bec3-243837c16469"
        },
        vmflavor: "2",
        ...
    }

Pick image names ("phone", "phablet", "tablet") of your choosing and set the following values to match an Image ID returned in the images command output. These are the device type names you will use when creating new user accounts with the `user-add` command.

Set `vmflavor` equal to one of the Image Flavor IDs returned by the 'images' command. This will pick which VM resource template to use (RAM, CPU, etc.). In this example, we defined the custom template "svmp-flavor" within OpenStack for use by SVMP virtual device instances.

In the case of public clouds that force you to choose one of a pre-existing set of templates, use one that has at least 1 CPU, 1GB RAM, and 1GB of root disk.

**Other Cloud Settings**

Depending on the network architecture of your chosen cloud it may be necessary to assign floating IPs to VMs in order for them be be accessible to the outside network.

If this is the case for your installation, set `use_floating_ips` to true and set `floating_ip_pool` to the IP pool to obtain addresses from. If this feature is not needed, leave `use_floating_ips` false, and the VMs will only receive private tenant network IPs.

    new_vm_defaults: {
        ...
        use_floating_ips: true,
        floating_ip_pool: "nova",
        ...
    }

The last variable, `pollintervalforstartup` tells the svmp-server how long it should wait after ordering a VM to be created before attempting to query the cloud API for the VM's existence. Different cloud installations will take different amounts of time to create new instances. Adjust this value until it is just slightly greater than the typical new VM startup time of your cloud.

    new_vm_defaults: {
        ...
        pollintervalforstartup: 2000
        ...
    }

**Data Volumes**

If you wish to use the the SVMP server's command line tool to create users' persistent data volumes, set the following variables:

    new_vm_defaults: {
        ...
        goldsnapshotId: "xxxx-xxxx-xxxx-xxxx",
        goldsnapshotSize: 6,
        ...
    }

Where `goldsnapshotId` is the UUID of a master Cinder volume to use when cloning the newly created user's volume. The `goldsnapshotSize` should be set no less than the size in GB of the master volume.

With these variables set correctly, the command line tool can then create user volumes like so:

    $ node bin/cli.js volume-create <username>

Which will create a new volume cloned from the master volume, and link it to that user account.

Alternatively, if the data volumes are created some other way external to SVMP, they can be manually assigned to user accounts within SVMP.

    $ node bin/cli.js volume-assign <username> <cinder-volume-uuid>

## Logging

The SVMP server can output detailed logs of user sessions and connection attempts. The log file and detail are controlled by two variables:

1. `log_file`: Full path to the output log file. Log messages are written in a machine readable JSON format with an attached timestamp value. Corresponding human readable messages are printed to stdout.

2. `log_level`: The verbosity of log output. For production deployments, 'info' is the recommended output level. The higher levels ("verbose", "debug", and "silly") may be useful to debug configuration and runtime errors.

   **Warning**: The highest levels ("silly" and "debug") will output potentially sensitive data to the log file including passwords and session tokens. Use these levels only if you know what you are doing, and **never** in production.

## SSL / TLS

To configure the server to use TLS encryption between it and the client set the following fields within `config/config-local.js` to enable use of TLS and provide the full paths of the key and certificate files.

    module.exports = {
        settings: {
            . . .
            tls_proxy: true,
            tls_certificate: '/path/to/server-cert.crt',
            tls_private_key: '/path/to/server-cert-key.priv',
            tls_private_key_pass: 'password for the private key',
            . . .
        },
        . . .
    }

If the private key file is not password protected, use an empty string value.

    tls_private_key_pass: '',

## Configuring Authentication

The SVMP server can use one of several methods to authenticate users.

### Password

Unless one of the other methods is specifically enabled, the server will default to password authentication. Passwords are stored in the server's MongoDB database.

**Warning**: In its current form, password authentication is meant for testing and development use only. The database stores password in the clear without either hashing or encryption. This is insecure for production use.

### PAM

The SVMP server can use the underlying system's Pluggable Authentication Modules service for performing user authentication. This is the preferred method for production environments. By leveraging PAM, the SVMP server can take advantage of any of the variety of authentication methods already implemented as PAM modules, including RADIUS, LDAP, /etc/passwd, and many more.

To enable PAM authentication, set the following fields in the config file:

    module.exports = {
        settings: {
            . . .
            use_pam: true,
            pam_service: 'service-name',
            . . .
        },
        . . .
    }

Where 'service-name' corresponds with a file `/etc/pam.d/service-name`.

Configuring PAM is outside the scope of this document, but there are many good resources available online. Only the "auth" facility is necessary.

An example using RADIUS:

    $ cat /etc/pam.d/svmp
    auth    required    pam_radius_auth.so


### PKI Certificates

To use x.509 PKI certificate based authentication:

1. Ensure the server is configured to use TLS encryption. See above.
2. Set `use_tls_user_auth` to true.
3. Set `tls_ca_cert` to the path of the certificate authority public cert used to sign the client certificates.
4. Install client certs signed by that CA on the client devices.

The default authentication logic when using certificates is to match the email address field from the user certificate offered against the server's username database. The correct logic and fields may differ from one deployment to another, so customize the `lib/authentication/plugins/tls.js` file to match the needs of your environment.
