# Les fichiers .env



## 1. Les principaux fichier .env

Dans un projet Symfony, plusieurs fichiers ``env`` peuvent exister

**.env**

C'est le fichier principal. Il contient des valeurs par défaut pour l'application

> ⚠️ Il est versionné dans Git, ne mettez pas d'information sensible

**.env.local**

Ce fichier surcharge la configuration pour la machine locale. Donc chaque developpeur peut avoir sa propre conf

> Par défaut il est ignoré par git (voir .gitignore)

**.env.dev**
Configuration spécifique à l’environnement dev.

**.env.test**
Configuration spécifique à l’environnement test.

**.env.prod**
Configuration spécifique à l’environnement prod.

## 2. Ordre de priorité

Symfony charge les variables dans un certain ordre

1. .env.local
2. .env.[env]
3. .env

```bash
# .env
APP_ENV=dev

# .env.local
APP_ENV=prod
```

La variable prise en compte sera ``APP_ENV = prod``

## 3. Optimisation

Vous pouvez optimiser vos fichiers d'environnement en créant un seul fichier.

La commande `php bin/console dump-env prod` par exemple, va lire toutes les variables dans les différents fichiers et génére un `.env.local.php`. Ce fichier sera lu à la place de tous les autres

## 4. Accéder aux variables d'environnement

Vous pouvez ajouter vos propre variables

Dans vitre fichier d'environnement :

```env
PRENOM=Pierrick
```

Dans services.yaml :

```yaml
parameters:
    prenom: '%env(NOM)%'
```


Pour l'utiliser, 2 choix :

#### 1 - Utilisation de Parameter 

```php
$prenom = $this->getParameter('prenom');
```

#### 2 - Injection directe

Dans services.yaml

```yaml
services:
    App\Controller\StudentController:
        arguments:
            $prenom: '%env(PRENOM)%'
```

Dans votre service / controleur :

```php
    private string $prenom;

    public function __construct(string $prenom)
    {
        $this->prenom = $prenom;
    }

    #[Route('/', name: 'app_index')]
    public function index(): Response
    {
        dd($this->prenom);
    }
```