# Entité - Doctrine - ORM



---
## 1. Entité

Une entité est une classe PHP qui représente une donnée de l'application. En général, une entité correspond à une table de la base de données, et ses propriétés correspondent aux colonnes de cette table. 
On retrouve les entités dans le dosser `/src/Entity` 

```php
#[ORM\Entity]
class Produit
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 255)]
    private ?string $nom = null;

    #[ORM\Column(type: 'text')]
    private ?string $description = null;

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getNom(): ?string
    {
        return $this->nom;
    }

    public function setNom(string $nom): static
    {
        $this->nom = $nom;
        return $this;
    }

    public function getDescription(): ?string
    {
        return $this->description;
    }

    public function setDescription(string $description): static
    {
        $this->description = $description;
        return $this;
    }
}
```

Une entité sert à :

- représenter les données de l’application
- structurer les informations en objets PHP
- faire le lien avec la base de données via Doctrine
- manipuler les données plus facilement dans le code

Elle contient des propriétés, des annotations / attributs Doctrine, des getters et des setters

## 2. Doctrine ORM

Doctrine est un ORM (Object-Relational Mapping)
Son rôle est de faire le lien entre :
- les objets PHP
- les tables relationnelles

Autrement dit, on manipule des objets en PHP et Doctrine s’occupe de traduire cela en requêtes SQL.
L'ORM est ici pour faire le pont entre la BDD et les objets, ainsi on peut manipuler des objets sans écrire de SQL, ce qui est bien pour des cas pas mal de cas. Il gérera aussi le CRUD

#### Les attributs Doctrine

| Attribut | Description |
| -------- | ------- |
| **#[ORM\Entity]** | Indique que la classe est une entité Doctrine |
| **#[ORM\Id]** | Indique la clé primaire |
| **#[ORM\GeneratedValue]** | Indique que la valeur est générée automatiquement |
| **#[ORM\Column]** | Définit une colonne en base de données. |

```php
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;
```

#### Correspondance objet / bdd

| PHP | Base de données |
| -------- | ------- |
| Classe | Table |
| Propriété | Colonne |
| Objet | Ligne |
| Entité | Enregistrement |

Par exemple, Doctrine peut traduire une entité ``Utilisateurs`` en SQL `SELECT * FROM utilisateurs;`

#### Les composants principaux de Doctrine

**Les entités** 
Représente les données de l'app (voir plus haut)


**Les repositories** 
Permettent de récupérer les données

```php
$utilisateurs = $utilisateursRepository->findAll();
```
**EntityManager**
L'EntityManager est responsable de la gestion des entités.
Il permet d'enregistrer, modifier et supprimer les données
```php
$article = new Article();
$article->setNom("Lait");

$entityManager->persist($article); 
$entityManager->flush();
```
`persist()` prépare l'enregistrement
`flush()` exécute les requêtes SQL

Doctrine exécura la requête SQL suivante : 
```sql
INSERT INTO article (nom) VALUES ('Lait');
```

**Les états d'une entité**

Une entité peut être dans plusieurs états

| État      | Description                                                                 | Exemple / Situation                          |
|-----------|-----------------------------------------------------------------------------|----------------------------------------------|
| NEW       | Entité créée mais inconnue de Doctrine                                     | `$user = new User();`                        |
| MANAGED   | Entité suivie par Doctrine (gérée par l’EntityManager)                     | `$em->persist($user);`                       |
| DETACHED  | Entité non suivie par Doctrine (plus synchronisée automatiquement)         | après `$em->clear();` ou fin de requête      |
| REMOVED   | Entité marquée pour suppression en base | `$em->remove($user);`

```php

$user = new User(); // NEW
$entityManager->persist($user); // MANAGED

$entityManager->flush(); // synchronisation avec la BDD

// L'entité est maintenant persistée en base

// Suppression de l'entité (REMOVE)
$entityManager->remove($user); // REMOVED

$entityManager->flush(); // synchronisation avec la BDD
```
**Les migrations**

Les migrations permettent de synchroniser la base de données avec les entités.

```bash
php bin/console make:migration # Crée le fichier de migration
php bin/console doctrine:migrations:migrate # Execute les fichiers de migration en attente
```
Ces commandes permettent de générer le SQL nécessaire pour modifier la structure de la base et de l'exécuer. Pensez toujours à vérifier ce qui a été généré

**Exemple du contenu d'une migration**

```php
public function up(Schema $schema): void
{
        $this->addSql('CREATE TABLE user (id INT AUTO_INCREMENT NOT NULL, email VARCHAR(180) NOT NULL, PRIMARY KEY (id))');
}

public function down(Schema $schema): void
{
        $this->addSql('DROP TABLE user');
}
```

#### Avantages de Doctrine
- Plus besoin d’écrire des requêtes SQL, ainsi on réduit le couplage entre le code et la BDD
- Productivité accrue car Doctrine gère quasiment tout
- On peut changer facilement de base de données
- Eviter les injections SQL par la gestion automatique des paramètres par Doctrine
- Jointure faites automatiquement
- Hydratation des objets

#### Désavantage de Doctrine
- Moins optimisé que du SQL brut
- Peut générer énormément de requête

#### Utilisation 

Vous avez 2 choix, soit en injection de dépendance du repository, soit via l'EntityManager

**DI Repository**

```php
public function index(UtilisateurRepository $repository)
{
    $utilisateurs = $repository->findAll();

    foreach ($utilisateurs as $utilisateur) {
        echo $utilisateur->GetNom()." - ".$utilisateur->getPrenom();
    }
}
```

**EntityManager**

```php
public function index(EntityManagerInterface $em)
{
    $utilisateurs = $em->getRepository(Utilisateurs::class)->findAll();

    foreach ($utilisateurs as $utilisateur) {
        echo $utilisateur->GetNom()." - ".$utilisateur->getPrenom();
    }
}
```

#### QueryBuilder et DQL

**DQL**

Le DQL (Doctrine Query Language) est un langage de requête utilisé par Doctrine pour interroger les entités.
Il ressemble à SQL, mais il travaille sur les objets PHP et non directement sur les tables.
```sql
SELECT u FROM App\Entity\Utilisateurs u
```

**QueryBuilder**

Le QueryBuilder permet de construire une requête  avec des méthodes PHP.
Au lieu d’écrire la requête sous forme de texte, on utilise des méthodes.

```php
$articles = $articleRepository
    ->createQueryBuilder('u')
    ->where('u.nom = :nom')
    ->setParameter('nom', 'Pierrick')
    ->getQuery()
    ->getResult();
```

On peut donc construire des requêtes dynamiquement
Exemple
```php
if ($valide) {
    $qb->andWhere('u.valide = :valide');
}
```