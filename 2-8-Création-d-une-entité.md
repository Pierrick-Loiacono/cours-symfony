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


