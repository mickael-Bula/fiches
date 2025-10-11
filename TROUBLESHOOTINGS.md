## Problème de connexion SSL : échec d'appel cURL

Ce problème a été rencontré lors de l'installation d'un projet Symfony avec composer :

```bash
    Failed to download symfony/cache from dist: curl error 60 while downloading https://api.github.com/repos/symfony/cache/zipball/b209751ed25f735ea90ca4c9c969d9413a17dfee: SSL certificate problem: unable to get local issuer certificate

    Now trying to download from source
```

### Solutions possibles

#### Installer les certificats cacert.pem

1. Téléchargez la dernière version du bundle de certificats CA : https://curl.se/docs/caextract.html (ou cherchez le dernier cacert.pem).

2. Le placer dans un endroit sûr (par exemple, dans le dossier de la version de PHP utilisée).

3. Dans le fichier php.ini, s'assurer que la directive curl.cainfo et openssl.cafile pointent vers ce fichier :

```bash
curl.cainfo="C:\path\to\php\extras\ssl\cacert.pem"
openssl.cafile="C:\path\to\php\extras\ssl\cacert.pem"
```

4. Redémarrez le serveur web ou la console.

#### Désactiver momentanèment l'anti-virus

Les suites de sécurité (AVG, Avast, ESET, Norton, etc.) possèdent souvent une fonction de balayage HTTPS (HTTPS Scanning) ou de Web Shield qui intercepte les requêtes SSL/TLS 
(y compris celles de curl/Composer) pour les analyser. Cela casse la chaîne de confiance pour Composer.

##### Solution :

1. Désactiver temporairement la protection Web ou le balayage HTTPS dans les paramètres de l'antivirus/pare-feu.

2. Relancer la commande Composer (par exemple, composer install).

3. Si cela fonctionne, réactivez immédiatement votre antivirus une fois l'installation terminée.

