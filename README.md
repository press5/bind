# bind

Ansible role to setup [BIND (Berkley Internet Naming Daemon)](https://www.isc.org/downloads/bind/).


## Requirements

- ansible >= 1.9.5


## Role Variables

- **debug**: flag to run debug tasks (default: false).
- **bind_dir_cache**: directory where to store the bind zone files.
- **bind_dir_log**: directory where to store the bind log files.
- **bind_default_resolvconf**: wether or not you want to run resolvconf.
- **bind_default_options**: extra parameters to pass to the bind daemon.
- **bind_local_includes**: any includes for `named.conf.local`.
- **bind_named_conf_acl**: content for the `named.conf` `acl` section.
- **bind_named_conf_controls**: content for the `named.conf` `controls` section.
- **bind_named_conf_keys**: keys for the `named.conf` `keys` section
                            (see example below for format).
- **bind_named_conf_logging**: `channels` and `categories` for the `named.conf` `logging` section
                               (see example below for format).
- **bind_named_conf_options**: content for the `named.conf` `options` section (mandatory).

Unless stated otherwise
a default value is provided for each of the variables mentioned above
in `defaults/main.yml`.


## Dependencies

None.


## Playbooks

Example:

    - hosts: servers
      vars:
        debug: yes

        bind_default_options: '-4 -u bind'
        bind_dir_cache: /var/cache/bind
        bind_dir_log: /var/log/named

        bind_named_conf_acl:
          trusted: |
            localhost;
            localnets;

        bind_named_conf_controls: |
          inet 127.0.0.1 port 953 allow { 127.0.0.1; };

        bind_named_conf_keys:
          mykey:
            algorithm: hmac-md5
            secret: QJc08cnP1xkoF4a/eSZZbw==
          mykey2:
            algorithm: hmac-md5
            secret: QJc08cnP1xkoF4a/eSZZbw==

        bind_named_conf_logging:
          channels:
            update_debug: |
              file "{{ bind_dir_log }}/update_debug.log" versions 3 size 100k;
              severity debug;
              print-severity  yes;
              print-time      yes;
            security_info: |
              file "{{ bind_dir_log }}/security_info.log" versions 1 size 100k;
              severity info;
              print-severity  yes;
              print-time      yes;
            bind_log: |
              file "{{ bind_dir_log }}/bind.log" versions 3 size 1m;
              severity info;
              print-category  yes;
              print-severity  yes;
              print-time      yes;
          categories:
            default: bind_log
            lame-servers: 'null'
            update: update_debug
            update-security: update_debug
            security: security_info

        bind_local_includes:
          - /etc/bind/named.local.includes

        bind_zones:
          example.com:
            type: master
            file: "\"{{ bind_dir_cache }}/db.example.com.zone\""

        bind_zone_databases:
            example.com:
              directives:
                ORIGIN: example.com.
                TTL: 3600
              resource_records:
                - { name: 'example.com.', class: 'IN', type: 'SOA', data: 'sid.example.com. root.example.com. ( 2007120710 1d 2h 4w 1h )' }

                - { name: '@', class: 'IN', type: 'NS', data: 'sid.example.com.' }
                - { name: '@', class: 'IN', type: 'MX', data: '10 sid.example.com.' }

                - { name: 'sid', class: 'IN', type: 'A', data: "192.168.0.1" }
                - { name: 'etch', class: 'IN', type: 'A', data: "192.168.0.2" }

                - { name: 'pop', class: 'IN', type: 'CNAME', data: 'sid' }
                - { name: 'www', class: 'IN', type: 'CNAME', data: 'sid' }
                - { name: 'mail', class: 'IN', type: 'CNAME', data: 'sid' }

      roles:
         - role: saucelabs-ansible.bind
           tags: bind


## Tags

- **apparmor**: apparmor configuration tasks.
- **configuration**: configuration tasks.
- **debug**: role variables debug task.
- **installation**: installation tasks.
- **validation**: role variables validation task.

## Test

To run the tests you will need to install:

- [tox](https://tox.readthedocs.org/)
- [vagrant](https://www.vagrantup.com/)

To run all tests against all pre-defined OS/distributions * ansible versions:

```
$ tox
```

To run tests for `trusty64`:

```
$ cd tests
$ bash test_idempotence.sh --box trusty64.vagrant.dev
# log file will be stores under tests/log
```

To perform debugging on a specific environment:

```
$ cd tests
$ vagrant up trusty64.vagrant.dev

# to provision using the test.yml playbook (as many time as you need)
$ vagrant provision trusty64.vagrant.dev

# to access the Vagrant box
$ vagrant ssh trusty64.vagrant.dev
```


## Links

- [Ubuntu 14.04 > Ubuntu Server Guide > Domain Name Service (DNS)](https://help.ubuntu.com/lts/serverguide/dns.html)
- [Debian > Wiki > Bind9](https://wiki.debian.org/Bind9)


## License

BSD


## Author Information

- [press5](https://github.com/press5/)
- [esacteksab](https://github.com/esacteksab/)
- [steenzout](https://github.com/steenzout/)
