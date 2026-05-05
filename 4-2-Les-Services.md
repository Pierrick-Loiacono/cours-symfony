# Les Services en Symfony

## Introduction

Dans Symfony, un **service** est un objet PHP qui accomplit une tâche précise et réutilisable : envoyer un email, manipuler des fichiers, calculer des données, interagir avec une API externe, etc.

Le système de services repose sur le **conteneur d'injection de dépendances** (Dependency Injection Container), qui instancie et fournit automatiquement les objets dont votre application a besoin — sans que vous ayez à les créer manuellement.

> **Principe clé** : plutôt que d'écrire `$mailer = new MailService()` dans chaque contrôleur, vous déclarez que votre classe *a besoin* d'un `MailService`, et Symfony se charge de le fournir.

---

## 1. Qu'est-ce que le conteneur de services ?

Le conteneur est un registre central qui :

- **Instancie** les services à la demande (lazy loading)
- **Injecte** automatiquement leurs dépendances
- **Partage** les instances (un service est créé une seule fois par requête)

Symfony scanne automatiquement le dossier `src/` et enregistre toutes vos classes comme services potentiels, grâce à l'**autowiring**.

---

## 2. Installer le composant Mailer

```bash
composer require symfony/mailer
```

Configurer le transport dans `.env` :

```dotenv
# .env
MAILER_DSN=smtp://user:password@smtp.example.com:587
```

---

## 3. Créer le service

Un service est simplement une classe PHP classique. Aucune interface ou classe de base n'est requise.

### Exemple : `MailService`

Un service qui centralise l'envoi des emails de l'application : confirmation d'inscription, réinitialisation de mot de passe, notification de contact.

```php
// src/Service/MailService.php

namespace App\Service;

use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mime\Address;
use Symfony\Component\Mime\Email;
use Twig\Environment;

class MailService
{
    public function __construct(
        private readonly MailerInterface $mailer,
        private readonly Environment     $twig,
        private readonly string          $senderEmail,
        private readonly string          $senderName,
    ) {}

    /**
     * Envoie un email de confirmation d'inscription.
     */
    public function sendRegistrationConfirmation(string $toEmail, string $username, string $confirmUrl): void
    {
        $html = $this->twig->render('emails/registration.html.twig', [
            'username'   => $username,
            'confirmUrl' => $confirmUrl,
        ]);

        $this->send(
            to:      $toEmail,
            subject: 'Confirmez votre inscription',
            html:    $html,
        );
    }

    /**
     * Envoie un email de réinitialisation de mot de passe.
     */
    public function sendPasswordReset(string $toEmail, string $resetUrl): void
    {
        $html = $this->twig->render('emails/password_reset.html.twig', [
            'resetUrl'  => $resetUrl,
            'expiresIn' => '1 heure',
        ]);

        $this->send(
            to:      $toEmail,
            subject: 'Réinitialisation de votre mot de passe',
            html:    $html,
        );
    }

    /**
     * Envoie une notification suite à un message de contact.
     */
    public function sendContactNotification(string $fromEmail, string $fromName, string $message): void
    {
        $html = $this->twig->render('emails/contact.html.twig', [
            'fromEmail' => $fromEmail,
            'fromName'  => $fromName,
            'message'   => $message,
        ]);

        $this->send(
            to:      $this->senderEmail, // Notifie l'administrateur
            subject: "Nouveau message de {$fromName}",
            html:    $html,
            replyTo: $fromEmail,
        );
    }

    /**
     * Méthode interne générique pour construire et envoyer un email.
     */
    private function send(string $to, string $subject, string $html, ?string $replyTo = null): void
    {
        $email = (new Email())
            ->from(new Address($this->senderEmail, $this->senderName))
            ->to($to)
            ->subject($subject)
            ->html($html);

        if ($replyTo) {
            $email->replyTo($replyTo);
        }

        $this->mailer->send($email);
    }
}
```

---

## 4. Configurer le service

Grâce à l'**autowiring automatique**, `MailerInterface` et `Environment` (Twig) sont injectés sans configuration. Seuls les paramètres scalaires (`$senderEmail`, `$senderName`) doivent être déclarés dans `config/services.yaml` :

