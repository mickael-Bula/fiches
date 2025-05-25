# 🚀 Déploiement d'une application Vue avec Docker dans WSL

Ce guide détaille toutes les étapes nécessaires pour accéder à une application Vue.js (ex. TodoList) déployée à l'aide de Docker dans un environnement WSL (Windows Subsystem for Linux), 
avec port forwarding configuré en PowerShell pour l'accès réseau local.

L'application est ainsi disponible depuis un autre poste utilisé comme serveur sur le réseau local.

---

## 📦 1. Prérequis

### Systèmes nécessaires

- Windows 10 ou 11 avec WSL2 activé
- Ubuntu installé via WSL
- docker installé dans WSL

---

## 🐧 2. Préparation de l’environnement WSL

### a. Mise à jour du système

```bash
sudo apt update && sudo apt upgrade -y
```

### b. Installation de Git et SSH

```bash
sudo apt install git openssh-client -y
```

---

## 🔑 3. Configuration des clés SSH

### a. Générer une clé SSH (si non existante)

```bash
ssh-keygen -t ed25519 -C "votre-email@example.com"
```

### b. Copier la clé publique vers GitHub

```bash
cat ~/.ssh/id_ed25519.pub
```

- Aller sur [https://github.com/settings/keys](https://github.com/settings/keys)
- Ajouter la clé publique

### c. Cloner le dépôt Git

```bash
cd ~/projects
git clone git@github.com:votre-utilisateur/todo-List-Vue.git
cd todo-List-Vue
```

---

## 🐳 4. Configuration de Docker

### a. Vérifier que Docker fonctionne dans WSL

```bash
docker --version
```

---

## 🛠️ 5. Fichiers nécessaires au conteneur

### a. `Dockerfile`

```Dockerfile
# Étape 1 : Build de l'application Vue avec Vite
FROM node:18 AS build-stage

WORKDIR /app

COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Étape 2 : Serveur Nginx
FROM nginx:stable-alpine

COPY --from=build-stage /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### b. `nginx.conf`

```nginx
server {
  listen 80;
  server_name localhost;

  root /usr/share/nginx/html;
  index index.html;

  location / {
    try_files $uri $uri/ /index.html;
  }
}
```

### c. `docker-compose.yaml` (optionnel)

```yaml
version: '3'

services:
  todolist:
    build: .
    ports:
      - "80:80"
    restart: unless-stopped
```

### d. `vite.config.js`

Vérifier que l'option **base** déclare un chemin relatif :

```js
export default defineConfig({
  // ...
  base: './',
})
```

✅ Ceci permet à Vite de générer des chemins relatifs dans index.html comme assets/index-xxxxx.js, et non /assets/index-xxxxx.js (chemin absolu qui casse dans nginx).

---

## Vérifiee que le pare-feu Windows ne bloque pas le port 80

Ouvre wf.msc (touche Windows + R → tape wf.msc → Entrée).

Va dans Règles de trafic entrant.

Vérifie s’il y a une règle qui autorise les connexions sur le port 80 (HTTP) :

Si oui : active-la si elle est désactivée.

Sinon : clique sur Nouvelle règle → Type : Port → TCP → Port 80 → Autoriser la connexion.

## ⚡ 6. Construction et lancement du conteneur

```bash
docker build -t todolist-app .
docker run -d --name todolist -p 80:80 todolist-app
```

Ou avec `docker-compose` :

```bash
docker compose up -d --build
```

---

## 🌐 7. Port Forwarding pour accès réseau local

## a. Récupérer l'IP dans WSL

```bash
ip addr show eth0
```

## b. Récupérer l'IP de Windows

```cmd
ipconfig
```

Si les IPs ne correspondent pas, il faut passer au point c pour rediriger le port de l'IP WSL vers un port Windows.

### c. Script PowerShell `port-forward.ps1`

```powershell
# port-forward.ps1
$WSL_IP = (wsl hostname -I).Trim().Split(' ')[0]
$port = 80
netsh interface portproxy add v4tov4 listenport=$port listenaddress=0.0.0.0 connectport=$port connectaddress=$WSL_IP
Write-Host "Port forwarding configuré : 0.0.0.0:$port -> $WSL_IP:$port"
```

### b. Exécuter le script PowerShell en tant qu’administrateur

```powershell
.\port-forward.ps1
```

---

## 🚀 8. Accès à l'application

### Depuis le navigateur du poste local :

```
http://localhost
```

### Depuis un autre poste du réseau :

Remplacer `192.168.x.x` par l'IP du PC hôte (vérifiable avec `ipconfig` dans PowerShell) :

```
http://192.168.x.x
```

---

## 🔁 9. Astuce : rendre le port forwarding persistant

### Ajouter une tâche planifiée qui exécute le script `port-forward.ps1` au démarrage de Windows (facultatif mais recommandé).

---

## 🧼 10. Nettoyage

### Arrêter le conteneur :

```bash
docker stop todolist
```

Ou avec `docker-compose` :

```bash
docker compose down
```

### Supprimer le conteneur :

```bash
docker rm todolist
```

---

## ✅ Résultat

✅ Le site déployé dans un conteneur Docker via WSL

✅ Le port 80 redirigé automatiquement depuis Windows vers WSL

✅ L’accès depuis un autre poste via l’adresse IP du PC serveur

Vous avez maintenant une application Vue.js déployée dans Docker via WSL, accessible sur votre réseau local depuis n’importe quel poste 🎉

Pour aller plus loin, voici quelques idées :

🔒 Sécurisation (facultatif)
Ajouter un pare-feu restreint : autoriser uniquement les IP de ton réseau local.

Activer HTTPS avec un reverse proxy comme Caddy ou Traefik, ou un certificat auto-signé si besoin.

🌐 Accès convivial
Créer une entrée DNS locale (ou modifier le hosts du poste distant) pour accéder par un nom comme http://todo.local.

🔁 Automatisation
Ajouter le script PowerShell dans une tâche planifiée au démarrage (je peux te la rédiger si tu veux).

Créer un Makefile ou script bash/sh pour builder/lancer ton conteneur automatiquement 🚀
