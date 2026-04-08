# Les formulaires


Dans Symfony les formulaires sont géré grâce au composant **Form** qui nous permettra de générer un formulaire complet en PHP et utiliser Twig pour le rendu. Il sera possible d'automatiser la validation et les erreurs, mapper un formulaire à un objet, gérer la sécurité (CSRF)...

---
## 1. Création

Les formulaires seront centralisés dans le répertoire `/src/Form` et leurs noms terminent par **Type**.
Vous pouvez en créer grâce à la CLI : `php bin/console make:form`


```php
namespace App\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class ContactType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('prenom', TextType::class)
            ->add('email', EmailType::class)
            ->add('message', TextareaType::class)
            ->add('envoyer', SubmitType::class);
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            // Configure your form options here
        ]);
    }
}
```

Chaque `add()` correspond à un champ du formulaire et peut être associé à un type de
 fourni par Symfony. Si vous ne préciser pas de type, par défaut c'est `TextType`

## 2. Utilisation dans un contrôleur

Le cycle de vie d'un formulaire est gérer directement dans le contrôleur.

#### Utilisation d'un formulaire via sa propre classe formulaire
```php
namespace App\Controller;

use App\Form\ContactType;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

final class ContactController extends AbstractController
{
    #[Route('/contact', name: 'app_contact')]
    public function contact(Request $request): Response
    {
        $form = $this->createForm(ContactType::class);

        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $data = $form->getData(); // Récupération des données
            
            $this->addFlash('success', 'Formulaire soumis avec succès !');

            return $this->redirectToRoute('app_contact');
        }

        return $this->render('contact/contact.html.twig', [
            'form' => $form->createView(),
        ]);
    }
}
```
- `Request $request` : injection du service Request
- `handleRequest($request)` : lie les données envoyées au formulaire 
- `$form = $this->createForm(ContactType::class)` : Permet de détecter si le formulaire a été soumis et récupére les données POST
-  `if ($form->isSubmitted() && $form->isValid())` : Vérifie la soumission du formulaire et si celui-ci est valide
- `'form' => $form->createView()` : Crée la réprésentation HTML pour Twig

#### Utilisation d'un formulaire via le FormBuilder

**createFormBuilder()** permet de créer un formulaire directement dans le contrôleur

```php
    $form = $this->createFormBuilder()
        ->add('prenom', TextType::class)
        ->add('email', EmailType::class)
        ->add('message', TextareaType::class)
        ->add('envoyer', SubmitType::class)
        ->getForm();
```

Résumé des fonctions utilisées : 
- `createForm()` : instancie un formulaire à partir d'une classe dédiée (**FormType**)
- `createFormBuilder()` : utilise un builder fourni par Symfony (via **AbstractController**) pour créer un formulaire rapidement
- `add()` : méthode pour ajouter des champs, avec leur type, options...
- `handleRequest($request)` : lie les données envoyées au formulaire 
- `isSubmitted()` / `isValid()` : conditions pour vérifier la soumission et validité. Déclencher les **asserts** des entités
- `getData()` : récupère le contenu des champs sous forme de tableau.
- `createView()` : prépare l’objet pour la vue Twig.

## 3. Types de champs courants

| Type                    | Description                        |
|-------------------------|----------------------------------|
| `TextType`              | Champ texte simple                |
| `TextareaType`          | Zone de texte (plusieurs lignes)       |
| `EmailType`             | email (validation automatique HTML5)   |
| `PasswordType`          | mot de passe               |
| `UrlType`               | URL               |
| `ChoiceType`            | Liste déroulante ou cases à cocher |
| `CheckboxType`          | Case à cocher                    |
| `DateType`              | Choix de date                    |
| `DateTimeType`              | Date + heure               |
| `IntegerType`              | Nombre entier                    |
| `NumberType`              | Nombre décimal                  |
| `FileType`              | Upload de fichier                 |
| `SubmitType`            | Bouton de soumission             |

## 4. Affichage dans une vue Twig

Twig fournit des fonctions spécifiques pour afficher les formulaires générés. L'objectif est d'éviter d'écrire manuellement tout le HTML du formulaire.

Quand Symfony génère le formulaire les noms des champs ont cet aspect : `nom_du_formulaire[nom_du_champs]`
Exemple : `contact[nom]`

Lorsque le formulaire est soumis, les données reçues sont automatiquement organisées sous forme de tableau grâce à la structure des nom des champs

```twig
{% extends 'base.html.twig' %}

{% block title %}Contact{% endblock %}

{% block body %}

<h1>Contact</h1>

{{ form_start(form) }}
    {{ form_widget(form) }}
{{ form_end(form) }}

{% endblock %}
```

- `form_start(form)` : Génère la balise `<form>` avec la méthode et le nom
- `form_end(form)` : Ferme le formulaire et ajoute le token CSRF
- `form_widget(form)` : Rend tous les champs du formulaire 
- `form_row(form.prenom)` : Rend un champs complet (label, champ, erreur)
- `form_label(form.prenom)` : Rend seulement le label du champs
- `form_widget(form.prenom)` : Rend seulement l'input
- `form_errors(form.prenom)` : Affiche les erreurs de validation
- `form_rest(form)` : Affiche le reste des champs non rendu

**Personnalisation**

Si on avait voulu personnalisé le formulaire, on aurait pu faire comme l'exemple suivant :

