# Script Ansible pour migrer des sites web vers un serveur AlternC

Ce script fournit un playbook Ansible pour migrer n’importe quel site web hébergé sur AlternC, Ubuntu ou Debian vers un serveur AlternC standard.

## Migrer entre deux serveurs AlternC

Avant de lancer une migration, vous devez définir un fichier `variables.yml`.
Un fichier `inventory.yml` optionnel peut également être défini si une configuration par défaut doit être appliquée à l'un des 2 serveurs.
La commande `script` n’est pas obligatoire, mais elle est utile pour relire ensuite l’ensemble du processus avec `less -r ansible.log`.

`script -c "ansible-playbook migrate_site.yml -i inventory.yml" ansible.log`

Un fichier `variables.yml` minimaliste peut être défini comme suit :

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

Cet exemple montre une migration WordPress du serveur `old.alternc.tld` vers le serveur `new.alternc.tld`.
Le domaine associé au CMS est `www.domain.tld`, sans alias.
Les connexions SSH sont établies vers les deux serveurs avec le même compte, `ssh_account`, et la même `key`.
Le compte AlternC est identique sur les deux serveurs.

### Usage de l’inventaire

Ansible nécessite [Python >= 3.9](https://docs.ansible.com/projects/ansible/latest/reference_appendices/python_3_support.html) sur chaque serveur.
Pour utiliser une version compatible sur d’anciennes versions de Debian comme Buster, nous suggérons d’utiliser `pyenv`, puis d’installer au minimum Python 3.11.2.
Le fichier `inventory.yml` est nécessaire pour définir le chemin de l’interpréteur Python, car l’autodécouverte d’Ansible peut ne pas fonctionner dans certaines situations.

Si l’hôte source `source.server.tld` utilise une version de Python trop ancienne, vous pouvez exécuter :

```bash
curl -fsSL https://pyenv.run | bash
apt install make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev curl git libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
apt-get install python3-dev default-libmysqlclient-dev build-essential pkg-config
pyenv install 3.11.2
pyenv local 3.11.2
python -V
pip install mysqlclient
```

Dans votre fichier `inventory.yml` :

    ungrouped:
        hosts:
            marsup.marsnet.org:
                ansible_python_interpreter: "/root/.pyenv/versions/3.11.2/bin/python"

## Options complètes de migration

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
