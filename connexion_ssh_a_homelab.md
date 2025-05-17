
# ✅ Installation et Configuration d’un Serveur OpenSSH sur Windows (Wampserver)

## 🔐 Objectif  
Permettre une **connexion SSH à distance** à un PC Windows utilisé comme serveur (ex. avec Apache via Wampserver).

---

## 1. 🛠️ Installation du Serveur OpenSSH sur le PC Serveur

### 📦 Installer OpenSSH Server

> **Prérequis** : Windows 10/11 (version 1809 ou ultérieure)

1. Ouvrir **Paramètres > Applications > Fonctionnalités facultatives**.
2. Cliquer sur **Ajouter une fonctionnalité**.
3. Rechercher et installer **OpenSSH Server**.
4. Redémarrer le PC si demandé.

> **Remarque** : OpenSSH Client est souvent installé par défaut. Ici, il faut le **serveur**.

---

### ⚙️ Activer et Configurer le Service

1. Ouvrir le **Gestionnaire des services** (Win + R → `services.msc`)
2. Rechercher **OpenSSH SSH Server** (nom du service : `sshd`)
3. Clic droit > **Démarrer**
4. Clic droit > **Propriétés** :
   - Définir le **Type de démarrage** sur `Automatique` (ou `Automatique (début différé)`)

---

## 2. 🔧 Configurer OpenSSH

### 📝 Modifier le fichier `sshd_config`

1. Ouvrir le fichier :
   ```
   C:\ProgramData\ssh\sshd_config
   ```
2. Vérifier ou ajouter ces lignes (non commentées) :
   ```bash
   Port 22
   ListenAddress 0.0.0.0
   PasswordAuthentication yes
   PermitRootLogin no
   ```

3. Redémarrer le service :
   ```powershell
   Restart-Service sshd  # depuis PowerShell en tant qu'administrateur
   ```

---

## 3. 🌐 Vérifier la Connectivité Réseau

### 📡 Ping depuis le poste client

```bash
ping 192.168.1.10
```

> Remplacez l'adresse IP par celle du serveur.  
> Si pas de réponse, vérifier la configuration réseau (pare-feu, IP, câble...).

---

### 🔥 Configurer le Pare-feu Windows

Vérifiez que le port 22 est ouvert. Pour créer une règle manuellement :

```powershell
New-NetFirewallRule -Name "SSH" -DisplayName "OpenSSH Server (sshd)" -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

---

### 📶 Tester avec Telnet (facultatif)

> ⚠️ `telnet` n’est pas installé par défaut – à activer dans les Fonctionnalités Windows

```bash
telnet 192.168.1.10 22
```

---

## 4. 🔗 Connexion SSH depuis un autre poste

### 🖥️ Méthode 1 : PowerShell ou Invite de commandes

```bash
ssh utilisateur@192.168.1.10
```

- `utilisateur` : ton **nom d'utilisateur Windows** sur le serveur
- `192.168.1.10` : IP du serveur
- Le mot de passe demandé est celui de ta session Windows

---

### 🧰 Méthode 2 : Utiliser PuTTY

- Hôte : `192.168.1.10`
- Port : `22`
- Type de connexion : `SSH`
- Identifiants : Nom d'utilisateur + mot de passe Windows

---

## 5. 🧪 Tester la Connexion SSH Localement

Depuis le PC serveur, ouvrir PowerShell ou CMD et taper :

```bash
ssh localhost
```

> Si la connexion réussit, le service SSH est opérationnel localement.

---

## 6. 🐧 Utiliser des commandes Linux via SSH

### Option 1 : Git Bash

Si **Git for Windows** est installé :

```bash
"C:\Program Files\Git\git-bash.exe"
```

### Option 2 : Windows Subsystem for Linux (WSL)

1. Installer WSL :
   ```bash
   wsl --install -d Ubuntu
   ```
2. Une fois connecté en SSH :
   ```bash
   wsl
   ```

> Cela permet d’utiliser un shell Bash et les commandes Linux dans un environnement Windows.

---

## ✅ Résumé

| Étape | Description |
|-------|-------------|
| 1 | Installer OpenSSH Server |
| 2 | Démarrer le service `sshd` |
| 3 | Configurer le fichier `sshd_config` |
| 4 | Ouvrir le port 22 dans le pare-feu |
| 5 | Tester avec `ping`, `telnet` et `ssh` |
| 6 | Se connecter via PowerShell, PuTTY ou Git Bash |
| 7 | Utiliser Git Bash ou WSL pour un shell Linux |
