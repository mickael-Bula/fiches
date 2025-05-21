
# Utiliser Docker dans WSL2 (Windows Subsystem for Linux)

## 🧰 Prérequis
- Windows 10 (2004+) ou Windows 11 avec le sous-système WSL activé.
- Docker Desktop installé.

---

## ✅ Étapes complètes

### 1. Activer WSL et installer une distribution Linux

```powershell
wsl --install -d Ubuntu
```

>NOTE : Le chargement d'une distribution depuis WSL semble plus rapide et stable que depuis le Store Windows.

> ⚠️ Redémarre ton PC si nécessaire.

---

### 2. Lancer la distribution Ubuntu

```powershell
wsl -d Ubuntu
```

---

### 3. Vérifier les distributions WSL installées

```powershell
wsl --list --verbose
```

---

### 4. Activer l'intégration Docker dans Docker Desktop

1. Lance **Docker Desktop** sur Windows.
2. Clique sur ⚙️ **Settings** > **Resources** > **WSL Integration**.
3. Active le bouton à côté de ta distribution (ex. : `Ubuntu`).
4. Clique sur "Apply & Restart".

---

### 5. Tester Docker dans WSL

```bash
docker run hello-world
```

Si tout est bien configuré, tu devrais voir un message de bienvenue de Docker.

---

## 🛠️ Dépannage

### Erreur : `The command 'docker' could not be found in this WSL 2 distro`
👉 Cela signifie que l'intégration Docker n’est pas activée pour cette distribution WSL. Active-la dans Docker Desktop comme indiqué en [Étape 4](#4-activer-lintégration-docker-dans-docker-desktop).

---

## 🔗 Ressources utiles

- [Documentation officielle Docker WSL2](https://docs.docker.com/desktop/wsl/)
- [Installer WSL](https://learn.microsoft.com/fr-fr/windows/wsl/install)
