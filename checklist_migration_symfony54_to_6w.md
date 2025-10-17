# 🧭 Checklist complète — Migration Symfony 5.4 → 6.x

## ⚙️ 1. Vérifications initiales

### ✅ Environnement
- [ ] PHP version ≥ **8.1** (tu es déjà en 8.2, parfait)
- [ ] Composer ≥ **2.5**
- [ ] Symfony CLI à jour (`symfony self:update`)
- [ ] `APP_ENV=dev` et `APP_DEBUG=1` pour voir toutes les dépréciations

### ✅ Vérifie tes dépendances
```bash
composer outdated symfony/* doctrine/* twig/*
```
- Mets à jour tout ce qui peut l’être **sans dépasser 5.4.x** :
  ```bash
  composer update symfony/* doctrine/*
  ```

---

## 🧩 2. Corriger toutes les dépréciations Symfony 5.4

Utilise :
```bash
SYMFONY_DEPRECATIONS_HELPER=weak vendor/bin/phpunit
```
et corrige **toutes les dépréciations “directes”** (celles venant de ton code, pas du vendor).

### 🪪 Cas typiques à corriger :
| Type | Ancien code | Nouveau code |
|------|--------------|---------------|
| Session | `$this->session` (service) | `$this->requestStack->getSession()` |
| Controller Annotations | `@Route`, `@Security` (Sensio) | `#[Route]`, `#[IsGranted]` |
| Event Listener args | `$event->getRequestType()` | `$event->isMainRequest()` |
| Cache | `CacheInterface::getItem()` (custom usage) | `Symfony\Contracts\Cache\CacheInterface` |
| Translation | `transChoice()` | `trans()` avec `%count%` |
| Validator | `@Assert\Length(minMessage=...)` | même annotation, mais attention aux paramètres renommés |
| Console | `$input->getOption('no-interaction')` | `$input->isInteractive()` |
| Doctrine | `getDoctrine()` (dans les contrôleurs) | `EntityManagerInterface` injecté via autowiring |

---

## 🧠 3. Migrer les bundles dépréciés ou abandonnés

| Bundle | État | Remplacement |
|---------|------|---------------|
| **SensioFrameworkExtraBundle** | ❌ Abandonné | Utiliser les **attributs natifs Symfony** (`#[Route]`, `#[IsGranted]`) |
| **DoctrineFixturesBundle** | ✅ Maintenu | Rien à faire |
| **TwigBundle** | ✅ Maintenu | Vérifie la compatibilité Twig ≥ 3.6 |
| **DoctrineMigrationsBundle** | ✅ Maintenu | Vérifie la version 3.6+ |
| **PhpUnitBridge** | ✅ Maintenu | Mettre à jour à `6.x` |

---

## 💾 4. Adapter le code applicatif

### ✅ Injection de dépendances
- Plus d’accès direct au container (`$this->container`)
- Utilise **l’autowiring** :
  ```php
  public function __construct(private LoggerInterface $logger) {}
  ```

### ✅ Événements du kernel
```diff
- if ($event->getRequestType() !== HttpKernelInterface::MASTER_REQUEST)
+ if (!$event->isMainRequest())
```

---

## 🧱 5. Doctrine ORM

- [ ] Doctrine ORM ≥ **2.15**
- [ ] `EntityManager::getClassMetadata()` renvoie un `ClassMetadata`
- [ ] Méthodes obsolètes (ex. `merge()`) à éviter
- [ ] Préparer Doctrine 4 : ne pas implémenter soi-même `ObjectManager`

> 🔧 Si tu utilises des repositories custom :  
> Assure-toi qu’ils étendent bien `ServiceEntityRepository`.

---

## 🎨 6. Twig

- [ ] Twig ≥ **3.6**
- [ ] Supprimer `{{ render(controller()) }}` → utiliser des composants ou des sous-requêtes contrôlées
- [ ] Remplacer `is defined` + `constant()` si tu manipules des enums PHP 8.1
- [ ] Vérifier les extensions Twig custom (elles doivent implémenter `RuntimeExtensionInterface` le cas échéant)

---

## 🧰 7. Tests et qualité

### ✅ PHPUnit
- [ ] PHPUnit ≥ **9.6**
- [ ] Supprimer l’usage de `setUpBeforeClass()` avec retours obsolètes
- [ ] Utiliser les assertions modernes (`assertSame`, `assertNull`, etc.)
- [ ] Adapter les mocks à `phpunit-mock-objects` intégré

### ✅ GrumPHP / PHPMD
- [ ] Adapter la config si certaines règles ne sont plus supportées avec PHP 8.2
- [ ] Vérifier que `grumphp.yml` appelle bien `vendor/bin/phpmd`
- [ ] Si tu utilises Rector dans ton CI :
  ```yaml
  tasks:
    rector:
      config: rector.php
      triggered_by: ['php']
  ```

---

## 🧩 8. Nettoyage & validation

### 🔍 Vérification des services :
```bash
bin/console lint:container
```

### 🔍 Vérification du routing :
```bash
bin/console debug:router
```

### 🔍 Vérification du cache :
```bash
bin/console cache:clear
```

### 🔍 Vérification finale :
```bash
vendor/bin/phpunit
```

---

## 🚀 9. Migration finale vers Symfony 6.x

1. Mettre à jour le framework :
   ```bash
   composer require symfony/framework-bundle:^6.4 --update-with-all-dependencies
   ```

2. Mettre à jour les autres composants Symfony :
   ```bash
   composer upgrade symfony/*
   ```

3. Supprimer les bundles obsolètes :
   ```bash
   composer remove sensio/framework-extra-bundle
   ```

4. Nettoyer et tester :
   ```bash
   bin/console cache:clear
   vendor/bin/phpunit
   ```

---

## 🧭 En résumé

| Étape | Objectif |
|-------|-----------|
| 1️⃣ | Mettre PHP & Composer à jour |
| 2️⃣ | Corriger les dépréciations Symfony 5.4 |
| 3️⃣ | Supprimer/migrer les bundles abandonnés |
| 4️⃣ | Adapter ton code (events, sessions, services) |
| 5️⃣ | Vérifier Doctrine, Twig, PHPUnit |
| 6️⃣ | Nettoyer et tester |
| 7️⃣ | Passer à Symfony 6.x proprement |
