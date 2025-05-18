
# 🔐 Configuration de l'authentification SSH par clé entre deux postes Windows

## ✅ Prérequis

- Deux machines Windows (client et serveur)
- OpenSSH installé sur les deux
- Accès administrateur
- Réseau fonctionnel entre les deux

---

## 1. 📦 Vérifier l’installation d’OpenSSH

Sur chaque machine, exécuter PowerShell en admin :

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

Si nécessaire, installer :

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

---

## 2. ▶️ Activer et configurer le service SSH (serveur uniquement)

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

---

## 3. 🔑 Générer une paire de clés SSH sur le client

Dans PowerShell ou Git Bash :

```bash
ssh-keygen -t ed25519 -C "clé pour serveur Windows"
```

Chemin par défaut :  
- Clé privée : `C:\Users\<Nom>\.ssh\id_ed25519`  
- Clé publique : `id_ed25519.pub`

---

## 4. 📋 Copier la clé publique sur le serveur

### ➤ Utilisateur standard :
Copier le contenu dans :

```
C:\Users\<Utilisateur>\.ssh\authorized_keys
```

### ➤ Utilisateur administrateur :
Copier le contenu dans :

```
C:\ProgramData\ssh\administrators_authorized_keys
```

Créer le fichier s’il n’existe pas.

---

## 5. 🔐 Régler les permissions

### ➤ Pour `authorized_keys` (utilisateur standard) :

```powershell
icacls "C:\Users\<Nom>\.ssh\authorized_keys" /inheritance:r
icacls "C:\Users\<Nom>\.ssh\authorized_keys" /grant "<Nom>:(R)"
```

### ➤ Pour `administrators_authorized_keys` :

```powershell
icacls "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r
icacls "C:\ProgramData\ssh\administrators_authorized_keys" /grant "Administrateurs:(R)"
```

---

## 6. ⚙️ Modifier le fichier de configuration sshd_config

Ouvrir avec Notepad en admin :

```powershell
notepad "C:\ProgramData\ssh\sshd_config"
```

Ajouter ou modifier les lignes suivantes :

```
PubkeyAuthentication yes
PasswordAuthentication no
Match Group administrators
    AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

ℹ️ Pour un utilisateur standard, utiliser :
```
AuthorizedKeysFile .ssh/authorized_keys
```

Puis redémarrer :

```powershell
Restart-Service sshd
```

---

## 7. 🔁 Tester la connexion

Depuis le client :

```bash
ssh <Utilisateur>@<IP_ou_nom_du_serveur>
```

Par exemple :

```bash
ssh Administrateur@192.168.1.42
```

---

## ✔️ Optionnel : désactiver l’authentification par mot de passe

Cela se fait déjà à l’étape 6 avec :

```
PasswordAuthentication no
```

---

## 🎉 Résultat

Vous pouvez maintenant accéder à votre serveur Windows via SSH en utilisant uniquement une clé, de manière sécurisée.
