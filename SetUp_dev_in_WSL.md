# Configuration de l'environnement de développement WSL
## Fiche de référence – PC Windows 11

**Date :** 31 octobre 2026  
**Environnement de référence :** PC Windows 11 – WSL2 Ubuntu 24.04  
**Objectif :** reproduire cette configuration sur le PC portable.

---

# 1. Architecture générale

L'environnement de développement repose sur Windows 11 et WSL2.

```text
Windows 11
│
├── PhpStorm 2026.2.3
│
├── Ollama
│   ├── qwen3-coder:30b
│   └── qwen2.5-coder:14b
│
└── WSL2 – Ubuntu 24.04
    │
    ├── PHP 8.3
    ├── Composer
    ├── Symfony CLI
    ├── Node.js / npm / NVM
    ├── Python / uv
    ├── Git
    ├── Ansible
    ├── OpenCode
    ├── Xdebug
    └── Projets
        └── /home/bulam/projects
```

Les projets sont stockés dans le système de fichiers Linux de WSL, et non dans `/mnt/c`.

# 2. Configuration de WSL2

## Installation

- WSL2 installé et à jour.
- Ubuntu 24.04 LTS.
- Utilisateur Linux : `bulam`.
- Répertoire personnel : `/home/bulam`.
- Shell : Bash.
- Systemd activé.

Vérification :

```bash
cat /etc/os-release
wsl --status
wsl -l -v
```

## Configuration `/etc/wsl.conf`

```ini
[boot]
systemd=true

[user]
default=bulam

[interop]
appendWindowsPath=false
```

Le paramètre `appendWindowsPath=false` empêche l'injection automatique des chemins Windows dans le PATH Linux.

Après modification :

```powershell
wsl --shutdown
```

## Organisation des répertoires

```text
/home/bulam/
├── .ssh/
├── .nvm/
├── .local/
├── projects/
└── tools/
    └── ansible/
```

Ne pas modifier le comportement de démarrage du terminal : Ubuntu doit démarrer dans `/home/bulam`, sans changement automatique de répertoire dans `.bashrc`.

# 3. Outils installés dans WSL

| Outil | Version constatée sur le PC |
|---|---|
| Ubuntu | 24.04.2 LTS |
| PHP | 8.3.6 |
| Composer | 2.10.3 |
| Symfony CLI | 5.20.0 |
| Git | 2.43.0 |
| Node.js | 18.20.8 |
| npm | 10.8.2 |
| NVM | 0.39.7 |
| Python système | 3.12.3 |
| Python uv | 3.12.14 |
| uv | 0.12.17 |
| Ansible | 14.4.0 |
| ansible-core | 2.21.4 |
| Xdebug | 3.2.0 |
| OpenCode | 1.18.34 |

## Installation et gestion des outils

- PHP installé directement dans Ubuntu.
- Composer installé dans `~/.local/bin`.
- Symfony CLI installé dans `/usr/bin`.
- Node.js géré avec NVM.
- Python géré avec uv.
- Ansible installé dans un environnement virtuel dédié.

NVM est initialisé dans `.bashrc` :

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

Environnement Ansible :

```text
~/tools/ansible/.venv
```

Environnement Python de projet :

```text
~/projects/python/.venv
```

Ne pas installer pip ou Ansible globalement via apt.

# 4. Configuration Git

Git est installé dans WSL, indépendamment de Git Windows.

Configuration globale :

```bash
git config --global user.name "Nom utilisateur"
git config --global user.email "adresse@email"
```

Vérification :

```bash
git config --global --list
git --version
```

L'authentification GitHub se fait par SSH.

Une clé SSH propre à chaque machine doit être créée :

```bash
ssh-keygen -t ed25519 -C "adresse@email"
```

Afficher la clé publique :

```bash
cat ~/.ssh/id_ed25519.pub
```

Ajouter cette clé publique au compte GitHub.

Tester :

```bash
ssh -T git@github.com
```

**Important :** ne jamais copier la clé privée du PC sur le portable. Chaque machine possède sa propre identité SSH.

# 5. PhpStorm et Xdebug

PhpStorm est installé nativement sous Windows.

Le projet est ouvert directement depuis WSL :

```text
\\wsl.localhost\Ubuntu\home\bulam\projects\
```

