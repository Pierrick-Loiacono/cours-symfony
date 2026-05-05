# Connexion Utilisateur avec Symfony

## Introduction

Symfony propose un système d'authentification robuste via le composant **Security**. On verra l'implémentation complète d'une connexion utilisateur : de l'installation à la gestion des sessions.

---

## 1. Prérequis

- Une base de données configurée dans votre fichier d'environnement

```bash
composer require symfony/security-bundle
composer require symfony/orm-pack
```

---

## 2. Créer l'entité User

Symfony fournit un maker pour générer l'entité utilisateur :

```bash
php bin/console make:user
```

Cette commande génère la classe `User` qui implémente `UserInterface` et `PasswordAuthenticatedUserInterface`.

```php
// src/Entity/User.php

namespace App\Entity;

use App\Repository\UserRepository;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface;
use Symfony\Component\Security\Core\User\UserInterface;

#[ORM\Entity(repositoryClass: UserRepository::class)]
#[ORM\Table(name: '`user`')]
#[ORM\UniqueConstraint(name: 'UNIQ_IDENTIFIER_EMAIL', fields: ['email'])]
class User implements UserInterface, PasswordAuthenticatedUserInterface
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 180)]
    private ?string $email = null;

    /**
     * @var list<string> Les rôles de l'utilisateur
     */
    #[ORM\Column]
    private array $roles = [];

    #[ORM\Column]
    private ?string $password = null;

    public function getId(): ?int { return $this->id; }

    public function getEmail(): ?string { return $this->email; }
    public function setEmail(string $email): static
    {
        $this->email = $email;
        return $this;
    }

    // Identifiant utilisé par Symfony pour l'authentification
    public function getUserIdentifier(): string
    {
        return (string) $this->email;
    }

    public function getRoles(): array
    {
        $roles = $this->roles;
        $roles[] = 'ROLE_USER'; // Tous les utilisateurs ont ce rôle par défaut
        return array_unique($roles);
    }

    public function setRoles(array $roles): static
    {
        $this->roles = $roles;
        return $this;
    }

    public function getPassword(): ?string { return $this->password; }
    public function setPassword(string $password): static
    {
        $this->password = $password;
        return $this;
    }

    public function eraseCredentials(): void
    {
        // Effacer les données sensibles temporaires si nécessaire
    }
}
```

---

## 3. Migrer la base de données

```bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

---

## 4. Configurer le Security Bundle

Le fichier `config/packages/security.yaml` est le cœur du système d'authentification.

```yaml
# config/packages/security.yaml

security:
    password_hashers:
        Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface: 'auto'

    providers:
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email

    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false

        main:
            lazy: true
            provider: app_user_provider

            form_login:
                login_path: app_login          # Route du formulaire de connexion
                check_path: app_login          # Route de traitement du formulaire
                enable_csrf: true              # Protection CSRF activée
                default_target_path: app_home  # Redirection après connexion

            logout:
                path: app_logout
                target: app_login              # Redirection après déconnexion

    # access_control:
    #     - { path: ^/login, roles: PUBLIC_ACCESS }
    #     - { path: ^/admin, roles: ROLE_ADMIN }
    #     - { path: ^/, roles: ROLE_USER }
```

> **Note** : `enable_csrf: true` est fortement recommandé en production pour protéger le formulaire de connexion contre les attaques CSRF.

---

## 5. Générer le formulaire de connexion

```bash
php bin/console make:security:form-login
```

Cela génère automatiquement :
- `SecurityController.php`
- `login.html.twig`

### SecurityController généré

```php
// src/Controller/SecurityController.php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;
use Symfony\Component\Security\Http\Authentication\AuthenticationUtils;

class SecurityController extends AbstractController
{
    #[Route(path: '/login', name: 'app_login')]
    public function login(AuthenticationUtils $authenticationUtils): Response
    {
        // Récupérer l'erreur de connexion s'il y en a une
        $error = $authenticationUtils->getLastAuthenticationError();

        // Dernier email saisi par l'utilisateur
        $lastUsername = $authenticationUtils->getLastUsername();

        return $this->render('security/login.html.twig', [
            'last_username' => $lastUsername,
            'error' => $error,
        ]);
    }

    #[Route(path: '/logout', name: 'app_logout')]
    public function logout(): void
    {
        throw new \LogicException('This method can be blank - it will be intercepted by the logout key on your firewall.');
    }
}

```


## 6. Template Twig du formulaire

```twig