```twig
{{ form_start(form) }}
    <div>
        {{ form_label(form.prenom) }}
        {{ form_widget(form.prenom) }}
        {{ form_errors(form.prenom) }}
    </div>

    <div>
        {{ form_label(form.email) }}
        {{ form_widget(form.email) }}
        {{ form_errors(form.email) }}
    </div>

    <div>
        {{ form_label(form.message) }}
        {{ form_widget(form.message) }}
        {{ form_errors(form.message) }}
    </div>

    <button type="submit">Envoyer</button>
{{ form_end(form) }}
```
> Par défaut, si vous ne rendez pas un champ, il sera rendu

## 5.  Flash messages

Les flash messages sont des messages temporaires stockés dans la sessions et qui sont affichés à l'utilisateur une seule fois, souvent après une action, tel que la soumission d'un formulaire : `$this->addFlash('success', 'Formulaire soumis avec succès !');`

Les flash messages suivent ce cycle :
- **1.** Le contrôleur ajoute un message 
- **2.** Symfony le stocke dans la session
- **3.** Après une redirection, Twig affiche le message
- **4.** Le message est supprimé automatiquement

Les types courants :

| Type | Utilisation |
| -------- | ------- |
| **success** | Action réussie |
| **error** | Erreur |
| **warning** | Avertissement |
| **info** | Information |

Vous pouvez ajouter ce code la dans votre ``base.html.twig``

```twig

{% for type, messages in app.flashes(['info', 'success', 'warning', 'danger']) %}
    {% for message in messages %}
        <div class="flash alert alert-{{ type }}" role="alert">
            {{ message }}
            <button type="button" class="btn-sm btn-close" onclick="this.parentElement.remove()" aria-label="Close"></button>
        </div>
    {% endfor %}
{% endfor %}
```

Pour pas s'embeter avec le css, vous pouvez ajouter aussi les CDN de Bootstrap pour avoir un peu de style, c'est pas important pour le cours

```html
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-sRIl4kxILFvY47J16cr9ZwB07vP4J8+LH7qKQnuqkuIAvNWLzeN8tE5YBujZqJLB" crossorigin="anonymous">

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/js/bootstrap.bundle.min.js" integrity="sha384-FKyoEForCGlyvwx9Hj09JcYn3nv7wiPVlz7YYwJrWVcXK/BmnVDxM+D2scQbITxI" crossorigin="anonymous"></script>
```

## 6. Quelques trucs sur les champs de formulaire

Chaque champ de formulaire Symfony peut être configuré avec de nombreuses options permettant de contrôler son affichage, son comportement et sa validation.

**Les options communes aux champs**

Quand on ajoute un champs dans un `FormType`, on peut lui passer un troisième paramètre : **un tableau d'options**. 

```php
    ->add('prenom', TextType::class, [
        'label' => 'Prénom',
        'required' => false,
        'help' => 'Entrez votre prénom',
        'attr' => [
            'class' => 'form-control',
            'placeholder' => 'Entrez votre prénom'
        ]
    ])
```

- **label** : Modifie le label (pas le nom du champs lui même)
- **required** : Indique, **côté front seulement**, si le remplissage champs est requis ou non
- **help** : Affiche un texte d'aide au dessus du champs
- **attr** : Permet d'ajouter des attributs HTML

**Les options liées aux données**
```php
    ->add('prenom', TextType::class, [
        'mapped' => false,
        'data' => 'Pierrick',
        'empty_data' => 'Entrez votre prénom',
        'disabled' => true
    ])
```

- **mapped** : Détermine si le champs est lié à une propriété de l'entité
- **data** : Permet de définir une valeur par défaut
- **empty_data** : Valeur utilisée si le champ est vide
- **disabled** : Désactive le champs

**Les options de validation**

Ce sont des contraintes de validation qui seront vérifiées après la soumission du formulaire. Si une erreur est détecté, une redirection vers le formulaire sera fait avec un message d'erreur

```php
use Symfony\Component\Validator\Constraints as Assert;

$builder->add('email', EmailType::class, [
    'constraints' => [
        new Assert\NotBlank(),
        new Assert\Email()
    ]
]);
```

Il existe une multitude de contrainte, en voici quelques unes :

- **Url**
- **Regex**
- **Length**
- **Type**

## 7. Formulaire lié à une entité

Il est possible d'automatiser la création / modification d'une entité en la liant directement au formulaire. Cette méthode permet de ne pas écrire toutes la récupération des champs du formulaire pour ensuite lier les données via les setters. A la soumission du formulaire, l'entité sera automatiquement remplit / mise à jour

Dans le form : 
```php
use App\Entity\User;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\OptionsResolver\OptionsResolver;

class UserType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('email', EmailType::class);
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'data_class' => User::class,
        ]);
    }
}
```

Dans le contrôleur : 
```php
$user = new User();

$form = $this->createForm(UserType::class, $user);
$form->handleRequest($request);
...
```

## 8. Validation des données (assert)

Dans Symfony, les contraintes (appelées asserts) permettent de vérifier que les données d’un objet respectent certaines règles avant d’être utilisées ou enregistrées en base de données. Le composant qui les exécute et **Symfony Validator**

Elles sont généralement définies directement dans les entités et sont automatiquement utilisées lors de la validation des formulaires. Grâce à ces contraintes, on s’assure que les données sont cohérentes, complètes et au bon format (par exemple : un email valide, un champ non vide, une longueur minimale, etc.).

```php
use Symfony\Component\Validator\Constraints as Assert;

class User
{
    #[Assert\NotBlank]
    #[Assert\Email]
    private string $email;

    #[Assert\Length(min: 6)]
    private string $password;
}
```