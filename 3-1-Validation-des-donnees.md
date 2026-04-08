# Validation des données


La validation des données consiste à vérifier que les informations reçues respectent des règles précises avant d’être utilisées ou enregistrées.
Le but est d’éviter les données incohérentes, incomplètes ou invalides.

Dans Symfony, cette validation repose en général sur le composant Validator et sur des contraintes que l’on place souvent sur les propriétés d’une entité ou d’un DTO.

Exemple d’objectifs de validation :

vérifier qu’un champ n’est pas vide
vérifier qu’un email a un format correct
vérifier qu’un nombre est dans une plage autorisée
vérifier qu’une chaîne a une longueur minimale ou maximale

La validation intervient souvent :

après la soumission d’un formulaire

---
## 1. Contraintes

On associe des règles à des propriétes directement dans l'entité. On oublie de faire les bons imports avec `use`

```php
namespace App\Entity;

use Symfony\Bridge\Doctrine\Validator\Constraints\UniqueEntity;
use Symfony\Component\Validator\Constraints as Assert;

#[UniqueEntity(fields: ['email'], message: "Cet email est déjà utilise.")]
class User
{
    #[Assert\NotBlank(message: "Le nom est obligatoire.")]
    #[Assert\Length(min: 3, max: 20, minMessage: "Minimum 3 caractères.", 
    maxMessage: "Maximum 20 caractéres")]
    private string $username;

    #[Assert\NotBlank]
    #[Assert\Email(message: "L'adresse email '{{ value }}' n'est pas valide.")]
    private string $email;

    #[Assert\NotBlank]
    #[Assert\Length(min: 6, minMessage: "Le mot de passe doit faire minimum 6 caractères.")]
    private string $password;
}

```

## 2. Contraintes les plus utilisées

| Contrainte              | Rôle                                              | Exemple                                      |
|------------------------|--------------------------------------------------|----------------------------------------------|
| **NotBlank**               | Vérifie que la valeur n’est pas vide             | ``#[Assert\NotBlank]``                           |
| **NotNull**                | Vérifie que la valeur n’est pas null             | ``#[Assert\NotNull]``                            |
| **Length**                 | Vérifie la longueur d’une chaîne                 | ``#[Assert\Length(min: 3, max: 255)]``           |
| **Email**                  | Vérifie le format d’un email                     | ``#[Assert\Email]``                              |
| **Range**                  | Vérifie qu’une valeur est dans un intervalle     | ``#[Assert\Range(min: 1, max: 100)]``            |
| **GreaterThan**           | Vérifie que la valeur est strictement supérieure | ``#[Assert\GreaterThan(0)]``                     |
| **GreaterThanOrEqual**    | Vérifie que la valeur est ≥                      | ``#[Assert\GreaterThanOrEqual(0)]``              |
| **LessThan**              | Vérifie que la valeur est strictement inférieure | ``#[Assert\LessThan(100)]``                      |
| **LessThanOrEqual**       | Vérifie que la valeur est ≤                      | ``#[Assert\LessThanOrEqual(20)]``                |
| **Regex**                  | Vérifie un format via une expression régulière   | ``#[Assert\Regex('/^[A-Z]{2}[0-9]{4}$/')]``      |
| **Choice**                 | Vérifie que la valeur est dans une liste         | ``#[Assert\Choice(['user', 'admin'])]``          |
| **Date**                   | Vérifie un format de date                        | ``#[Assert\Date]``                               |
| **DateTime**              | Vérifie un format date + heure                   | ``#[Assert\DateTime]``                           |
| **Url**                | Vérifie un format d’URL                          | ``#[Assert\Url]``                                |
| **Positive**            | Vérifie que la valeur est > 0                    | ``#[Assert\Positive]``                           |
| **PositiveOrZero**        | Vérifie que la valeur est ≥ 0                    | ``#[Assert\PositiveOrZero]``                     |

## 3. La validation côté formulaire

Lorsqu’un formulaire est envoyé, Symfony récupère les données via l’objet Request, puis applique automatiquement les règles de validation définies avec les contraintes.
La validation est donc effectuée côté serveur au moment du traitement du formulaire.

```php
...
if ($form->isSubmitted() && $form->isValid()) {
    // Si les données sont valides, on peut déclencher le traitement
}
```

#### Données invalides

Si le formulaire est invalide, les données saisies sont conservées et réaffichées
Un message d'erreur sera afficher sur les champs

`$form->handleRequest($request);` récupère les données envoyées, les injecte dans l’objet formulaire, lance la validation.
Même si c’est invalide, les données restent dans le formulaire

Les erreurs sont affichées automatiquement si vous utilisez :

```twig
{{ form(form) }}
```