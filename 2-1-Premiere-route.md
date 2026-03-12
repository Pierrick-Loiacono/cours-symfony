# Première route

### Contexte

Dans une app web, une route permet d'associer une URL à une action dans l'application. Quand on saisit une URL, une requête HTTP est envoyée au serveur. Symfony Analyse l'URL et cherche une route correspondante. Si une route correspond, Symfony appelle le contrôleur associé, qui va executer le code présent dans celui-ci et retourner une réponse HTTP, une page HTML par exemple

---


## 1. Notre première route
### Avec annotation

Pour simplifier un peu la création d'un contrôleur, on peut utiliser la CLI
- **`php bin/console make:controller`** : Cette commande va créer notre contrôleur ainsi que le fichier twig associé
> Vous pouvez préciser le nom que vous souhaitez directement dans la commande `php bin/console make:controller IndexController`.

> ⚠️ Symfony ajoutera automatiquement le suffixe Controller si vous ne le mettez pas. Notez que c'est une bonne pratique de le faire

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

final class IndexController extends AbstractController
{
    #[Route('/index', name: 'app_index', methods: ['GET'])]
    public function index(): Response
    {
        return new Response('Hello première route');
    }
}

```

- **`#[Route]`** : Annotation PHP indiquant la route (depuis Symfony 5), avant PHP 8 cela se faisant avec des doctrine annotation ``/** */``
- **`/index`** : URL de la page
- **`name:`** : nom interne de la route, doit être unique
- **`methods:`** : indique les méthodes que la route accepte
- **`Response`** : Response est un objet qui contient tout ce que le serveur renvoie au client : le contenu de la page, le code HTTP et les en-têtes.

Si on avait voulu retourner une vue Twig plutot qu'un simple texte dans une réponse on aurait fait ça :

```php
    #[Route('/index', name: 'app_index', methods: ['GET'])]
    public function index(): Response
    {
        return $this->render('index/index.html.twig', [
            'nom' => 'Loiacono',
            'prenom' => 'Pierrick',
        ]);
    }
```
- **`return $this->render(...)`** : C'est la vue Twig qui sera envoyé à l'utilisateur
- **`['controller_name' => 'IndexController']`** : C'est une variable qui conduit le texte 'IndexController', qui pourra être utilisé directement dans la vue Twig

### Route dans le fichier YAML routes.yaml / fichier YAML custom
```yaml
index:
    path: /index
    controller: App\Controller\IndexController::index
    methods: GET
```

Vous pouvez créer des fichiers yaml dans `/config/routes/`, ils seront chargés automatiquement par Symfony

## 2. Comment symfony connait les routes créées ?

Symfony connaît les routes grâce à un système de chargement des routes (le Router) qui lit les définitions de routes dans les fichiers de configuration ou dans les attributs des contrôleurs.

Grâce au fichier `config/routes.yaml`, il sait ou chercher les routes
```yaml
controllers:
    resource: routing.controllers
```

> Vous pouvez voir la liste des routes avec la commande : **`php bin/console debug:router`**

## 3. Paramètres dans une route

**Paramètre du contrôleur**

Symfony injecte automatiquement les paramètres de route dans les arguments de la méthode

```php
#[Route('/index/{nom}', name: 'app_index', methods: ['GET'])]
public function show(string $nom): Response
{
    return new Response("Hello $nom");
}
```

Ici, **`{nom}`** est passé en paramètre dans la route, et peut donc être récupéré directement dans la variable du même nom **`$nom`** avec le typage défini. 

**Avec l'objet Request**

```php
use Symfony\Component\HttpFoundation\Request;
#[Route('/index/{nom}', name: 'app_index', methods: ['GET'])]
public function show(Request $request): Response
{
    $slug = $request->attributes->get('nom');

    return new Response($nom);
}
```

> Ne pas confondre avec la query string, qui n'est pas un paramètre de route :  ?nom=pierrick  => $request->query->get('nom')