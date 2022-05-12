# Wildfly

[![Build Status](https://travis-ci.org/KAMI911/ansible-role-wildfly.svg?branch=master)](https://travis-ci.org/KAMI911/ansible-role-wildfly)

# Ansible Role: Installs Wildfly Java Application server (optionally with Hugepages)

Installs Wildfly Java Application server. Most complete Wildfly installation, supporting, init.d script, application naming, separated WIldfly distribuution and application.

## Table of Contents

1. [Requirements][Requirements]
2. [Installation][Installation]
3. [Role Variables][Role Variables]
4. [Dependencies][Dependencies]
5. [Example Playbook][Example Playbook]
6. [Licensing][Licensing]
7. [Author Information][Author Information]
8. [Support][Support]
9. [Admin User][Admin User]
10. [SSL Support][SSL Support]
11. [Contributing][Contributing]
12. [Donation][Donation]

## Requirements

None.

## Installation

    ansible-galaxy install kami911.wildfly

## Role Variables
It's important to change the bind addresses to localhost or internal network in
production environments. The management user is also intended for
non-production environments, so you must change these variables for production
or undefine them and the user creation task will be skipped. You can also set
the variable `wildfly_management_user_overwrite` to `no` to avoid the user
creation or override and have the correct change status.

Available variables are listed below, along with default values (see `defaults/main.yml`):

    wildfly_force_update: false

Force update when same version is already installed. Old version is always updated.

    wildfly_service_status: 'restarted'

Specify the state of service. Possible values are: reloaded, restarted, started, stopped. Default is restarted. The started and stopped state are idempotent actions that will not run commands
unless necessary. The restarted state will always bounce the service and reloaded state will always reload.


    wildfly_system_integration: false

Put Wildfly configuration files into Unix filesystem standard location (true), or put files
 into Wildfly application folders and create symlinks to binaries (false). Default: false

    wildfly_system_dc_language: en_US

Export this language as LANG and LC_ALL shell variables for the domain contorller.

    wildfly_system_language: {{ wildfly_system_dc_language }}

Export this language as LANG and LC_ALL shell variables for the domain nodes.

    wildfly_version: 24.0.1.Final

    wildfly_file_version: '17.0'

Specify Wildfly config file version. 14 = '8.0'; 15-18 = '10.0'; 20 = '13.0'; 24 = 17.0.


### User and group managemenet related options

    wildfly_manage_user: true

Ansible role will manage user, create user and group with a specified uid and gid, shell access and SSH access.

    wildfly_user: wildfly

System user who will run Wildfly application server.

    wildfly_user_id: 363

User id of Wildfly user above.

    wildfly_user_shell: ''

Specifly the Wildfly Linux user's default Shell. Can be '' or a Shell like '/bin/bash'. The role creates the required Wildfly user and group only when wildfly_manage_user is set to true.

    wildfly_group: wildfly

System user's group.

    wildfly_group_id: 363

Group id of Wildfly user above.

    wildfly_user_ssh_key: []
    # - name: username of key
    #   key:
    #   options:

Add multiple users' SSH key when wildfly_user_ssh_key is not empty ([]). The you can specify a name (display only in the file), a SSH key and SSH authorized_keys options for each SSH user.

    wildfly_base_download_url: http://download.jboss.org/wildfly
    wildfly_name: wildfly-{{ wildfly_version }}
    wildfly_download_file: "{{ wildfly_name }}.tar.gz"
    wildfly_download_validate_certs: "yes"
    wildfly_download_url: "{{ wildfly_base_download_url }}/{{ wildfly_version }}/\
                        {{ wildfly_download_file }}"
    wildfly_download_dir: /tmp

    wildfly_install_dir: /opt
    wildlfy_data: ''

Locaton of optional data folder. The data directory is available for use by services that want to store content
in the file system. It holds persistent data for services intended to survive a server restart. Serveral Wildfly
services, such as the embedded Hypersonic database instance, store data here.
Wildfly parameter name is 'jboss.server.data.dir'.

    wildfly_dir: "{{ wildfly_install_dir }}/{{ wildfly_name }}"

    wildfly_console_log_dir: "/var/log/wildfly"
    wildfly_console_log_file: "console.log"
    wildfly_console_log: "{{ wildfly_console_log_dir }}/\
                        {{ wildfly_console_log_file }}"

    wildfly_var_dir: '{{ wildfly_install_dir }}/{{ wildfly_instance_name }}/var'

Specify the Wildfly var dir for lock and pid files when wildfly_system_integration is disabled

    wildfly_conf_dir: /etc/wildfly
    wildfly_config_file: standalone.xml
    wildfly_config_path: "{{ wildfly_dir }}/standalone/configuration/\
                                    {{ wildfly_standalone_config_file }}"
    wildfly_init_dir: /etc/init.d

### Port, connection, and firewall related options

    wildfly_bind_address: 0.0.0.0
    wildfly_management_bind_address: 0.0.0.0
    wildfly_manage_http_port: 9990
    wildfly_manage_https_port: 9993
    wildfly_http_port: 8080

Base port for HTTP connection. The Wildfly's servers use HTTP base ports plus server wildfly_create_server dictionary's offset port.

    wildfly_https_port: 8443

Base port for HTTPS connection. The Wildfly's servers use HTTPS base ports plus server wildfly_create_server dictionary's offset port.

    wildfly_manage_firewalld: true

Role manages the firewalld settings of required ports.

    wildfly_enable_firewalld: false

Force install and enable firewalld service.

    wildfly_manage_firewalld_use_zone: true

Wildfly firewalld uses zones (default) or use source addresses.

### Wildfly Vault related options

    wildfly_vault_enable: false

Create Wildfly Vault when it is true.

    wildfly_vault_name: vault.store

File name of Wildfly Vault.

    wildfly_vault_path: '{{ wildfly_home }}/{{ wildfly_mode }}/configuration/store'

File path of Wildfly Vault.

    wildfly_vault_file: '{{ wildfly_vault_path }}/{{ wildfly_vault_name }}'

File location of Wildfly Vault.

    wildfly_vault_alias: myvault

    wildfly_vault_keystore_password: 'bfNpAVdIklPWcta7WA8qsx'
    wildfly_vault_keyalg: 'AES'
    wildfly_vault_keysize: 256
    wildfly_vault_iteration: 50
    wildfly_vault_salt: QTu1Mo4Z

Wildfly Vault Salt Must be 8 characters according to Wildfly Vault documentation.

    wildfly_vault:
    - block: 'ds_ExampleDS'
        name: password
        value: sa

Wildfly Vault elements. Name is the name of stored attribute. Value is the secret to store in Wildfly Vault.

    wildfly_enable_ssl: no
    wildfly_keystore_name: my.jks
    wildfly_keystore_path: "{{ wildfly_dir }}/standalone/configuration/\
                            {{ wildfly_keystore_name }}"
    wildfly_keystore_alias: my
    wildfly_keystore_password: "secret"
    wildfly_key_password: "secret"
    wildfly_application_ssl_identity: '
        <server-identities>
            <ssl>
                <keystore path="{{ wildfly_keystore_name }}"
                relative-to="jboss.server.config.dir"
                alias="{{ wildfly_keystore_alias }}"
                keystore-password="{{ wildfly_keystore_password }}"
                key-password="{{ wildfly_key_password }}"/>
            </ssl>
        </server-identities>'
    wildfly_https_listener: '
        <https-listener name="https-server" socket-binding="https"
        security-realm="ManagementRealm"/>'

    # Manually defined variables
    # wildfly_management_user: admin
    # wildfly_management_password: admin

### Standalone or Domain mode configuration

    wildfly_mode: domain

Select Wildfly configuration mode. Supported modes are standalone and domain

    wildfly_role: domain-master-slave  # supported roles: domain-master domain-master-slave domain-slave domain

In case of Wildfly domain mode, please select roles for the host. For domain mode you must have one non slave mode Wildfly host. Hence you must have at least one domain-master or domain-master-slave or domain mode host.
The difference between domain-master or domain-master-slave or domain mode are:
* domain-master: contains only domain controller. Uses one Wildfly instance.
* domain-master-slave: contains a domain controller and a separated domain slave. Uses two Wildfly instances in smart way.
* domain: contains a domain controller and an intagrated domain slave. Uses one Wildfly instance.

    wildfly_system_name: app-system

Specify the system wide name of this installation. This has same value accross multiple hosts of same domain.

    wildfly_instance_name: 'wildfly_{{ wildfly_system_name }}'

Instance name of standalone and domain slave mode Wildfly installation. For example the name of init.s and systemd scripts.

    wildfly_dc_instance_name: 'wildfly-dc_{{ wildfly_system_name }}'

Instance name of domain master mode Wildfly installation. For example the name of init.s and systemd scripts.

    wildfly_dc_backup_mode: false

Switch domain slaves to backup mode. Causes the slave host controller to create and maintain a local copy (domain.cached-remote.xml) of the domain configuration. If ignore-unused-configuration is unset in host.xml,a complete copy of the domain configuration will be stored locally, otherwise the configured value of ignore-unused-configuration in host.xml will be used.

    wildfly_dc_cached_mode: false

Switch domain slaves to cached domain controller mode. If the slave host controller is unable to contact the master domain controller to get its configuration at boot, this option will allow the slave host controller to boot and becomeoperational using a previously cached copy of the domain configuration (domain.cached-remote.xml.) If the cached configuration is not present, this boot will fail. This file is created using using one ofthe following methods:
* A previously successful connection to the master domain controller using --backup or --cached-dc.
* Copying the domain configuration from an alternative host to domain/configuration/domain.cached-remote.xml.
The unavailable master domain controller will be polled periodically for availability, and once becoming available, the slave host controller will reconnect to the master host controller and synchronize the domainconfiguration. During the interval the master domain controller is unavailable, the slave host controller will not be able make any modifications to the domain configuration, but it may launch servers and handlerequests to deployed applications etc.

#### Additional Java parameters

[Check Java parameters by versions.](https://chriswhocodes.com/vm-options-explorer.html)

### Process limits configuration

    wildfly_limits_file: '/etc/security/limits.d/{{ wildfly_user }}'

Name and location of limits file to control system resource allocations.

    wildfly_limits_conf:
     - domain: '{{ wildfly_user }}'
       type: soft
       item: nofile
       value: 14000
       # Note: systemd service file will use this value to set LimitNOFILE
       # value for node, domain controller is fixed to 2048.
     - domain: '{{ wildfly_user }}'
       type: hard
       item: nofile
       value: 16384
     - domain: '{{ wildfly_user }}'
       type: soft
       item: nproc
       value: 7000
     - domain: '{{ wildfly_user }}'
       type: hard
       item: nproc
       value: 8192

Set limits for specified user (domain). Value is the limit.
Note: systemd service file will use this value to set LimitNOFILE
value for node, domain controller is fixed to 2048.

Type is one of these:

Limit Type | Description
---------------------------------
hard       | hard limit
soft       | soft limit
–          | both hard and soft limit


Item is one of these:

Item         | Description
---------------------------------------------------------
core         | limits the core file size (KB)
data         | max data size (KB)
fsize        | maximum filesize (KB)
memlock      | max locked-in-memory address space (KB)
nofile       | max number of open file descriptors
rss          | max resident set size (KB)
stack        | max stack size (KB)
cpu          | max CPU time (MIN)
nproc        | max number of processes
as           | address space limit (KB)
maxlogins    | max number of logins for this user
maxsyslogins | max number of logins on the system
priority     | the priority to run user process with
locks        | max number of file locks the user can hold
sigpending   | max number of pending signals
msgqueue     | max memory used by POSIX message queues (bytes)
nice         | max nice priority allowed to raise to values: [-20, 19]
rtprio       | max realtime priority
chroot       | change root to directory (Debian-specific)

### Wildfly log compression options

    wildfly_log_rotate_minsize: 20K

Log files are rotated when they grow bigger than size bytes, but not before the additionally
specified time interval (daily, weekly, monthly, or yearly). The related size option is
 similar except that it is mutually exclusive with the time interval options, and it causes
log files to be rotated without regard for the last rotation time. When minsize is used,
both the size and timestamp of a log file are considered.

    wildfly_log_compress_command: '/usr/bin/xz'

Location is the compressor program. Default is xz so the dafault value is '/usr/bin/xz'.
You can also use for example: '/usr/bin/gzip'.

    wildfly_log_compress_options: '-9e'

Options for compression program. For xz the default is '-9e' that means maximum compression.
For gzip íou can use for example: '-9'.

    wildfly_log_compress_extension: '.xz'

File extension for logrotate specification. xz is '.xz', and gzip is 'gz'.

## Dependencies

None.

## Example Playbook

    - hosts: all
      roles:
        - wildfly

## Licensing

The Wildfly Ansible role application and documantations are licensed under the terms of
the MIT / BSD, you will find a copy of this license in the
[LICENSE](LICENSE) file included in the source package.

## Author Information

This role was created in 2015 by Juan Diego Romero González, additional work in 2016-2018 by Kálmán Szalai - KAMI

## Support

If you have any question, do not hesitate and drop me a line.
If you found a bug, or have a feature request, you can [fill an issue](https://github.com/KAMI911/ansible-role-wildfly/issues).

### Using as a submudule of an AWX playbook

#### Add as a submodule

```
git submodule add --force git@github.com:KAMI911/ansible-role-wildfly.git roles/wildfly
```

#### Update as sumodule

Update only this submodule

```
git submodule update --remote roles/wildfly/
```

Update all submodules:

```
git submodule foreach git pull origin master
```

## Admin User

It's recommended that you create Wildfly's admin user separately as follows:

    $ ansible-playbook main.yml --extra-vars "wildfly_management_user=admin wildfly_management_password=admin"

## SSL Support

In order to enable SSL for applications and the management interface you have
to set the `wildfly_enable_ssl` variable to `yes` and put the keystore file
into this role files folder.

You can create a self signed keystore file with the following command:

    $ keytool -genkey -alias mycert -keyalg RSA -sigalg MD5withRSA -keystore my.jks -storepass secret  -keypass secret -validity 9999

It's recommended that the first and last name is your hostname. After this file
is created, you have to set the keystore related variable in order to work
correctly.

To create a keystore with your own certificate you need to run the following commands:

    $ cat /etc/ssl/certs/ca-bundle.crt intermediate.crt > allcacerts.crt
    $ openssl verify -CAfile allcacerts.crt certificate.crt
    $ openssl pkcs12 -export -chain -CAfile allcacerts.crt -in certificate.crt -inkey private.key -out my.p12 -name my
    $ keytool -importkeystore -destkeystore my.jks -srckeystore my.p12 -srcstoretype pkcs12 -alias my

The first command will add your intermediate to the openssl's CAs; the order is
important and you may need to put the itermediate file before `ca-bundle.crt`.
The second command is to verify that your certificate is signed by a known CA,
usually if this step fails, the rest of the process will fail too.
The third command is to import the all our certificate files in pkcs12 format.
Finally we use keytool to create the keystore to be used in wildfly.

## Troubleshooting

  - If you're getting a `Connection refused` error in your browser check that
  you have the correct IP address in your `wildfly_bind_address` variable.

## Contributing

There are many ways to contribute to ansible-role-wildfly -- whether it be sending patches,
testing, reporting bugs, or reviewing and updating the documentation. Every
contribution is appreciated!

Please continue reading in the [contributing chapter](CONTRIBUTING.md).

### Fork me on Github

https://github.com/KAMI911/ansible-role-wildfly

Add a new remote `upstream` with this repository as value.

```
git remote add upstream https://github.com/KAMI911/ansible-role-wildfly.git
```

You can pull updates to your fork's master branch:

```
git fetch --all
git pull upstream HEAD
```

## Donation

If you find this useful, please consider a donation:

[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=RLQZ58B26XSLA)

<!-- TOC URLs -->
[Requirements]: #requirements
[Installation]: #installation
[Role Variables]: #role_variables
[Dependencies]: #dependencies
[Example Playbook]: #example_playbook
[Licensing]: #licensing
[Author Information]: #author_information
[Support]: #support
[Admin User]: #admin_user
[SSL Support]: #ssl_support
[Contributing]: #contributing
[Donation]: #donation
