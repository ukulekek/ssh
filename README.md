# ssh
![CI Testing](https://github.com/linux-system-roles/ssh/workflows/tox/badge.svg)

An Ansible role for managing ssh clients configuration.

## Requirements

This role should work on any system that provides openssh client and is
supported by ansible. The role was tested on:

 * RHEL/CentOS 6, 7, 8
 * Fedora 32, 33
 * Debian
 * Ubuntu

## Role Variables

By default, the role should not modify the system configuration and generate
global `ssh_config` that matches OS default (the generated configuration does
not keep comments and order of the options).

 * `ssh_user`:

By default (*null*) the role will modify the global configuration for all
users. Other values will be interpretted as a username and the role will
modify per-user configuration stored under `~/.ssh/config` of the given user.
The user needs to exists before invoking this role otherwise it will fail.

 * `ssh_skip_defaults`:

By default (`auto`), the role writes the system-wide configuration file
`/etc/ssh/ssh_config` and keeps OS defaults defined there (*true*). This is
automatically disabled, when a drop-in configuration file is created
(`ssh_drop_in_name!=null`) or when per-user configuration file is created
(`ssh_user!=null`).

 * `ssh_drop_in_name`:

This defines the name for the drop-in configuration file to be placed in
system-wide drop-in directory. The name is used in the template
defined by `__ssh_drop_in_template` (by default
`/etc/ssh/ssh_config.d/{name}.conf`) to reference the configuration file
to be modified. If the system does not support drop-in directory, setting
this option will make the play fail. Default is `null` if the system does
not support drop in directory and `00-ansible` otherwise.

The suggested format is `NN-name`, where `NN` is two-digit number used for
sorting the and `name` is any descriptive name for the content or the owner
of the file.

 * `ssh`:

A dict containing configuration options and respective values. See example
below.

 * `ssh_...`:

Simple variables consisting of the option name prefixed with `ssh_` can be
used rather than a dict above. The simple variable overrides values in dict
above.

 * `ssh_additional_packages`:

This role automatically installs packages needed for most common use cases
on given platform. If some additional packages need to be installed (for
example `openssh-keysign` for hostbased authentication), they can be specified
in this variable.

### Secondary role variables:

These variables are used by the role internals and can be used to override
the defaults that correspond to each supported platform.

 * `__ssh_packages`:

Minimal list of packages to install on a given platform for openssh clients
to function. Do not override this variable if you need to install additional
packages. Use `ssh_additional_packages` instead.

 * `ssh_config_file`:

The configuration file that will be written by this role. The default is
defined by template `__ssh_drop_in_template` if system has drop-in directory
or `/etc/ssh/ssh_config` otherwise. If `ssh_user!=null`, the
default is `~/.ssh/config`.

 * `__drop_in_template`:

The template for a filename used for global drop-in configuration snippets.
The default value is `/etc/ssh/ssh_config.d/{name}.conf`.

 * `ssh_config_owner`, `ssh_config_group`, `ssh_config_mode`:

The owner, group and mode of the created configuration file. The files are
owned by `root:root` with mode `0644` by default, unless
`ssh_user!=null`. In that case, the mode is `0600` and owner and
group are derived from username given in `ssh_user` variable.


## Dependencies

none

## Example Playbook

Including an example of how to use your role (for instance, with variables
passed in as parameters) is always nice for users too:

```yaml
- hosts: all
  tasks:
  - name: "Configure ssh clients"
    include_role:
      name: linux-system-roles.ssh
    vars:
      ssh_user: root
      ssh:
        Compression: true
        GSSAPIAuthentication: no
        ControlMaster: auto
        ControlPath: ~/.ssh/.cm%C
        Match:
          - Condition: "final all"
            GSSAPIAuthentication: yes
        Host:
          - Condition: example
            Hostname: example.com
            User: somebody
      ssh_ForwardX11: no
```

More examples can be provided in the [`examples/`](examples) directory. These
can be useful especially for documentation.

## License

LGPLv3, see the file LICENSE for more information.

## Author Information

Jakub Jelen, 2021
