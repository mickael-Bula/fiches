# Envoi d'email

Pour envoyer des emails, il faut utiliser mailer, fournit avec symfony --full.
Il faut configurer '.env.local' (ici avec 'mailtrap' dans la phase de dev) :

```
###> symfony/mailer ###
MAILER_DSN=smtp://1b6d6e9439f364:f773d3bae6ec1f@smtp.mailtrap.io:2525?encryption=tls&auth_mode=login
###< symfony/mailer ###
```

Dans le controller, injecter MailerInterface, puis  :

```php
// ajouter le use Symfony\Component\Mime\Email pour utiliser la classe Email pour écrire un mail simple, use Symfony\Bridge\Twig\Mime\TemplatedEmail pour quelque chose de plus élaboré

$mail = (new Email())
    ->from('expediteur@demo.test')
    ->to('destinataire@demo.test')
    ->subject('Mon beau sujet')
    ->html('<p>Ceci est mon message en HTML</p>')
;

$mailer->send($mail);
```

## problème éventuel

Si les mails ne sont pas directement envoyés (j'ai rencontré le cas lors de ma première utilisation) et que Symfony réclame un TRANSPORT_DSN, deux solutions :
- mettre en attente pour diffusion différée. Pour cela, il faut avoir une BDD configurée puis ajouter le TRANSPORT_DSN en le décommentant dans .env :

```
###> symfony/messenger ###
# Choose one of the transports below
MESSENGER_TRANSPORT_DSN=doctrine://default
# MESSENGER_TRANSPORT_DSN=amqp://guest:guest@localhost:5672/%2f/messages
# MESSENGER_TRANSPORT_DSN=redis://localhost:6379/messages
###< symfony/messenger ###
```

- une fois enregistrés en BDD, on peut utiliser une commande pour lancer l'envoi :

```bash
$ symfony console messenger:consume async [-vv]
```

Pour vérifier et accéder aux mails, il faut se conneecter à la plateforme de Mailtrap à l'adresse https://mailtrap.io/

On doit alors voir dans mailtrap les mails récupérés.

## autre solution pour l'envoi direct

Pour corriger le problème rencontré lors de ma première utilisation de mailer et permettre une distribution directe des mails, il faut ajouter ceci dans config/packages/mailer.yaml :

```
framework:
    mailer:
        dsn: '%env(MAILER_DSN)%'
        message_bus: false
```

En refusant l'utilisation par défaut d'un bus pour gérer les messages, ils sont envoyés directement.

NOTE : ne pas configurer de messenger lance une exception : le fichier de config de messenger réclame une variable d'environnement - et une BDD définie pour utiliser le messenger.
