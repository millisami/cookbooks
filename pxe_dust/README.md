Description
===========

Configures a tftpd server for serving Ubuntu installers over PXE and setting them to run a provided preseed.cfg.

Requirements
============

Requires Chef 0.10.0+.

## Platform:

Tested on:

* Ubuntu 10.04-11.10
* Debian 6.0

## Cookbooks:

Required: apache2, tftp

Optional (recommended): apt (for `recipe[apt::cacher]`)

pxe_dust Data Bag
=================

In order to manage configuration of machines registering themselves with their Chef Server or Opscode Hosted Chef, we will use the `pxe_dust` data bag.

```
% knife data bag create pxe_dust
% knife data bag from file pxe_dust examples/default.json
```

Here is an example of the default.json:

```json
{
    "id": "default",
    "arch": "amd64",
    "version": "lucid",
    },
    "user": {
        "fullname": "Ubuntu",
        "username": "ubuntu",
        "crypted_password": "$6$Trby4Y5R$bi90k7uYY5ImXe5MWGFW9kel2BnMCcYO9EnwngTFIXKG2/nWcLKTJZ3verMFnpFbITI9.eHwZ.HR1UPeKbCAV1"
    }
}
```

Here are currently supported options available for inclusion in the `default.json`.:

* `arch`: Architecture of the netboot.tar.gz to use as the source of pxeboot images, default is 'amd64'.
* `version`: Ubuntu version of the netboot.tar.gz to use as the source of pxeboot images, default is 'lucid'.
* `domain`: Default domain for nodes, default is none.
* `run_list`: Default run list for nodes, default is none.
* `bootstrap`: Optional additional bootstrapping configuration.
    `bootstrap_version_string`: for building specific version of Chef, default is none.
    `http_proxy`: HTTP proxy, default is none.
    `http_proxy_user`: HTTP proxy user, default is none.
    `http_proxy_pass`: HTTP proxy pass, default is none.
    `https_proxy`: HTTPS proxy, default is none.
* `user`:
    `crypted_password`: SHA512 password for the default user, default 'password'. This may be generated and added to the data bag.
    `fullname`: Full name of the default user, default 'Ubuntu'.
    `username`: Username of the default user, default 'ubuntu'.

Templates
=========

syslinux.cfg.erb
----------------

Sets the boot prompt to automatically run the installer.

txt.cfg.erb
-----------

Sets the URL to the preseed file, architecture, the domain and which interfaces to use.

preseed.cfg.erb
---------------

The preseed file is full of opinions, you will want to update this. If there is a node providing an apt-cacher proxy via `recipe[apt::cacher]`, it is provided in the preseed.cfg. The initial user and password is configured. The preseed finishes by calling the `chef-bootstrap` script.

chef-bootstrap.sh.erb
---------------------

This is the `preseed/late_command` that bootstraps the node with Chef via gems.

Recipes
=======

Default
-------

The default recipe includes recipe `pxe_dust::server`.

Server
------

`recipe[pxe_dust::server]` includes the `apache2` and `tftp::server` recipes.

The recipe does the following:

1. Downloads the proper netboot.tar.gz to boot from.
2. Untars it to the `[:tftp][:directory]` directory.
3. Instructs the installer prompt to automatically install.
4. Passes the URL of the preseed.cfg to the installer.
5. Uses the preseed.cfg template to pass in any `apt-cacher` proxies.

Usage
=====

Add `recipe[pxe_dust::server]` to a node's or role's run list. Create the `pxe_dust` data bag and update the `defaults.json` item before adding it.

On an Ubuntu system, the password can be generated by installing the `mkpasswd` package and running:

    mkpasswd -m sha-512

The default is the hash of the password `ubuntu`, if you'd like to test. This must be set in the `pxe_dust` data bag to a valid sha-512 hash of the password or you will not be able to log in.

This cookbook does not provide DHCP or bootp to listen for PXE boot requests, this URL will have to be provided by another cookbook or manually. The author had to do this manually on a DD-WRT router.

Side note, for DD-WRT bootp support [this forum post was followed](http://www.dd-wrt.com/phpBB2/viewtopic.php?t=4662). The key syntax was

    dhcp-boot=pxelinux.0,,192.168.1.147

in the section `Additional DNSMasq Options` where the IP address is that of the tftpd server we're configuring here and pxelinux.0 is from the netboot tarball.

Changes
=======

## v1.1.2:

* Fixes COOK-481, COOK-594

License and Author
==================

Author:: Matt Ray <matt@opscode.com>
Author:: Joshua Timberman <joshua@opscode.com>

Copyright:: 2011 Opscode, Inc

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.