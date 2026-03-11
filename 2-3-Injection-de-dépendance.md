# Injection de dépendance

L'injection de dépendance est un principe de conception qui consiste à fournir les objets dont une classe a besoin, plutôt que de les créer elle-même. Une classe ne crée pas ses dépendances, elles lui sont injectées. Symfony facilite ça avec le conteneur de services et l'autowiring

Ça permet de rendre le code :
- plus modulaire
- plus maintenable
- plus testable

---

**Conteneur de service**

Le conteneur de services est un registre d'objets de l'application, chargé de les créer et de les fournir aux autres classes. La plupart des classes sont automatiquement enregistrées comme services, notamment celles du dossier `/src`

**Autowiring**

L'autowiring est un mécanisme qui permet à Symfony d'injecter automatiquement les dépendances

## Injection

**Dans la méthode**
```php
public function index(MailService $mailService)
{
}
```

**Dans le constructeur**

```php
public function __construct(UserRepository $repository)
{
    $this->repository = $repository;
}
```

> Rappel : Le constructeur est une méthode spécial appelée automatiquement lorsqu'un objet est créé, elle permet d'initialiser l'objet lors de sa création

## Pourquoi éviter les créations manuelles ?

La création manuelle d'objet rend le code plus difficile à maintenir et à tester
```php
$mailService = new MailService();
```

**Couplage fort entre les classes**

Quand une classe crée elle-même ses dépendances, elle devient fortement liée à leur implémentation
Le code dépendra alors directement du service créé. Dans ce cas, si on veut remplacer ce service par une implémentation, il faudra modifier le code ...

Avec l'injection la classe devient plus flexible