## Interpréteur PHP

Dans PhpStorm :

- Type : WSL.
- Distribution : Ubuntu.
- Exécutable PHP : `/usr/bin/php`.
- Version : PHP 8.3.6.
- Xdebug : 3.2.0.

## Configuration Xdebug dans WSL

Fichier :

```text
/etc/php/8.3/cli/php.ini
```

Configuration :

```ini
zend_extension=xdebug.so

[xdebug]
xdebug.mode=develop,debug
xdebug.start_with_request=yes
xdebug.client_host=172.30.48.1
xdebug.client_port=9003
xdebug.discover_client_host=false
```

L'adresse `172.30.48.1` correspond à la passerelle Windows vue depuis le réseau WSL du PC. Elle devra être redéterminée sur le portable.

## Configuration PhpStorm

Serveur :

```text
Name       : WSL
Host       : localhost
Port       : 80
Debugger   : Xdebug
```

Mapping du projet :

```text
Local:
\\wsl.localhost\Ubuntu\home\bulam\projects\xdebug-test

Server:
 /home/bulam/projects/xdebug-test
```

Le chemin distant doit être un chemin Linux, et non un chemin UNC Windows.

Dans `.bashrc` :

```bash
export PHP_IDE_CONFIG="serverName=WSL"
```

Cette variable est nécessaire dans la configuration CLI testée pour que PhpStorm applique le mapping.

## Pare-feu Windows

Règle entrante :

```text
Nom           : WSL Xdebug 9003
Protocole     : TCP
Port local    : 9003
Adresse source: 172.30.48.0/20
Action        : Allow
```

Sur le portable, adapter la plage réseau à la configuration WSL réellement constatée.

# 6. Ollama sous Windows et OpenCode dans WSL

## Ollama

Ollama est installé nativement sous Windows afin d'utiliser le GPU NVIDIA.

Modèles présents :

```text
qwen3-coder:30b
qwen2.5-coder:14b
```

Le serveur Ollama écoute sur le port :

```text
11434
```

## Accès depuis WSL

L'adresse utilisée et validée est :

```text
http://host.docker.internal:11434
```

Tests :

```bash
curl http://host.docker.internal:11434/api/tags
curl http://host.docker.internal:11434/v1/models
```

Les deux modèles doivent apparaître.

## OpenCode

OpenCode est installé dans WSL.

Version constatée :

```bash
opencode --version
```

Configuration globale :

```text
~/.config/opencode/opencode.json
```

Le fournisseur Ollama est configuré pour utiliser :

```text
http://host.docker.internal:11434/v1
```

Les modèles utilisés sont :

```text
qwen3-coder:30b
qwen2.5-coder:14b
```

Le paramètre `num_ctx` a dû être augmenté pour permettre leur utilisation avec OpenCode.

**À reproduire sur le portable :**
- installer Ollama sous Windows ;
- télécharger les mêmes modèles ;
- vérifier l'accès depuis WSL ;
- installer OpenCode dans WSL ;
- reproduire la configuration fonctionnelle du PC, notamment le contexte des modèles.

Ne pas supposer que la configuration GPU et la mémoire disponible seront identiques entre les deux machines.

# 7. Connexion SSH au homelab

Le PC possède sa propre clé SSH.

Les connexions validées comprennent :

```bash
ssh admin@192.168.1.23
ssh admin@192.168.1.31
ssh root@192.168.1.10
```

Correspondances :

| Adresse | Équipement |
|---|---|
| 192.168.1.10 | Proxmox / homelab |
| 192.168.1.23 | LXC 120 – Webtrader |
| 192.168.1.31 | VM 202 – Docker |

Sur le portable :
- créer une nouvelle clé SSH ;
- installer les clés publiques sur les équipements concernés ;
- vérifier les connexions ;
- ne pas recopier les clés privées du PC.

# 8. Architecture homelab et PostgreSQL

Le homelab conserve les services partagés.

```text
Proxmox
│
├── LXC 102 – PostgreSQL
│   └── 192.168.1.55
│
├── LXC 103 – cloudflared / API relay
│   └── 192.168.1.34
│
├── LXC 120 – Webtrader
│   └── 192.168.1.23
│
└── VM 202 – Docker
    └── 192.168.1.31
```

## Bases de données

