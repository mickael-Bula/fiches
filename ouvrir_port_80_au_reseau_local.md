
# Utiliser un PC comme serveur sur un réseau local

La configuration suivante permet de servir des applications web depuis un PC sur lequel un serveur Apache est installé via WampServer.

Les applications ne sont accessibles qu'aux postes présents sur le **même réseau local domestique** (réseau privé). Cette limitation est volontaire et peut être modifiée pour une ouverture à Internet.

---

## ⚙️ 1. Configurer le Pare-feu Windows

### Autoriser le Trafic Entrant sur le Port 80

1. Allez dans **Panneau de configuration** > **Système et sécurité** > **Pare-feu Windows Defender**.
2. Cliquez sur **Paramètres avancés**.
3. Dans le volet de gauche, sélectionnez **Règles de trafic entrant**.
4. Dans le volet de droite, cliquez sur **Nouvelle règle...**.
5. Sélectionnez **Port**, puis cliquez sur **Suivant**.
6. Choisissez **TCP** et entrez `80` dans le champ **Ports locaux spécifiques**.
7. Cliquez sur **Suivant**.
8. Sélectionnez **Autoriser la connexion**, puis cliquez sur **Suivant**.
9. Cochez le profil **Privé** uniquement.
10. Donnez un nom à la règle (ex. : `Autoriser HTTP`) et cliquez sur **Terminer**.

**Astuce :** si vous souhaitez également autoriser HTTPS, répétez la procédure pour le port 443.

---

## ⚙️ 2. Configurer WampServer pour Accepter les Connexions Externes

### Modifier le Fichier de Configuration Apache (`httpd.conf`)

1. Ouvrez `C:\wamp64\bin\apache\apacheX.X.XX\conf\httpd.conf`.
2. Recherchez toutes les occurrences de `Require local` et remplacez-les par :

```apache
Require all granted
```

3. Par exemple, pour un projet Symfony situé dans `www/votre_projet` :

```apache
<Directory "C:/wamp64/www/votre_projet/public">
    AllowOverride All
    Require all granted
</Directory>
```

### Vérifier les Hôtes Virtuels (`httpd-vhosts.conf`)

1. Ouvrez le fichier : `C:\wamp64\bin\apache\apacheX.X.XX\conf\extra\httpd-vhosts.conf`.
2. Assurez-vous que chaque VirtualHost contient `Require all granted`.

### Redémarrer les Services Wamp

- Cliquez sur l'icône WampServer dans la barre des tâches.
- Choisissez **Redémarrer tous les services**.
- L'icône Wamp doit devenir **verte**.

---

## 🔍 3. Trouver l'Adresse IP du PC Serveur

1. Ouvrez l’invite de commandes (Win + R → `cmd` → Entrée).
2. Tapez :

```bash
ipconfig
```

3. Repérez l’adresse **IPv4** dans l’adaptateur réseau utilisé (ex : `192.168.1.10`).

---

## 🎯 4. Accéder à l'Application Web depuis un Autre PC

1. Sur un autre PC du réseau, ouvrez un navigateur web.
2. Saisissez l’adresse IP du serveur, suivie du chemin vers votre projet, par exemple :

```
http://192.168.1.10/votre_projet
```

**Note (Symfony)** : si vous utilisez Symfony sans virtual host, accédez à l'application via :

```
http://192.168.1.10/votre_projet/public
```

---

## ✅ Vérifications et Conseils

- **Dépannage :** Si l’accès échoue, vérifiez :
  - Que le pare-feu autorise bien le port 80.
  - Que l’adresse IP est correcte et accessible depuis le poste client (testez avec `ping`).
  - Que le navigateur n’ajoute pas automatiquement `https://` (supprimez le `s`).

- **Sécurité :** Cette configuration est suffisante pour un usage local sécurisé. Ne l’exposez pas à Internet sans configuration avancée (proxy, certificats, sécurité d’accès).

---

© Guide généré avec ❤️
