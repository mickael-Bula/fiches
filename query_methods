Symfony met à disposition de nombreuses méthodes pour récupérer des données en base. En voici un florilège :

```php
    public function showAll(MovieRepository $repository, GenreRepository $genreRepository, EntityManagerInterface $manager): Response
    {
        // récupération des données depuis les repositories instanciés automatiquement grâce au type hitting et à l'autowiring
        $movies = $repository->findAll();
        $genres = $genreRepository->findAll();
        
        // différentes manières de récupérer des données en base à partir de méthodes de l'entityManangerInterface
        $movie = $manager->find(Movie::class, 1);
        $genre = $manager->getRepository(Genre::class)->find(1);
        dump($movie, $genre);

        // récupération de la connexion DBAL pour faire une requête en langage SQL
        $conn = $manager->getConnection();

        // la même requête, cette fois en DQL - paramètre placé sous la forme ?n, nommé :name
        $query = $manager->createQuery('SELECT m FROM App\Entity\Movie m WHERE m.id = ?1');
        $query->setParameter(1, 5);
        $result = $query->getResult();
        dump($result);

        // et cette fois avec l'aide de l'API du DQL et l'utilisation de la méthode queryBuilder()
        $qb = $manager->createQueryBuilder();
        $result = $qb
            ->select('g')
            ->from('App:Genre', 'g')
            ->where('g.id = ?1')
            ->setParameter(1, 4)
            ->getQuery()
            ->getResult();
        dump($result);

        // préparation d'une requête avec un paramètre placé
        $sql = "SELECT * FROM genre WHERE id = ?";
        $stmt = $conn->prepare($sql);    # on peut préparer la requête en amont
        $stmt->bindValue(1, 1);  # on peut faire un bind pour assigner une valeur à une clé
        $result = $stmt->executeQuery([2])->fetchAssociative();  # ou on peut assigner la valeur directement dans executeQuery()
        dump($result);

        // J'utilise la même méthode, mais cette fois la préparation est rendue implicite par l'utilisation d'un paramètre donné en second argument. Préparation avec paramètre nommé
        $sql = "SELECT * FROM movie WHERE id = :id";
        $result = $conn->executeQuery($sql, ["id" => 3])->fetchAssociative();   # ici la requête n'est pas préparée en amont, mais passée dans executeQuery()
        dump($result);

        return $this->render('front/movie/list.html.twig', compact('movies', 'genres'));
    }
    ```
