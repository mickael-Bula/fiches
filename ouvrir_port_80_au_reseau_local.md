# Utiliser un PC comme serveur sur un réseau local

La configuration suivante permet de servir des applications depuis un PC sur lequel un serveur Apache est installé.

Les applications ne sont cependant ouvertes qu'aux postes présents sur le même réseau local domestique (donc privé).
Cette limitation est un choix et peut être modifiée.

## 1. Configurer le Pare-feu Windows ⚙️

### Autoriser le Trafic Entrant sur le Port 80

- Allez dans le **Panneau de configuration** > **Système et sécurité** > **Pare-feu Windows Defender**.
- Cliquez sur **Paramètres avancés** pour ouvrir le Pare-feu Windows Defender avec sécurité avancée.
- Dans le volet de gauche, cliquez sur **Règles de trafic entrant**.
- Dans le volet de droite, cliquez sur **Nouvelle règle....**
- Sélectionnez **Port**, puis cliquez sur **Suivant**.
- Sélectionnez **TCP** et entrez le port 80 dans le champ **Ports locaux spécifiques**.
- Cliquez sur **Suivant**.
Sélectionnez **Autoriser la connexion**, puis cliquez sur **Suivant**.
Choisissez les profils auxquels la règle s'applique (Privé), puis cliquez sur **Suivant**.
Donnez un nom à la règle, par exemple, "Autoriser le trafic HTTP", puis cliquez sur **Terminer**.

## 2. Configurer WampServer pour Accepter les Connexions Externes ⚙️

### Modifier le Fichier de Configuration d'Apache

Ouvrez le fichier de configuration d'Apache (httpd.conf) situé dans le répertoire d'installation de WampServer (par exemple, C:\wamp64\bin\apache\apacheX.X.XX\conf\httpd.conf).

Recherchez la ligne suivante : `Require local`

Remplacez-la par : `Require all granted` :

```bash
  <Directory "C:/wamp64/www/votre_projet">
      Require all granted
  </Directory>
```

Cela permettra à Apache d'accepter les connexions depuis d'autres machines sur le réseau local.

### Vérifier les Fichiers de Configuration des Hôtes Virtuels

Si vous utilisez des hôtes virtuels, vérifiez les fichiers de configuration dans le répertoire C:\wamp64\bin\apache\apacheX.X.XX\conf\extra\httpd-vhosts.conf.
Assurez-vous que les directives Require all granted sont également présentes dans ces fichiers.

### Enregistrer et Fermer le Fichier :

- Enregistrez les modifications apportées au fichier httpd-vhosts.conf.

### Redémarrer les Services Apache et MySQL

- Dans WampServer, cliquez sur l'icône dans la barre des tâches et sélectionnez Redémarrer tous les services.
- Assurez-vous que les services Apache et MySQL sont bien en cours d'exécution (l'icône WampServer devrait être verte).

## 3. Trouver l'Adresse IP du PC Serveur 🔍

### Utiliser l'Invite de Commandes

- Ouvrez l'invite de commandes et tapez ipconfig.
- Notez l'adresse IPv4 de votre PC serveur (par exemple, 192.168.1.10).

## 4. Accéder à l'Application Web depuis l'Autre PC 🎯

### Utiliser un Navigateur Web

Sur l'autre PC, ouvrez un navigateur web et entrez l'adresse IP du PC serveur suivie du chemin vers votre application web (par exemple, http://192.168.1.10/votre_projet).

NOTE : ajouter `/public` au chemin vers une application Symfony si aucun virtual host n'a été configuré.
