# Ansible role `dhcp`

Ansible role for setting up ISC DHCPD. The responsibilities of this role are to:

- Install packages
- Manage configuration ([dhcpd.conf(5)](http://linux.die.net/man/5/dhcpd.conf))

The following are NOT concerns of this role:

- Managing firewall configuration: Use e.g. [bertvv.el7](https://galaxy.ansible.com/list#/roles/2305) for this.

## A note on compatibility

Version 2.0.0 of this role uses features that are introduced in Ansible 2.0. Users of older versions of Ansible can use version 1.1.0 which has identical functionality. An overview of role compatibility:

| Role version | Ansible version | Supported platform                         |
| :---         | :---            | :---                                       |
| 1.1.0        | 1.7.x - 1.9.x   | CentOS 6-7                                 |
| 2.0.0        | >= 2.0          | CentOS 6-7, Fedora 23, Ubuntu 14.04, 16.04 |

Only the platforms on which the role was successfully tested are mentioned, but it should also work on other versions of these distributions. If you get it working on another platform, let me know and I will add it to the list.

## Requirements

No specific requirements

## Role Variables

This role is able to set (some) global options, and to specify subnet declarations.

See the test playbook (see below) for a working example of a DHCP server with two subnets. This section is mainly a reference of all supported options.

### Global options

The following variables, when set, will be added to the global section of the DHCP configuration file. There is no default value, so when they are not set, they will be left out.

See the [dhcp-options(5)](http://linux.die.net/man/5/dhcp-options) man page for more information about these options.

| Variable                          | Comments                                                                |
| :---                              | :---                                                                    |
| `dhcp_global_authoritative`       | Global authoritative statement (authoritative, not authoritative)       |
| `dhcp_global_log_facility`        | Global log facility                                                     |
| `dhcp_global_booting`             | Global booting (allow, deny, ignore)                                    |
| `dhcp_global_bootp`               | Global bootp (allow, deny, ignore)                                      |
| `dhcp_global_broadcast_address`   | Global broadcast address                                                |
| `dhcp_global_classes`             | Class definitions with a match statement(2)                             |
| `dhcp_global_default_lease_time`  | Default lease time in seconds                                           |
| `dhcp_global_domain_name_servers` | A list of IP addresses of DNS servers(1)                                |
| `dhcp_global_domain_name`         | The domain name the client should use when resolving host names         |
| `dhcp_global_domain_search`       | A list of domain names to be used by the client to locate non-FQDNs(1)  |
| `dhcp_global_filename`            | Filename to request for boot                                            |
| `dhcp_global_includes`            | List of config files to be included (from `dhcp_config_dir`)            |
| `dhcp_global_max_lease_time`      | Maximum lease time in seconds                                           |
| `dhcp_global_next_server`         | IP for boot server                                                      |
| `dhcp_global_ntp_servers`         | List of IP addresses of NTP servers                                     |
| `dhcp_global_routers`             | IP address of the router                                                |
| `dhcp_global_server_name`         | Server name sent to the client                                          |
| `dhcp_global_server_state`        | Service state (started, stopped)                                        |
| `dhcp_global_subnet_mask`         | Global subnet mask                                                      |
| `dhcp_global_omapi_port`          | OMAPI port                                                              |
| `dhcp_global_omapi_secret`        | OMAPI secret                                                            |
| `dhcp_global_other_options`       | Array of arbitrary additional global options                            |

(1) This option may be written either as a list (when you have more than one item), or as a string (when you have only one). The following snippet shows an example of both:

```Yaml
# A single DNS server
dhcp_global_domain_name_servers: 8.8.8.8

# A list of DNS servers
dhcp_global_domain_name_servers:
  - 8.8.8.8
  - 8.8.4.4
```

(2) This role supports the definition of classes with a match statement, e.g.:

```Yaml
# Class for VirtualBox VMs
dhcp_global_classes:
  - name: vbox
    match: 'match if binary-to-ascii(16,8,":",substring(hardware, 1, 3)) = "8:0:27"'
```

Class names can be used in the definition of address pools (see below).

(3) This role also supports the definition of a failover peer, e.g.:

```Yaml
# Failover peer definition
failover_peer: failover-group
failover:
  role: primary # | secondary
  address: 192.168.56.101
  port: 647
  peer_address: 192.168.56.102
  peer_port: 647
  max_response_delay: 15
  max_unacked_updates: 10
  load_balance_max_seconds: 5
  split: 255
  mclt: 3600
```

An alphabetical list of supported options in a failover declaration:

| Option                	  | Required | Comment                                                               |
| :---                  	  | :---:    | :--                                                                   |
| `address`             	  | no       | This server's IP address                                              |
| `failover_peer`       	  | no       | Name of the configured peer, to be used on a per pool basis           |
| `hba`                 	  | no       | colon-separated-hex-list                                              |
| `load_balance_max_seconds`	  | no       | Cutoff after which load balance is disabled (3 to 5 recommended)	     |
| `max-balance`  		  | no       | Failover pool balance statement					     |
| `max-lease-misbalance`          | no       | Failover pool balance statement					     |
| `max-lease-ownership`           | no       | Failover pool balance statement					     |
| `max_response_delay`  	  | no       | Maximum seconds without contact before engaging failover              |
| `max_unacked_updates` 	  | no       | Maximum BNDUPD it can send before receiving a BNDACK (10 recommended) |
| `mclt`                	  | no       | Maximum Client Lead Time                                              |
| `min-balance`  		  | no       | Failover pool balance statement					     |
| `peer_address`        	  | no       | Failover peer's IP addres                                             |
| `peer_port`           	  | no       | This server's port (generally 519/520 or 647/847)                     |
| `port`                	  | no       | This server's port (generally 519/520 or 647/847)                     |
| `role`                	  | no       | primary, secondary                                                    |
| `split`               	  | no       | Load balance split (0-255)                                            |

The failover peer directive has to be in the definition of address pools (see below).

### Subnet declarations

The role variable `dhcp_subnets` contains a list of dicts, specifying the subnet declarations to be added to the DHCP configuration file. Every subnet declaration should have an `ip` and `netmask`, other options are not mandatory. We start this section with an example, a compelete overview of supported options follows.

```Yaml
dhcp_subnets:
  - ip: 192.168.222.0
    netmask: 255.255.255.128
    domain_name_servers:
      - 10.0.2.3
      - 10.0.2.4
    range_begin: 192.168.222.50
    range_end: 192.168.222.127
  - ip: 192.168.222.128
    default_lease_time: 3600
    max_lease_time: 7200
    netmask: 255.255.255.128
    domain_name_servers: 10.0.2.3
    routers: 192.168.222.129
```

An alphabetical list of supported options in a subnet declaration:

| Option                | Required | Comment                                                               |
| :---                  | :---:    | :--                                                                   |
| `booting`             | no       | allow,deny,ignore                                                     |
| `bootp`               | no       | allow,deny,ignore                                                     |
| `default_lease_time`  | no       | Default lease time for this subnet (in seconds)                       |
| `domain_name_servers` | no       | List of domain name servers for this subnet(1)                        |
| `domain_search`       | no       | List of domain names for resolution of non-FQDNs(1)                   |
| `filename`            | no       | filename to retrieve from boot server                                 |
| `hosts`               | no       | List of fixed IP address hosts for each subnet, similar to dhcp_hosts |
| `ip`                  | yes      | **Required.** IP address of the subnet                                |
| `max_lease_time`      | no       | Maximum lease time for this subnet (in seconds)                       |
| `netmask`             | yes      | **Required.** Network mask of the subnet (in dotted decimal notation) |
| `next_server`         | no       | IP address of the boot server                                         |
| `range_begin`         | no       | Lowest address in the range of dynamic IP addresses to be assigned    |
| `range_end`           | no       | Highest address in the range of dynamic IP addresses to be assigned   |
| `ranges`              | no       | If multiple ranges are needed, they can be specified as a list (2)    |
| `routers`             | no       | IP address of the gateway for this subnet                             |
| `server_name`         | no       | Server name sent to the client                                        |
| `subnet_mask`         | no       | Overrides the `netmask` of the subnet declaration                     |

You can specify address pools within a subnet by setting the `pools` options. This allows you to specify a pool of addresses that will be treated differently than another pool of addresses, even on the same network segment or subnet. It is a list of dicts with the following keys, all of which are optional:

| Option                | Comment                                             |
| :---                  | :---                                                |
| `allow`               | Specifies which hosts are allowed in this pool(1)   |
| `default_lease_time`  | The default lease time for this pool                |
| `deny`                | Specifies which hosts are not allowed in this pool  |
| `domain_name_servers` | The domain name servers to be used for this pool(1) |
| `max_lease_time`      | The maximum lease time for this pool                |
| `min_lease_time`      | The minimum lease time for this pool                |
| `range_begin`         | The lowest address in this pool                     |
| `range_end`           | The highest address in this pool                    |

(1) For the `allow` and `deny` fields, the options are enumerated in [dhcpd.conf(5)](http://linux.die.net/man/5/dhcpd.conf), but include:

- `booting`
- `bootp`
- `client-updates`
- `known-clients`
- `members of "CLASS"`
- `unknown-clients`

(2) For multiple subnet ranges, they can be specified, thus:

```Yaml
ranges:
  - { begin: 192.168.222.50, end: 192.168.222.99 }
  - { begin: 192.168.222.110, end: 192.168.222.127 }
```

### Host declarations

You can specify hosts that should get a fixed IP address based on their MAC by setting the `dhcp_hosts` option. This is a list of dicts with the following three keys, of which `name` and `mac` are mandatory:

| Option | Comment                                   |
| :---   | :---                                      |
| `name` | The name of the host                      |
| `mac`  | The MAC address of the host               |
| `ip`   | The IP address to be assigned to the host |

```Yaml
dhcp_hosts:
  - name: cl1
    mac: '00:11:22:33:44:55'
    ip: 192.168.222.150
  - name: cl2
    mac: '00:de:ad:be:ef:00'
    ip: 192.168.222.151
```

## Dependencies

No dependencies.

## Example Playbook

See the [test playbook](https://github.com/bertvv/ansible-role-dhcp/blob/tests/test.yml)

## Testing

Tests for this role are provided in the form of a Vagrant environment that is kept in a separate branch, `tests`. I use [git-worktree(1)](https://git-scm.com/docs/git-worktree) to include the test code into the working directory. Instructions for running the tests:

1. Fetch the tests branch: `git fetch origin tests`
2. Create a Git worktree for the test code: `git worktree add tests tests` (remark: this requires at least Git v2.5.0). This will create a directory `tests/`.
3. `cd tests/`
4. `vagrant up` will then create a VM and apply a test playbook (`test.yml`).

You may want to change the base box into one that you like. The current one, [bertvv/centos72](https://atlas.hashicorp.com/bertvv/boxes/centos72) was generated using a Packer template from the [Boxcutter project](https://github.com/boxcutter/centos) with a few modifications.

## Contributing

Issues, feature requests, ideas are appreciated and can be posted in the Issues section. Pull requests are also very welcome. Preferably, create a topic branch and when submitting, squash your commits into one (with a descriptive message).

## License

BSD

## Contributors

- [Bert Van Vreckem](https://github.com/bertvv) (maintainer)
- [Birgit Croux](https://github.com/birgitcroux/)
- [@donvipre](https://github.com/donvipre)
- Felix Egli
- [Jonathan Piron](https://github.com/jpiron)
- [Josh Benner](https://github.com/joshbenner)
- [Rian Bogle](https://github.com/rbogle/)
- [Stuart Knight](https://github.com/blofeldthefish)
