# ansible-fail2ban

A role to install and configure fail2ban on a target host.

## Requirements

### Ansible version

Minimum required ansible version is 2.0.


## Role Variables

### Variables conditionally loaded

Add jails with custom filters.


Example below bans IP if too many requests give a 403 using a custom filter and custom ban action

```yaml
- role: fail2ban
  fail2ban_jails:
      - name: kinto_auth  # nginx 403 for kinto fails
        enabled: 'true'
        filter: kinto-auth
        action: nginx-blacklist
        logpath: /var/log/nginx/kinto_nginx_access.log
  fail2ban_filters: # custom filters
      - name: kinto-auth
        failregex: '<HOST> - .* \[.*\] ".*" 403 \d+ ".*" ".*" ".*"'
  fail2ban_actions: # Custom actions, all keys optionnal
      - name: nginx-blacklist
        actionstart:
            - echo "" > /etc/nginx/ip_blacklist.conf # Unban all before starting
            - touch /var/run/fail2ban/fail2ban.dummy
            - printf %%b "<init>\n" >> /var/run/fail2ban/fail2ban.dummy
        actionstop:
            - echo "" > /etc/nginx/ip_blacklist.conf # Unban all before stopping
            - systemctl reload openresty
            - rm -f /var/run/fail2ban/fail2ban.dummy
        actioncheck:
            - echo "ok"
        actionban:
            - echo "<ip> 0;" >> /etc/nginx/ip_blacklist.conf
            - systemctl reload openresty
        actionunban:
            - sed -i "/$(echo "<ip>" | sed 's/\./\\\./g') 0;/d" /etc/nginx/ip_blacklist.conf
            - systemctl reload openresty
        init:
            init: "something"

```

### Default vars

Defaults from `defaults/main.yml`.

```yaml
# defaults file for fail2ban

# service
fail2ban_svc_state: started
fail2ban_svc_enabled: yes

fail2ban_pkg_state: latest
fail2ban_use_firewalld: no

# defaults
fail2ban_jail_default:
  bantime: 600
  maxretry: 3
  banaction: iptables-multiport

# fail2ban_sshd
fail2ban_jails:
  - name: sshd
    enabled: 'true'
    maxretry: '5'

```


## Installation

### Install with Ansible Galaxy

```shell
ansible-galaxy install archf.fail2ban
```

Basic usage is:

```yaml
- hosts: all
  roles:
    - role: archf.fail2ban
```

### Install with git

If you do not want a global installation, clone it into your `roles_path`.

```shell
git clone git@github.com:archf/ansible-fail2ban.git /path/to/roles_path
```

But I often add it as a submdule in a given `playbook_dir` repository.

```shell
git submodule add git@github.com:archf/ansible-fail2ban.git <playbook_dir>/roles/fail2ban
```

As the role is not managed by Ansible Galaxy, you do not have to specify the
github user account.

Basic usage is:

```yaml
- hosts: all
  roles:
  - role: fail2ban
```

## Ansible role dependencies

None.

## License

MIT.

## Author Information

Felix Archambault.

## Role stack

This role was carefully selected to be part an ultimate deck of roles to manage
your infrastructure.

All roles' documentation is wrapped in this [convenient guide](http://127.0.0.1:8000/).


---
This README was generated using ansidoc. This tool is available on pypi!

```shell
pip3 install ansidoc

# validate by running a dry-run (will output result to stdout)
ansidoc --dry-run <rolepath>

# generate you role readme file
ansidoc <rolepath>
```

You can even use it programatically from sphinx. Check it out.
