# Ansible script to migrate websites to an AlternC server

This script provides an Ansible playbook to migrate any website hosted on AlternC, Ubuntu, or Debian to a standard AlternC server.

## Migrate between two AlternC servers

Before launching a migration, you must define a `variables.yml` file.
An optional `inventory.yml` can also be defined if a default configuration must be applied to one or more AlternC servers.
The `script` command is not mandatory, but it is useful for reviewing the full process afterwards with ```less -r ansible.log```.

```script -c "ansible-playbook migrate_site.yml -i inventory.yml" ansible.log```

A minimalist `variables.yml` file could be defined as:

    ---
    source:
        domain: www.domain.tld
        cms: wordpress
        host: old.alternc.tld
        user: ssh_account
        key: /home/my/.ssh/id_rsa
        path: /var/www/[...]/www/mywebsite/
        database:
            host: 127.0.0.1
            user: "database_user"
            pass: "database_pass"
            name: "database_name"
    target:
        host: new.alternc.tld
        user: ssh_account
        key: /home/my/.ssh/id_rsa
        path: /var/www/alternc/m/mywebsite/www/
        database:
            pass: "new_database_pass"

This example provides a WordPress migration from the `old.alternc.tld` server to the `new.alternc.tld` server.
The domain associated with the CMS is `www.domain.tld`, without any aliases.
SSH connections are established to both servers using the same account `ssh_account` and the same `key`.
The AlternC account is identical on both servers.

### Inventory usage

Ansible requires [Python >= 3.9](https://docs.ansible.com/projects/ansible/latest/reference_appendices/python_3_support.html) on each server.
To use a compatible version on older Debian releases such as Buster, we suggest using `pyenv` and then installing at least Python 3.11.2.
`inventory.yml` is required to define the Python interpreter path, because Ansible autodiscovery may not handle this properly in some cases.

If the source host `source.server.tld` has a version of Python that is too old, you can run:

```bash
curl -fsSL https://pyenv.run | bash
apt install make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev curl git libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
apt-get install python3-dev default-libmysqlclient-dev build-essential pkg-config
pyenv install 3.11.2
pyenv local 3.11.2
python -V
pip install mysqlclient
```

Local python installation must be ran by ssh_account used in `variables.yml`

In your `inventory.yml` file:

    ungrouped:
        hosts:
            source.server.tld:
                ansible_python_interpreter: "[/home/ssh_account]/.pyenv/versions/3.11.2/bin/python"

## Full migration options

    ---
    source:
        domain: www.domain.tld
        #alias: [ cdn.domain.tld ]
        #ssl: true
        cms: wordpress
        #cms_path: htdocs
        #proxy: false|true (default true)
        host: old.alternc.tld
        #port: 2121
        user: ssh_account
        pass: anypassword
        key: /home/my/.ssh/id_rsa
        path: /any/absolute/path/on/server
        database:
            host: 127.0.0.1
            user: "database_user"
            pass: "database_pass"
            name: "database_name"
    target:
        host: new.alternc.tld
        port: 2222
        user: ssh_account
        pass: anypassword
        key: /home/my/.ssh/id_rsa
        path: /any/absolute/path/on/server
        alternc: new_alternc_name
        database:
            #host: 127.0.0.1
            #user: "new_database_user"
            pass: "new_database_pass"
            #name: "new_database_name"
