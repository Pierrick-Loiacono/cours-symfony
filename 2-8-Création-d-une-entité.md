# Création d'une entité


Dans Symfony, les entités sont généralement créées avec une commande fournie par Symfony MakerBundle.
Cette commande permet de générer :

- La classe de l'entité
- Les propriétés
- Les getters et setters
- Le repository associé

---
## 1. Création

Executez cette commande dans votre terminal, à la racine de votre projet :
```bash
php bin/console make:entity
```

Symfony demandera :
- le nom de l'entité
- les propriétés à ajouter
- le type des champs

Prenons l'exemple de la création de l'entité Produit avec un champs nom de type string

`src/Entity/Produit.php`

```php
<?php

namespace App\Entity;

use App\Repository\ProduitRepository;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: ProduitRepository::class)]
class Produit
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 255)]
    private ?string $nom = null;

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
}
```

Le repository a aussi été généré dans `src/Repository/ProduitRepository`

## 2. Relation

Dans Doctrine ORM, les relations entre entités permettent de représenter les liens entre les tables en base de données. On retrouve 3 associations : **OneToMany**, **ManyToOne** et **ManyToMany** 

Les entités peuvent être liées entre elles par des associations. Chaque association sont définis par des attributs que l'on retrouve dans les entités, et c'est grâce a ces attributs que Doctrine va gérer les jointures



### OneToMany / ManyToOne

Un objet est lié à plusieurs autres objet. Dans l'exemple, un **User** possède plusieurs commandes, et inversement une commande appartient à un utilisateur

- Un **User** possède plusieurs **Commandes** et la relation est gérée par la propriété auteur dans **Commande**
- Une **Commande** appartient à un seul **User** et correspond à la collection commandes dans **User**

```php
#[ORM\Entity]
class User 
{
    // ...

    #[ORM\OneToMany(targetEntity: Commande::class, mappedBy: 'auteur')]
    private $commandes;

    // ...

    public function __construct()
    {
        $this->commandes = new ArrayCollection();
    }

    public function getCommandes(): Collection
    {
        return $this->commandes;
    }

    public function addCommande(Commande $commande): self
    {
        if (!$this->commandes->contains($commande)) {
            $this->commandes[] = $commande;
            $commande->setAuteur($this);
        }

        return $this;
    }

    public function removeCommande(Commande $commande): self
    {
        if ($this->commandes->removeElement($commande)) {
            if ($commande->getAuteur() === $this) {
                $commande->setAuteur(null);
            }
        }

        return $this;
    }
}

#[ORM\Entity]
class Commande 
{
    // ...

    #[ORM\ManyToOne(targetEntity: User::class, inversedBy: 'commandes')]
    #[ORM\JoinColumn(nullable: false)]
    private User $auteur;

    // ...

    public function getAuteur(): ?User
    {
        return $this->auteur;
    }

    public function setAuteur(?User $auteur): self
    {
        $this->auteur = $auteur;
        return $this;
    }
}
```


### ManyToMany

Plusieurs entités peuvent être liées à plusieurs autres

- Un Article peut avoir plusieurs Tags
- Un Tag peut être utilisé dans plusieurs Articles

Comment ça fonctionne ? En utilisant une table de liaison (table pivot)


#### L'attribut de l'entité Article 
```php
#[ORM\ManyToMany(targetEntity: Tag::class, inversedBy: 'articles')]
private Collection $tags;
```

#### L'attribut de l'entité Tag 
```php
#[ORM\ManyToMany(targetEntity: Article::class, mappedBy: 'tags')]
private Collection $articles;
```

Ainsi comme pour le OneToMany, il y aura une méthode associée qui sera propre a chaque entité par rapport à l'autre. Par exemple dans l'entité **Article**

```php
public function addTag(Tag $tag): self
{
    if (!$this->tags->contains($tag)) {
        $this->tags[] = $tag;
        $tag->addArticle($this);
    }

    return $this;
}
```

> IMPORTANT : la table de liaison ne contient que les id des 2 entités. Si vous souhaitez ajouter d'autres informations il faudra créer une entité intermédiaire