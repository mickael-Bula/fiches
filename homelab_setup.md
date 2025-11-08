# Homelab Setup – Webtrader & Proxmox

## Table des matières
1. [Objectif du projet](#objectif-du-projet)  
2. [Architecture du homelab](#architecture-du-homelab)  
3. [Configuration réseau et Proxmox](#configuration-réseau-et-proxmox)  
4. [Mises à jour et dépôts Proxmox](#mises-à-jour-et-dépôts-proxmox)  
5. [Création d’un container LXC Debian](#création-dun-container-lxc-debian)  
6. [Connexion SSH et gestion des utilisateurs](#connexion-ssh-et-gestion-des-utilisateurs)  
7. [Installation des outils sur le LXC](#installation-des-outils-sur-le-lxc)  
8. [Configuration Apache et Virtual Hosts](#configuration-apache-et-virtual-hosts)  
9. [Déploiement depuis PhpStorm](#déploiement-depuis-phpstorm)  
10. [Clé SSH et automatisation](#clé-ssh-et-automatisation)  
11. [Notes et avancement](#notes-et-avancement)  
12. [TODO](#todo)

---

## Objectif du projet

Pour servir les données de **Webtrader**, je souhaite remplacer le scraping depuis *Investing* par une récupération via **Yahoo Finance**.  
Le projet Python `candlesticks_in_db` permet déjà cette récupération, mais utilise **HeidiSQL** pour gérer la base de données.

HeidiSQL n’étant pas très pratique, je vais gérer la base **PostgreSQL** sur un serveur dédié, hors de Laragon.  
Les données devront être accessibles depuis Webtrader.

---

## Architecture du homelab

Le **homelab** sera une machine **Debian** avec **Proxmox** installé.  
Proxmox hébergera trois environnements :

- **1 VM Debian (prod)** : PHP 8.2, npm, Python, etc.  
- **1 LXC PostgreSQL** : serveur de base de données.  
- **1 LXC Debian de développement** par projet, avec Docker pour les services spécifiques (Redis, ElasticSearch, etc.).

---

## Configuration réseau et Proxmox

**Adresse MAC du NucBox M5 Plus** :
```
Ethernet 1 : 84-47-09-67-3C-B7
Ethernet 2 : 84-47-09-67-3C-BA
```

**Configuration IP :**
```
Adresse IPv4 : 192.168.1.77 (préférée)
Passerelle : 192.168.1.1
```

**Adresse du serveur Proxmox :**
```
IP : 192.168.1.10
Hostname : homelab.local
```

Pour l’accès :
- URL directe : `https://192.168.1.10:8006`
- Ajout dans le fichier hosts Windows :
  ```
  192.168.1.10 homelab.local homelab
  ```

> 💡 Cette opération est à répéter pour chaque hostname ou virtual host Apache.

---

## Mises à jour et dépôts Proxmox

### Activer les dépôts communautaires

1. **Ouvrir les dépôts :**
   ```bash
   nano /etc/apt/sources.list.d/pve-enterprise.sources
   nano /etc/apt/sources.list.d/ceph.sources
   ```

2. **Commenter les lignes d’entreprise** et vérifier la présence du dépôt communautaire :
   ```
   Types: deb
   URIs: http://download.proxmox.com/debian/pve
   Suites: trixie
   Components: pve-no-subscription
   Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
   ```

3. **Mise à jour :**
   ```bash
   apt update
   ```

4. **Ajout d’un template Debian 13 :**
   ```bash
   pveam update
   pveam available | grep debian
   pveam download local debian-13-standard_13.1-2_amd64.tar.zst
   ```

---

## Création d’un container LXC Debian

Le container Debian servira à héberger **Webtrader**.

**Configuration principale :**
```
Node : homelab
VM ID : 101
Hostname : webtrader-dev
Unprivileged container : oui
Nesting : oui
Storage : local-lvm (20 Go)
CPU : 4
Mémoire : 4096 Mo
Swap : 4096
Adresse IP : 192.168.1.21/24
Passerelle : 192.168.1.1
DNS : 8.8.8.8, 1.1.1.1
```

**Vérification réseau :**
```bash
ip a
ip route
ping google.com
```

Le site est ensuite accessible via :  
👉 `http://webtrader.local`

---

## Connexion SSH et gestion des utilisateurs

### Génération et déploiement d’une clé SSH pour Proxmox
```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_proxmox -C "proxmox@laptop_mika"
ssh-copy-id -i ~/.ssh/id_ed25519_proxmox.pub root@192.168.1.10
ssh -i ~/.ssh/id_ed25519_proxmox root@192.168.1.10
```

### Ajout d’une clé SSH dans le LXC
Modifier `/etc/ssh/sshd_config` :
```
PermitRootLogin yes
```

Créer un utilisateur dédié :
```bash
adduser --disabled-password --gecos "" admin
usermod -aG www-data admin
chown -R admin:www-data /var/www/html
chmod -R 755 /var/www/html
```

---

## Installation des outils sur le LXC

### Composer
```bash
apt update && apt install -y wget unzip
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
rm composer-setup.php
composer install
```

### Symfony CLI
```bash
wget https://get.symfony.com/cli/installer -O - | bash
mv /root/.symfony5/bin/symfony /usr/local/bin/symfony
symfony -v
```

---

## Configuration Apache et Virtual Hosts

**Exemple de configuration :**
```apache
<VirtualHost *:80>
    ServerName mon-domaine.local
    DocumentRoot /var/www/html/public

    <Directory /var/www/html/public>
        AllowOverride All
        Require all granted
        FallbackResource /index.php
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/mon-site_error.log
    CustomLog ${APACHE_LOG_DIR}/mon-site_access.log combined
</VirtualHost>
```

Activation :
```bash
a2ensite mon-site.conf
systemctl restart apache2
apache2ctl configtest
```

Ajouter dans `C:\Windows\System32\drivers\etc\hosts` :
```
192.168.1.21 mon-domaine.local
```

---

## Déploiement depuis PhpStorm

Si des problèmes de droits apparaissent :
```bash
chown -R admin:www-data /var/www/html
chmod -R 755 /var/www/html
chmod -R 775 /var/www/html/var
chmod -R 775 /var/www/html/public
```

Ces commandes ne sont nécessaires qu’après un **nouveau déploiement complet**.

---

## Clé SSH et automatisation

Pour permettre la connexion sans mot de passe au user `admin` :
```bash
mkdir -p /home/admin/.ssh
chmod 700 /home/admin/.ssh
nano /home/admin/.ssh/authorized_keys
chmod 600 /home/admin/.ssh/authorized_keys
chown -R admin:admin /home/admin/.ssh
```

**Test de connexion :**
```bash
ssh -i ~/.ssh/id_ed25519_lxc_webtrader-dev admin@192.168.1.21
```

**Script de permissions automatisé :**
```bash
ssh -i ~/.ssh/id_ed25519_lxc_webtrader-dev admin@192.168.1.21 "/home/admin/fix_permissions.sh"
```

---

## Notes et avancement

**25/10/2025 :**
- Base `yfinance_db` conteneurisée et script Python exécuté avec succès.  
- Données chargées et persistées.  
- Visualisation possible dans PhpStorm via `docker-compose.yml` du projet `candlesticks_in_db`.  
- Le dépôt GitHub correspondant est `save_ticks_in_db`.  
- L’exécution du script `draw_candlesticks.py` génère les graphiques du CAC sur trois horizons de temps.

---

## TODO

- [ ] Adapter Webtrader pour lire les données PostgreSQL.  
- [ ] Vérifier le chargement des données LVC depuis Yahoo Finance.  
- [ ] Utiliser Stimulus pour alléger le JS.  
- [ ] Remplacer les datatables Vue.js par des versions natives.  
- [ ] Afficher les graphiques générés côté Python.  
- [ ] Créer un module de calcul des performances annuelles.  
- [ ] Planifier l’exécution quotidienne du script Python.  
- [ ] Uniformiser les environnements Python utilisés.
