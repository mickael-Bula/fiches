# Créer un double de la Session Symfony

Pour réaliser des tests utilisant la session, il est possible de créer un double avec la méthode `MockFileSessionStorage()`.

Le principe de cette méthode permettant l'enregistrement et la lecture de données sans que pourtant une session soit lancée, est d'utiliser un fichier dont le chemin doit être passé en argument de `MockFileSessionStorage`.

Ainsi, après s'être assuré de l'existence de ce fichier, on instancie la classe `MockFileSessionStorage` avec ce fichier dans un premier temps, puis on passe le retour de cet appel en paramètre d'une nouvelle instance de `Session`.

Ne reste plus qu'à transmettre cette Session en paramètre de le la classe à tester.

Voici un exemple avec la classe Utils :

        // Je crée le répertoire destiné à accueillir les données de la session mockée
        $path = realpath('.' . '/tests');
        $fileSystem = new Filesystem();
        if (!$fileSystem->exists($path . '/mockFileSessionStorage')) {
            $fileSystem->mkdir($path . '/mockFileSessionStorage');
            $path = $path . '/mockFileSessionStorage';
        }
        // j'utilise la classe MockFileSessionStorage qui simule le comportement d'une session par enregistrement dans un fichier
        $storage = new MockFileSessionStorage($path);
        $session = new Session($storage);

        // je passe la session créée en paramètre du constructeur de ma classe Utils afin que les données y soient enregistrées
        $utils = new Utils($this->entityManager, $this->logger, $session);

        // je peux maintenant interagir avec le contenu de ma session (en fait avec les données conservées dans le fichier précédemment créé
        $results = $utils->setEntityInSession(Cac::class);
