# os-hardening (Chef cookbook)

[![Supermarket](http://img.shields.io/cookbook/v/os-hardening.svg)][1]
[![Build Status](https://travis-ci.org/dev-sec/chef-os-hardening.svg?branch=master)][2]
[![Code Coverage](http://img.shields.io/coveralls/dev-sec/chef-os-hardening.svg)][3]

## Description

This cookbook provides numerous security-related configurations, providing all-round base protection.

It configures:

 * Configures package management e.g. allows only signed packages
 * Remove packages with known issues
 * Configures `pam` and `pam_limits` module
 * Shadow password suite configuration
 * Configures system path permissions
 * Disable core dumps via soft limits
 * Restrict Root Logins to System Console
 * Set SUIDs
 * Configures kernel parameters via sysctl

It will not:

 * Update system packages
 * Install security patches

## Requirements

* Chef >= 14.0

### Platform

- Debian 7, 8
- Ubuntu 14.04, 16.04, 18.04
- RHEL 6, 7
- CentOS 6, 7
- Oracle Linux 6, 7
- Fedora 26, 27
- OpenSuse Leap 42
- Amazon Linux 1, 2


## Attributes

* `['os-hardening']['components'][COMPONENT_NAME]` - allows the fine control over which components should be executed via default recipe. See below for more details
* `['os-hardening']['desktop']['enable'] = false`
  true if this is a desktop system, ie Xorg, KDE/GNOME/Unity/etc
* `['os-hardening']['network']['forwarding'] = false`
  true if this system requires packet forwarding (eg Router), false otherwise
* `['os-hardening']['network']['ipv6']['enable'] = false`
* `['os-hardening']['network']['arp']['restricted'] = true`
  true if you want the behavior of announcing and replying to ARP to be restricted, false otherwise
* `['os-hardening']['env']['extra_user_paths'] = []`
  add additional paths to the user's `PATH` variable (default is empty).
* `['os-hardening']['env']['umask'] = "027"`
* `['os-hardening']['env']['root_path'] = "/"`
  where root is mounted
* `['os-hardening']['auth']['pw_max_age'] = 60`
  maximum password age
* `['os-hardening']['auth']['pw_min_age'] = 7`
  minimum password age (before allowing any other password change)
* `['os-hardening']['auth']['pw_warn_age'] = 7`
  number of days before maximum password age occurs to warn of impending
  change
* `['os-hardening']['auth']['uid_min'] = 1000`
  lower bound of UIDs assigned by useradd
* `['os-hardening']['auth']['uid_max'] = 60000`
  upper bound of UIDs assigned by useradd
* `['os-hardening']['auth']['gid_min'] = 1000`
  lower bound of GIDs assigned by groupadd
* `['os-hardening']['auth']['gid_max'] = 60000`
  upper bound of GIDs assigned by groupadd
* `['os-hardening']['auth']['retries'] = 5`
  the maximum number of authentication attempts, before the account is locked for some time
* `['os-hardening']['auth']['lockout_time'] = 600`
  time in seconds that needs to pass, if the account was locked due to too many failed authentication attempts
* `['os-hardening']['auth']['timeout'] = 60`
  authentication timeout in seconds, so login will exit if this time passes
* `['os-hardening']['auth']['allow_homeless'] = false`
  true if to allow users without home to login
* `['os-hardening']['auth']['pam']['passwdqc']['enable'] = true`
  true if you want to use strong password checking in PAM using passwdqc
* `['os-hardening']['auth']['pam']['passwdqc']['options'] = "min=disabled,disabled,16,12,8"`
  set to any option line (as a string) that you want to pass to passwdqc
* `['os-hardening']['auth']['pam']['passwdqc']['template_cookbook'] = 'os-hardening'`
  set to the name of the cookbook from which the template is obtained for the `/usr/share/pam-configs/passwdqc` file
* `['os-hardening']['auth']['pam']['tally2']['template_cookbook'] = 'os-hardening'`
  set to the name of the cookbook from which the template is obtained for the `/usr/share/pam-configs/tally2` file
* `['os-hardening']['auth']['pam']['system-auth']['template_cookbook'] = 'os-hardening'`
  set to the name of the cookbook from which the template is obtained for the `/etc/pam.d/system-auth-ac` file
* `['os-hardening']['security']['users']['allow'] = []`
  list of things, that a user is allowed to do. May contain: `change_user`
* `['os-hardening']['security']['kernel']['enable_module_loading'] = true`
  true if you want to allowed to change kernel modules once the system is running (eg `modprobe`, `rmmod`)
* `['os-hardening']['security']['kernel']['disable_filesystems'] = ['cramfs', 'freevxfs', 'jffs2', 'hfs', 'hfsplus', 'squashfs', 'udf', 'vfat']`
  list of kernel file system modules, which are blacklisted for loading (e.g. they are unused and can be disabled). Set this to `[]` to completely avoid this blacklisting
* `['os-hardening']['security']['kernel']['enable_sysrq'] = false`
* `['os-hardening']['security']['kernel']['enable_core_dump'] = false`
* `['os-hardening']['security']['suid_sgid']['enforce'] = true`
  true if you want to reduce SUID/SGID bits. There is already a list of items which are searched for configured, but you can also add your own
* `['os-hardening']['security']['suid_sgid']['blacklist'] = []`
  a list of paths which should have their SUID/SGID bits removed
* `['os-hardening']['security']['suid_sgid']['whitelist'] = []`
  a list of paths which should not have their SUID/SGID bits altered
* `['os-hardening']['security']['suid_sgid']['remove_from_unknown'] = false`
  true if you want to remove SUID/SGID bits from any file, that is not explicitly configured in a `blacklist`. This will make every Chef run search through the mounted filesystems looking for SUID/SGID bits that are not configured in the default and user blacklist. If it finds an SUID/SGID bit, it will be removed, unless this file is in your `whitelist`.
* `['os-hardening']['security']['suid_sgid']['dry_run_on_unknown'] = false`
  like `remove_from_unknown` above, only that SUID/SGID bits aren't removed.
  It will still search the filesystems to look for SUID/SGID bits but it will only print them in your log. This option is only ever recommended, when you first configure `remove_from_unknown` for SUID/SGID bits, so that you can see the files that are being changed and make adjustments to your `whitelist` and `blacklist`.
* `['os-hardening']['security']['packages']['clean'] = true`
  removes packages with known issues.
* `['os-hardening']['security']['packages']['list'] = ['xinetd','inetd','ypserv','telnet-server','rsh-server']`
  list of packages to remove, by default we remove the following packages:
  * xinetd ([NSA](http://www.nsa.gov/ia/_files/os/redhat/rhel5-guide-i731.pdf), Chapter 3.2.1)
  * inetd ([NSA](http://www.nsa.gov/ia/_files/os/redhat/rhel5-guide-i731.pdf), Chapter 3.2.1)
  * tftp-server ([NSA](http://www.nsa.gov/ia/_files/os/redhat/rhel5-guide-i731.pdf), Chapter 3.2.5)
  * ypserv ([NSA](http://www.nsa.gov/ia/_files/os/redhat/rhel5-guide-i731.pdf), Chapter 3.2.4)
  * telnet-server ([NSA](http://www.nsa.gov/ia/_files/os/redhat/rhel5-guide-i731.pdf), Chapter 3.2.2)
  * rsh-server ([NSA](http://www.nsa.gov/ia/_files/os/redhat/rhel5-guide-i731.pdf), Chapter 3.2.3)
* `['os-hardening']['security']['selinux_mode'] = 'unmanaged'`
  set to `unmanaged` if you want to let selinux configuration as it is. Set to `enforcing` to enforce or `permissive` to permissive SELinux.

### Controlling the included components

`default.rb` includes other components based on the ohai autodetection attributes of your system. E.g. do not execute selinux on non-RHEL systems. You can override this behavior and force components to be executed or not via setting attributes in `node['os-hardening']['components']` on the override level. Example

```ruby
# some attribute file
# do not include sysctl and auditd
override['os-hardening']['components']['sysctl'] = false
override['os-hardening']['components']['auditd'] = false

# force selinux to be included
override['os-hardening']['components']['selinux'] = true
```

In the current implementation different components are located in the different recipes. See the available recipes or `default.rb` for possible component names.

## Usage

Add the recipes to the `run_list`, it should be last:

    "recipe[os-hardening]"

Configure attributes:

    "security" : {
      "kernel" : {
        "enable_module_loading" : true
      }
    },

## Local Testing

### Local testing

Please install [chef-dk](https://downloads.chef.io/chefdk), [VirtualBox](https://www.virtualbox.org/) or VMware Workstation and [Vagrant](https://www.vagrantup.com/).

Linting is checked with [rubocop](https://github.com/bbatsov/rubocop) and [foodcritic](http://www.foodcritic.io/):

```bash
$ chef exec rake lint
.....
```

Unit/spec tests are done with [chefspec](https://github.com/sethvargo/chefspec):

```bash
$ chef exec rake spec
.....
```

Integration tests are done with [test-kitchen](http://kitchen.ci/) and [inspec](https://www.inspec.io/):

```bash
$ chef exec rake kitchen
.....
# or you can use the kitchen directly
$ kitchen test
```

### CI testing of forks

You can enable testing of your fork in [Travis CI](http://travis-ci.org/). By default you will get linting, spec tests and integration tests with [kitchen-dokken].

Integration tests with [kitchen-dokken] do not cover everything as they run in the container environment.
Full integration tests can be executed using [DigitalOcean](http://digitalocean.com/).

If you want to have full integration tests for your fork, you will have to add following [environment variables](https://docs.travis-ci.com/user/environment-variables/#Defining-Variables-in-Repository-Settings) in the settings of your fork:
- `DIGITALOCEAN_ACCESS_TOKEN` - [access token for DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-api-v2)
- `CI_SSH_KEY` - private part of some ssh key, available on DigitalOcean for your instances, in base64 encoded form (e.g. `cat id_rsa | base64 -w0 ; echo`)
- `DIGITALOCEAN_SSH_KEY_IDS` - ID in DigitalOcean of `CI_SSH_KEY`, see [this](https://github.com/test-kitchen/kitchen-digitalocean#installation-and-setup) for more information

## Contributors + Kudos

* Dominik Richter [arlimus](https://github.com/arlimus)
* Bernhard Weisshuhn [bkw](https://github.com/bkw)
* Christoph Hartmann [chris-rock](https://github.com/chris-rock)
* Edmund Haselwanter [ehaselwanter](https://github.com/ehaselwanter)
* Patrick Meier [atomic111](https://github.com/atomic111)
* Artem Sidorenko [artem-sidorenko](https://github.com/artem-sidorenko)

This cookbook is mostly based on guides by:

* [Arch Linux wiki, Sysctl hardening](https://wiki.archlinux.org/index.php/Sysctl)
* [Ubuntu Security/Features](https://wiki.ubuntu.com/Security/Features)
* [NSA: Guide to the Secure Configuration of Red Hat Enterprise Linux 5](https://www.iad.gov/iad/library/ia-guidance/security-configuration/operating-systems/guide-to-the-secure-configuration-of-red-hat-enterprise.cfm)
* [Deutsche Telekom, Group IT Security, Security Requirements (German)](https://www.telekom.com/psa)


Thanks to all of you!!

## Contributing

See [contributor guideline](CONTRIBUTING.md).

## License and Author

* Author:: Dominik Richter <dominik.richter@googlemail.com>
* Author:: Deutsche Telekom AG

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

[1]: https://supermarket.getchef.com/cookbooks/os-hardening
[2]: http://travis-ci.org/dev-sec/chef-os-hardening
[3]: https://coveralls.io/r/dev-sec/chef-os-hardening
[kitchen-dokken]: https://github.com/someara/kitchen-dokken
