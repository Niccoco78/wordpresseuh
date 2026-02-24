# Ansible Role — WordPress

Rôle Ansible pour le **déploiement automatisé de WordPress** avec Apache et MariaDB sur des conteneurs Docker (Ubuntu et Rocky Linux) **sans systemd**.

## Description

Ce rôle transforme le script `Install_wordpress.sh` en un rôle Ansible propre, maintenable et idempotent. Il effectue les étapes suivantes :

1. **Installation des paquets** — Apache, PHP, MariaDB, wget, unzip (avec adaptation Debian/RedHat)
2. **Configuration MariaDB** — Démarrage, sécurisation, création de la base et de l'utilisateur WordPress
3. **Déploiement WordPress** — Téléchargement, extraction et configuration via template Jinja2
4. **Configuration Apache** — VirtualHost (Ubuntu) ou httpd.conf (Rocky), activation de `mod_rewrite`

## Plateformes supportées

| OS | Famille | Paquets |
|----|---------|---------|
| Ubuntu | Debian | `apache2`, `php`, `libapache2-mod-php`, `php-mysql`, `mariadb-server` |
| Rocky Linux | RedHat | `httpd`, `php`, `php-mysqlnd`, `mariadb-server` |

## Variables

### Variables configurables (`defaults/main.yml`)

| Variable | Défaut | Description |
|----------|--------|-------------|
| `wp_db_name` | `wordpress` | Nom de la base de données |
| `wp_db_user` | `example` | Utilisateur de la base |
| `wp_db_password` | `examplePW` | Mot de passe de l'utilisateur |
| `wp_db_root_password` | `examplerootPW` | Mot de passe root MariaDB |
| `wp_site_url` | `http://localhost` | URL du site |
| `wp_web_root` | `/var/www/html` | Racine web Apache |
| `wp_download_url` | `https://wordpress.org/latest.zip` | URL de téléchargement WordPress |
| `wp_tmp_dir` | `/tmp` | Répertoire temporaire |

### Variables internes (`vars/main.yml`)

Les variables suivantes sont automatiquement définies selon `ansible_os_family` :
- `wordpress_packages` — Liste des paquets à installer
- `wordpress_apache_service` — Nom du service Apache (`apache2` / `httpd`)
- `wordpress_apache_user` — Utilisateur Apache (`www-data` / `apache`)
- `wordpress_apache_group` — Groupe Apache (`www-data` / `apache`)

## Structure du rôle

```
ansible-role-wordpress/
├── defaults/
│   └── main.yml          # Variables par défaut configurables
├── handlers/
│   └── main.yml          # Handlers (restart apache, restart mariadb)
├── meta/
│   └── main.yml          # Métadonnées Ansible Galaxy
├── tasks/
│   ├── main.yml          # Orchestrateur principal
│   ├── install.yml       # Installation des paquets
│   ├── mariadb.yml       # Configuration MariaDB
│   ├── wordpress.yml     # Déploiement WordPress
│   └── apache.yml        # Configuration Apache
├── templates/
│   ├── wp-config.php.j2  # Template wp-config.php
│   └── wordpress.conf.j2 # Template VirtualHost Apache
├── tests/
│   └── test.yml          # Playbook de test
├── vars/
│   └── main.yml          # Variables OS-spécifiques
└── README.md             # Cette documentation
```

## Handlers

| Handler | Description |
|---------|-------------|
| `restart apache` | Redémarre Apache (`service apache2 restart` ou `httpd -k restart`) |
| `restart mariadb` | Redémarre MariaDB (`mysqld_safe`) |

## Boucles et conditions utilisées

- **`loop`** : Installation des paquets via une boucle sur `wordpress_packages`
- **`loop`** : Application des permissions sur les répertoires WordPress
- **`when: ansible_os_family == "Debian"`** : Tâches spécifiques Ubuntu
- **`when: ansible_os_family == "RedHat"`** : Tâches spécifiques Rocky Linux

## Utilisation

### Exemple de playbook

```yaml
---
- name: Déploiement WordPress
  hosts: webservers
  become: yes
  roles:
    - role: ansible-role-wordpress
      vars:
        wp_db_name: monsite
        wp_db_user: admin
        wp_db_password: MonMotDePasse123
        wp_db_root_password: RootSecure456
```

### Exécution

```bash
ansible-playbook test-wordpress.yaml
```

## Prérequis

- Ansible >= 2.14
- Accès SSH aux machines cibles
- Conteneurs Docker avec SSH activé (lab ftutorials)

## Auteur

nico

## Licence

MIT