{# templates/security/login.html.twig #}

{% extends 'base.html.twig' %}

{% block title %}Log in!{% endblock %}

{% block body %}
    <form method="post">
        {% if error %}
            <div class="alert alert-danger">{{ error.messageKey|trans(error.messageData, 'security') }}</div>
        {% endif %}

        {% if app.user %}
            <div class="mb-3">
                You are logged in as {{ app.user.userIdentifier }}, <a href="{{ logout_path() }}">Logout</a>
            </div>
        {% endif %}

        <h1 class="h3 mb-3 font-weight-normal">Please sign in</h1>
        <label for="username">Email</label>
        <input type="email" value="{{ last_username }}" name="_username" id="username" class="form-control" autocomplete="email" required autofocus>
        <label for="password">Password</label>
        <input type="password" name="_password" id="password" class="form-control" autocomplete="current-password" required>
        <input type="hidden" name="_csrf_token" data-controller="csrf-protection" value="{{ csrf_token('authenticate') }}">

        {#
            Uncomment this section and add a remember_me option below your firewall to activate remember me functionality.
            See https://symfony.com/doc/current/security/remember_me.html

            <div class="checkbox mb-3">
                <input type="checkbox" name="_remember_me" id="_remember_me">
                <label for="_remember_me">Remember me</label>
            </div>
        #}

        <button class="btn btn-lg btn-primary" type="submit">
            Sign in
        </button>
    </form>
{% endblock %}

```

> **Important** : Les champs `_username`, `_password` et `_csrf_token` sont des noms **réservés** utilisés par Symfony pour le traitement du formulaire.

---

## 7. Enregistrement d'un utilisateur (optionnel)

Pour créer un utilisateur avec un mot de passe haché :

```bash
php bin/console make:registration-form
```

Ou manuellement dans un contrôleur :

```php
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;

#[Route('/register', name: 'app_register')]
public function register(
    Request $request,
    UserPasswordHasherInterface $hasher,
    EntityManagerInterface $em
): Response {
    $user = new User();
    $user->setEmail('user@example.com');
    $user->setPassword($hasher->hashPassword($user, 'mon_mot_de_passe'));

    $em->persist($user);
    $em->flush();

    return $this->redirectToRoute('app_login');
}
```

---

## 8. Accéder à l'utilisateur connecté

### Dans un contrôleur

```php
// Méthode 1 : via AbstractController
$user = $this->getUser();

// Méthode 2 : via Security
use Symfony\Bundle\SecurityBundle\Security;

public function __construct(private Security $security) {}

public function someAction(): Response
{
    $user = $this->security->getUser();
}
```

### Dans Twig

```twig
{% if is_granted('IS_AUTHENTICATED_FULLY') %}
    <p>Bonjour, {{ app.user.email }} !</p>
    <a href="{{ path('app_logout') }}">Se déconnecter</a>
{% else %}
    <a href="{{ path('app_login') }}">Se connecter</a>
{% endif %}
```

---

## 9. Contrôler les accès

### Via `access_control` dans security.yaml

```yaml
access_control:
    - { path: ^/admin, roles: ROLE_ADMIN }
    - { path: ^/profil, roles: ROLE_USER }
```

### Via les attributs PHP dans les contrôleurs

```php
use Symfony\Component\Security\Http\Attribute\IsGranted;

#[IsGranted('ROLE_ADMIN')]
#[Route('/admin/dashboard', name: 'admin_dashboard')]
public function dashboard(): Response
{
    // Accessible uniquement aux admins
}
```

### Via `denyAccessUnlessGranted` dans une méthode

```php
public function edit(): Response
{
    $this->denyAccessUnlessGranted('ROLE_ADMIN', null, 'Accès refusé.');
    // ...
}
```

---

## 10. Se souvenir de moi (Remember Me)

Activer dans `security.yaml` :

```yaml
firewalls:
    main:
        remember_me:
            secret: '%kernel.secret%'
            lifetime: 604800  # 7 jours en secondes
            path: /
```

Et ajouter la case à cocher dans le formulaire (déjà incluse dans l'exemple Twig ci-dessus).

---

## Récapitulatif du flux d'authentification

```
[Utilisateur] 
     │
     ▼
[GET /login] → SecurityController::login() → Affiche le formulaire
     │
     ▼
[POST /login] → Symfony intercepte la requête
     │
     ├─ Authentification échouée → Redirige vers /login avec erreur
     │
     └─ Authentification réussie → Redirige vers default_target_path
```