
# Installer Docker Engine dans une distribution Ubuntu dédiée sous WSL

## Pourquoi une distro Ubuntu séparée ?

- Permet d’installer Docker Engine indépendamment de Docker Desktop.
- Évite les conflits entre Docker Desktop + intégration WSL et une installation Docker native.
- Gestion autonome du démon Docker (`dockerd`) dans cette distro.

---

## 1. Créer une nouvelle distribution Ubuntu WSL

```powershell
# Installer une nouvelle distro Ubuntu 20.04 (exemple)
wsl --install -d Ubuntu-20.04 --name Ubuntu-Docker

# OU cloner une distro existante
wsl --export Ubuntu Ubuntu.tar
wsl --import Ubuntu-Docker C:\WSL\Ubuntu-Docker Ubuntu.tar --version 2
```

## 2. Lancer la nouvelle distro

```bash
wsl -d Ubuntu-Docker
```

## 3. Installer Docker Engine

```bash
sudo apt update
sudo apt upgrade -y

sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

## 4. Démarrer le démon Docker

```bash
sudo dockerd &
# OU si systemd est actif
sudo service docker start
```

## 5. Tester Docker

```bash
docker run hello-world
```

---

## Notes importantes

- Docker Desktop et cette installation native Docker peuvent coexister, mais il faut bien gérer quel démon est lancé.
- Docker Desktop doit être fermé si tu souhaites utiliser uniquement cette deuxième distro avec Docker Engine.
- Cette méthode permet une isolation propre entre les environnements Docker sous WSL.

---
