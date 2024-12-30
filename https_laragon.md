# Activer le https dans Laragon

**NOTE** : Pour illustrer la démarche, j'utilise le virtual host `test_auth.local` déjà configuré dans Laragon, mais en http.

## Activer SSL pour le projet

### Ouvrir Laragon et activer SSL :

Cliquer sur Menu > Apache > SSL > Activer SSL pour ce projet.

Ce que Laragon fait automatiquement :

Il génère un certificat auto-signé pour le projet.
Il configure Apache pour utiliser ce certificat.
Il met à jour l’entrée du Virtual Host pour supporter HTTPS.

### Redémarrer Apache :

Cliquer sur Menu > Apache > Redémarrer pour appliquer les changements.

## Modifier le fichier Virtual Host pour rediriger HTTP vers HTTPS

Ouvrir le fichier `C:\laragon\bin\apache\httpd-vhosts.conf` dans Laragon :

Rechercher ou ajouter le Virtual Host du projet, et le modifier comme suit pour forcer la redirection de HTTP vers HTTPS :

```bash
define ROOT "C:/laragon/www/test_auth/public"
define SITE "test_auth.local"

<VirtualHost *:80>
    ServerName test_auth.local
    Redirect permanent / https://test_auth.local/
</VirtualHost>

<VirtualHost *:443>
    DocumentRoot "${ROOT}"
    ServerName ${SITE}
    ServerAlias *.${SITE}
    <Directory "${ROOT}">
        AllowOverride All
        Require all granted
    </Directory>

    SSLEngine on
    SSLCertificateFile      C:/laragon/etc/ssl/laragon.crt
    SSLCertificateKeyFile   C:/laragon/etc/ssl/laragon.key
 
</VirtualHost>
```

## Modification de sites.conf

Ouvrir sites.conf de Laragon, pour jouter la ligne suivante à la fin du fichier du projet :

```bash
test_auth=https://test_auth.local
```

VIsiter le site depuis Laragon > www > test_auth, ou avec l'URL `https://test_auth.local/`

Le certificat est automatiquement ajouté au na

Explication : la ligne `test_auth=https://test_auth.local` informe Laragon que lorsqu'on clique sur Menu > www > test_auth,
il doit ouvrir directement l'URL avec le protocole HTTPS.
