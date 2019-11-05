# Ansible role 'ansible-role-redhat-base'

An Ansible role for initial base setup of RedHat family distributions, specifically:

- Manage repositories
- Manage package installation and removal
- Turn specific services on/off
- Create local users and groups
- Set up an administrator account with an SSH key
- Apply basic security settings such as SELinux and the firewall
- Manage firewall rules in the public zone

## Requirements

- RedHat family OS

## Role Variables
| Variable		| Default		| Comments (type) |
| :---			| :---			| :---		  |
| `base_enable_repos`             | []              | List of dicts specifying repositories to be enabled. See below for details.                                           |
| `base_firewall_allow_ports`     | []              | List of ports to be allowed to pass through the firewall, e.g. 80/tcp, 53/udp, etc.                                   |
| `base_firewall_allow_services`  | []              | List of services to be allowed to pass through the firewall, e.g. http, dns, etc.(1)                                  |
| `base_firewall_interfaces`      | []              | List of network interfaces to be added to the public zone of the firewall ruleset.                                    |
| `base_hosts_entry`              | true            | When set, an entry is added to `/etc/hosts` with the machine's host name. This speeds up gathering facts.             |
| `base_install_packages`         | []              | List of packages that should be installed. URLs are also allowed.                                                     |
| `base_motd`                     | false           | When set, a custom `/etc/motd` is installed with info about the host name and IP addresses.                           |
| `base_remove_packages`          | []              | List of packages that should **not** be installed                                                                     |
| `base_repo_exclude_from_update` | []              | List of packages to be excluded from an update. Wildcards allowed, e.g. `kernel*`.                                    |
| `base_repo_exclude`             | []              | List of repositories that should be disabled in yum/dnf.conf                                                          |
| `base_repo_gpgcheck`            | false           | When set, GPG checks will be performed when installing packages.                                                      |
| `base_repo_installonly_limit`   | 3               | The maximum number of versions of a package (e.g. kernel) that can be installed simultaneously. Should be at least 2. |
| `base_repo_remove_dependencies` | true            | When set, dependencies that become unused after removing a package will be removed as well.                           |
| `base_repositories`             | []              | List of RPM packages (including URLs) that install external repositories (e.g. `epel-release`).                       |
| `base_selinux_state`            | enforcing       | The default SELinux state for the system. Just [leave this as is](http://stopdisablingselinux.com/).                  |
| `base_selinux_booleans`         | []              | List of SELinux booleans to be set to on, e.g. httpd_can_network_connect                                              |
| `base_ssh_key`                  | -               | The public SSH key for the admin user that allows her to log in without a password. The user should exist.            |
| `base_ssh_user`                 | -               | The name of the user that will manage this machine. The SSH key will be installed into the user's home directory.(3)  |
| `base_start_services`           | []              | List of services that should be running and enabled.                                                                  |
| `base_stop_services`            | []              | List of services that should **not** be running                                                                       |
| `base_tz`                       | :/etc/localtime | Sets the `$TZ` environment variable (4)                                                                               |
| `base_update`                   | false           | When set, a package update will be performed.                                                                         |
| `base_user_groups`              | []              | List of user groups that should be present.                                                                           |
| `base_users`                    | []              | List of dicts specifying users that should be present. See below for an example.                                      |

**Remarks:**

(1) A complete list of valid values for `base_firewall_allow_services` can be enumerated with the command `firewall-cmd --get-services`.

(2) This is a workaround for [CentOS bug #7407](https://bugs.centos.org/view.php?id=7407). NetworkManager by default manages firewall zones, which overrides rules you add with `--permanent`.

(3) Setting the variable `base_ssh_user` does not actually create a user, but installs the `base_ssh_key` in that user's home directory (`~/.ssh/authorized_keys`). Consequently, `base_ssh_user` should be the name of an existing user, specified in `base_users`.

(4) setting `$TZ` variable may reduce the number of system calls. See <https://blog.packagecloud.io/eng/2017/02/21/set-environment-variable-save-thousands-of-system-calls/>

### Enabling repositories

Enable (installed, but disabled) repositories by specifying `base_enable_repos` as a list of dicts with keys `name:` (required) and `section:` (optional), e.g.:

```Yaml
base_enable_repos:
  - name: CentOS-fasttrack
    section: fasttrack
  - name: epel-testing
```

When the section is not specified, it defaults to the repository name.

### Adding users

Users are specified by dicts like this:

```Yaml
base_users:
  - name: johndoe
    comment: 'John Doe'
    groups:
      - users
      - devs
    password: '$6$WIFkXf07Kn3kALDp$fHbqRKztuufS895easdT [...]'
  - name: janedoe
```

The only mandatory key is `name`.

| Key        | Required | Default     | Comments                               |
| :---       | :---:    | :---:       | :---                                   |
| `name`     | yes      | -           | The user name                          |
| `comment`  | no       | ''          | Comment string                         |
| `shell`    | no       | '/bin/bash' | The user's command shell               |
| `groups`   | no       | []          | Groups this user should be added to(1) |
| `password` | no       | '!!'        | The user's password hash(2)            |

**Remarks:**

(1) If you want to make a user an administrator, make sure they are member of the group `wheel` (See [RedHat System Administrator's Guide](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/chap-Gaining_Privileges.html#sect-Gaining_Privileges-The_su_Command).

(2) The password should be specified as a hash, as returned by [crypt(3)](http://man7.org/linux/man-pages/man3/crypt.3.html), in the form `$algo$salt$hash`. For tests and proof-of-concept VMs, you can take a look at <https://www.mkpasswd.net/> for generating hashes in the correct form. Typical hash types for Linux are MD5 (crypt-md5, hashes starting with `$1$`) and SHA-512 (crypt-sha-512, hashes starting with `$6$`).

## Dependencies

## Example Playbook
```Yaml
- hosts: foo
  roles:
    - role: ansible-role-redhat-base
```

## Testing


## License

BSD

## Contributors

Issues, feature requests, ideas, suggestions, etc. are appreciated and can be posted in the Issues section. Pull requests are also very welcome. Please create a topic branch for your proposed changes, it's the easiest way to merge back into the project.

- [Oscar Petersson](https://github.com/oscpe262/) (Maintainer)

## Main influence
- [Bert Van Vreckem](https://github.com/bertvv/)