```yaml
# config/services.yaml

parameters:
    app.sender_email: 'noreply@mon-site.fr'
    app.sender_name:  'Mon Site'

services:
    _defaults:
        autowire: true
        autoconfigure: true

    App\:
        resource: '../src/'
        exclude:
            - '../src/DependencyInjection/'
            - '../src/Entity/'
            - '../src/Kernel.php'

    App\Service\MailService:
        arguments:
            $senderEmail: '%app.sender_email%'
            $senderName:  '%app.sender_name%'
```

> **Règle pratique** : si le constructeur ne contient que des classes typées, aucune configuration n'est nécessaire. Les scalaires seuls requièrent une déclaration explicite.

---

## 5. Exemple d'un template lié a une méthode du service

```twig
{# templates/emails/registration.html.twig #}

<!DOCTYPE html>
<html>
<body>
    <h1>Bienvenue, {{ username }} !</h1>
    <p>Merci de vous être inscrit. Veuillez confirmer votre adresse email en cliquant sur le lien ci-dessous :</p>
    <a href="{{ confirmUrl }}">Confirmer mon inscription</a>
    <p>Ce lien est valable 24 heures.</p>
</body>
</html>
```

---

## 6. Utiliser le service

### Dans un contrôleur d'inscription

```php
// src/Controller/RegistrationController.php

namespace App\Controller;

use App\Service\MailService;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class RegistrationController extends AbstractController
{
    public function __construct(
        private readonly MailService $mailService
    ) {}

    #[Route('/register', name: 'app_register', methods: ['POST'])]
    public function register(Request $request): Response
    {
        // ... création de l'utilisateur et génération du token ...

        $this->mailService->sendRegistrationConfirmation(
            toEmail:    $request->request->get('email'),
            confirmUrl: "On a pas encore de route pour ça, ça arrive fort",
        );

        $this->addFlash('success', 'Un email de confirmation vous a été envoyé.');

        return $this->redirectToRoute('app_login');
    }
}
```

---

## 7. Les modes d'injection

Symfony supporte trois façons d'injecter des dépendances :

### Injection par le constructeur ✅ (recommandée)

```php
public function __construct(private readonly MailService $mailService) {}
```

- Dépendances obligatoires et clairement déclarées
- Favorise l'immutabilité (`readonly`)
- Facilite les tests unitaires

### Injection par setter

```php
private ?LoggerInterface $logger = null;

public function setLogger(LoggerInterface $logger): void
{
    $this->logger = $logger;
}
```

- Utile pour les dépendances **optionnelles**

### Injection par attribut PHP 8

```php
use Symfony\Contracts\Service\Attribute\Required;

#[Required]
public LoggerInterface $logger;
```

---

## 8. Tester le service mail

J'utilise Mailpit en local pour tester les mails : https://mailpit.axllent.org/

**Mailpit**

Téléchargez et d'exécuter le .exe. Mailpit démarre généralement sur http://localhost:1025

**Configuration Symfony**

Modifier votre variable d'environnement `MAILER_DSN`
 `MAILER_DSN=smtp://mailpit:1025`

**Envoie des mails**
Avec Symfony messenger : `php bin/console messenger:consume async`

**Quelques options pour la commande messenger:consume**

`php bin/console messenger:consume async --limit=10`
`php bin/console messenger:consume async --time-limit=60`
`php bin/console messenger:consume async --memory-limit=128M`
`php bin/console messenger:consume async --sleep=1000000` (en microseconde)

---


## 9. Bonnes pratiques

| ✅ À faire | ❌ À éviter |
|---|---|
| Un service = une responsabilité | Envoyer des emails directement dans les contrôleurs |
| Injection par constructeur | `new MailService()` instancié à la main |
| Templates Twig pour le contenu HTML | Concaténer du HTML dans le service |
| Paramètres dans `services.yaml` | Coder l'adresse expéditeur en dur |
| Tester avec le transport `null://null` ou un SMTP local | Envoyer de vrais emails en environnement de test avec de vraies adresses |

---

## Récapitulatif

```
[Conteneur Symfony]
        │
        ├── Injecte MailerInterface  (autowiring)
        ├── Injecte Twig\Environment (autowiring)
        ├── Injecte $senderEmail     (services.yaml)
        └── Injecte $senderName      (services.yaml)
                │
                ▼
        [MailService]
                │
                ├── sendRegistrationConfirmation()
                ├── sendPasswordReset()
                └── sendContactNotification()
                        │
                        ▼
              [Contrôleurs, autres Services]
```

Le service `MailService` centralise toute la logique d'envoi d'emails en un seul endroit, rendant les contrôleurs simples et la logique facilement testable et réutilisable.

---