PostgreSQL 16.11 est centralisé dans le LXC 102.

Les environnements de développement disposent de bases séparées :

```text
PostgreSQL LXC 102
│
├── yfinance
│   └── développement portable
│
└── webtrader_dev_pc
    └── développement PC
```

La base `webtrader_dev_pc` a été créée comme copie complète de `yfinance` avec `CREATE DATABASE ... TEMPLATE`.

Le portable doit conserver sa propre configuration et sa base dédiée.

Ne pas partager les écritures entre les deux environnements de développement.

# 9. Projet Webtrader

Dépôt GitHub :

```text
https://github.com/mickael-Bula/webtrader_assetmapper.git
```

Emplacement WSL :

```text
~/projects/webtrader_assetmapper
```

Installation réalisée :

```bash
git clone git@github.com:mickael-Bula/webtrader_assetmapper.git
cd ~/projects/webtrader_assetmapper
composer install
```

L'installation Composer a réussi.

Le projet utilise Symfony 7.4 et accepte PHP >= 8.2.

La machine de développement PC utilise PHP 8.3.6.

## Variables d'environnement

Chaque machine possède son propre fichier `.env.local`, non versionné.

Il contient notamment sa propre `DATABASE_URL`.

Exemple conceptuel pour le PC :

```dotenv
DATABASE_URL="postgresql://webtrader_pc:...@192.168.1.55:5432/webtrader_dev_pc"
```

Ne jamais versionner les secrets ou partager les mots de passe dans Git.

# 10. Synchronisation entre les deux machines

GitHub est le point de synchronisation du code.

Avant de commencer :

```bash
cd ~/projects/webtrader_assetmapper
git pull
```

Pendant le développement :

```bash
git status
```

En fin de session :

```bash
git add .
git commit -m "Description des modifications"
git push
```

Sur l'autre machine :

```bash
git pull
```

Les deux environnements restent indépendants pour :
- leurs clés SSH ;
- leurs configurations PhpStorm ;
- leurs fichiers `.env.local` ;
- leurs bases PostgreSQL ;
- leurs installations locales et leurs modèles Ollama.

Le code commun est synchronisé par Git.

# 11. Principes d'architecture à conserver

1. Les projets vivent dans le filesystem Linux de WSL.
2. PHP, Composer, Symfony, Node, Python, Git, Ansible et OpenCode sont gérés dans WSL.
3. PhpStorm reste une application Windows native.
4. Ollama reste natif Windows pour exploiter le GPU.
5. PostgreSQL reste centralisé dans le homelab.
6. Chaque machine dispose de sa propre base de développement.
7. GitHub est la source de vérité du code.
8. Chaque machine possède ses propres clés SSH.
9. Les secrets ne sont jamais versionnés.
10. Le homelab héberge les services, mais n'est pas le poste de développement.

---

# 12. Vérification finale du portable

- [ ] WSL2 installé et à jour.
- [ ] Ubuntu 24.04 installé.
- [ ] Utilisateur Linux créé.
- [ ] `/etc/wsl.conf` configuré.
- [ ] PATH Windows non injecté dans WSL.
- [ ] PHP installé et vérifié.
- [ ] Composer installé.
- [ ] Symfony CLI installé.
- [ ] Node.js/NVM installés.
- [ ] Python/uv installés.
- [ ] Ansible installé dans son environnement virtuel.
- [ ] Git configuré.
- [ ] Clé SSH GitHub propre au portable créée.
- [ ] Connexion SSH au homelab validée.
- [ ] PhpStorm connecté à WSL.
- [ ] Xdebug configuré et testé.
- [ ] Ollama installé sous Windows.
- [ ] Modèles Qwen installés.
- [ ] Accès Ollama depuis WSL validé.
- [ ] OpenCode installé et configuré.
- [ ] Dépôt Webtrader cloné.
- [ ] `composer install` exécuté.
- [ ] `.env.local` configuré pour la base du portable.
- [ ] Connexion PostgreSQL testée.
- [ ] Application Symfony démarrée localement.
- [ ] Cycle Git pull / commit / push validé.

**Important :** cette fiche décrit l'environnement de référence du PC. Les versions indiquées sont les versions constatées lors de sa configuration ; elles constituent un point de départ, pas une obligation de figer éternellement tous les logiciels à ces versions.
