# Ansible role `dhcp`

Ansible role for setting up ISC DHCPD on RHEL/CentOS 7. Specifically, the responsibilities of this role are to:

- Install packages
- Manage configuration

The following are NOT concerns of this role:

- Managing firewall configuration: Use e.g. [bertvv.el7](https://galaxy.ansible.com/list#/roles/2305) for this.


## Requirements

No specific requirements

## Role Variables


| Variable   | Required | Default | Comments (type)  |
| :---       | :---     | :---    | :---             |
| `role_var` | no       | -       | (scalar) PURPOSE |

## Dependencies

No dependencies.

## Example Playbook

See the [test playbook](tests/test.yml)

## Testing

The `tests` directory contains tests for this role in the form of a Vagrant environment. The directory `tests/roles/dhcp` is a symbolic link that should point to the root of this project in order to work. To create it, do

```ShellSession
$ cd tests/
$ mkdir roles
$ ln -frs ../../PROJECT_DIR roles/dhcp
```

You may want to change the base box into one that you like. The current one is based on Box-Cutter's [CentOS Packer template](https://github.com/boxcutter/centos).

The playbook [`test.yml`](tests/test.yml) applies the role to a VM, setting role variables.

## Contributing

Issues, feature requests, ideas are appreciated and can be posted in the Issues section. Pull requests are also very welcome. Preferably, create a topic branch and when submitting, squash your commits into one (with a descriptive message).

## License

BSD

## Author Information

Bert Van Vreckem (bert.vanvreckem@gmail.com)
