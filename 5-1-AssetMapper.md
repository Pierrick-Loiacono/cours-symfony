# AssetMapper en Symfony

## Introduction

**AssetMapper** est le système de gestion des assets front-end introduit dans Symfony 6.3, proposé comme alternative moderne à Webpack Encore. Il permet de gérer CSS, JavaScript et autres fichiers statiques **sans aucun outil Node.js** — pas de `npm`, pas de `webpack`, pas de `node_modules`.

Son fonctionnement repose sur les **Import Maps**, une fonctionnalité native des navigateurs modernes qui permet d'importer des modules JavaScript par nom plutôt que par chemin.

> **Quand choisir AssetMapper ?** Pour les projets qui n'ont pas besoin de transpilation TypeScript complexe, de JSX/React natif ou de pre-processing CSS avancé. Pour tout le reste, il est souvent suffisant et bien plus simple à maintenir.

---

## 1. Installation

Maintenant de base avec Symfony
```bash
composer require symfony/asset-mapper symfony/asset
```

Cette commande installe le bundle et génère automatiquement les fichiers de base :

```
assets/
├── app.js          ← point d'entrée JavaScript
└── styles/
    └── app.css     ← feuille de style principale

config/
└── packages/
    └── asset_mapper.yaml

importmap.php       ← registre des dépendances JS
templates/
└── base.html.twig  ← mis à jour avec les balises nécessaires
```

---

## 2. Configuration

### `config/packages/asset_mapper.yaml`

```yaml
framework:
    asset_mapper:
        # The paths to make available to the asset mapper.
        paths:
            - assets/
        missing_import_mode: strict

        # Préfixe des URLs générées (utile en production avec un CDN)
        # public_prefix: '/assets/'

when@prod:
    framework:
        asset_mapper:
            missing_import_mode: warn

```

AssetMapper surveille tous les fichiers dans les dossiers déclarés sous `paths` et les rend accessibles via une URL versionnée automatiquement.

### `importmap.php`

Ce fichier est le registre des bibliothèques JavaScript tierces. Il est géré via la CLI — il est rare de l'éditer à la main.

```php
<?php

// importmap.php — généré et mis à jour par AssetMapper

return [
    'app' => [
        'path' => './assets/app.js',
        'entrypoint' => true,
    ],
];
```

---

## 3. Intégrer les assets dans Twig

Le fichier `base.html.twig` doit inclure deux éléments essentiels :

```twig
{# templates/base.html.twig #}

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Mon Application{% endblock %}</title>

    {# Charge les feuilles de style liées au point d'entrée #}
    {% block stylesheets %}
        {{ importmap('app') }}
    {% endblock %}
</head>
<body>
    {% block body %}{% endblock %}

    {# Charge l'importmap et le JavaScript #}
    {% block javascripts %}
        {{ importmap('app') }}
    {% endblock %}
</body>
</html>
```

> La fonction Twig `{{ importmap('app') }}` génère à la fois la balise `<script type="importmap">` et l'import du point d'entrée `app.js`. Elle gère aussi automatiquement le chargement du CSS importé depuis JavaScript.

---

## 4. Le point d'entrée JavaScript

```javascript
// assets/app.js

// Importer la feuille de style principale
import './styles/app.css';

// Importer des modules locaux
import './bootstrap.js';

console.log('AssetMapper chargé !');
```

---

## 5. Ajouter des dépendances JavaScript

AssetMapper fournit une CLI pour gérer les packages — l'équivalent de `npm install`.

### Ajouter un package depuis jsDelivr (CDN)

```bash
# Ajouter Bootstrap (oui oui bootstrap)
php bin/console importmap:require bootstrap

```

Le fichier `importmap.php` est mis à jour automatiquement :

```php
<?php

return [
    'app' => [
        'path'       => './assets/app.js',
        'entrypoint' => true,
    ],
    'bootstrap' => [
        'version' => '5.3.2',
    ]
];
```

### Télécharger les packages en local

Par défaut, les packages sont chargés depuis un CDN. Pour les embarquer localement (recommandé en production) :

```bash
php bin/console importmap:install
```

Les fichiers sont téléchargés dans `assets/vendor/` et référencés localement dans `importmap.php`.

### Utiliser un package dans JavaScript

```javascript
// assets/app.js

import _ from 'lodash';
import 'bootstrap';

const users = ['Alice', 'Bob', 'Charlie'];
console.log(_.shuffle(users));
```

---

## 6. Gérer le CSS

### CSS global

Importez votre CSS directement depuis `app.js` :

```javascript
// assets/app.js
import './styles/app.css';
```

### Plusieurs feuilles de style

```javascript
// assets/app.js
import './styles/app.css';
import './styles/components.css';
import './styles/utilities.css';
```

### CSS d'une bibliothèque tierce

```bash
php bin/console importmap:require bootstrap/dist/css/bootstrap.min.css
```

```javascript
// assets/app.js
import 'bootstrap/dist/css/bootstrap.min.css';
import 'bootstrap';
```

---

## 7. Référencer des assets dans Twig

Pour les images, polices et autres fichiers statiques, utilisez la fonction `asset()` :

```twig
{# Image #}
<img src="{{ asset('images/logo.png') }}" alt="Logo">

{# Favicon #}
<link rel="icon" href="{{ asset('images/favicon.ico') }}">

{# Police locale #}
<link rel="preload" href="{{ asset('fonts/inter.woff2') }}" as="font" crossorigin>
```

AssetMapper calcule automatiquement l'URL versionnée (`/assets/images/logo-a1b2c3d4.png`) pour l'invalidation du cache navigateur.

---

## 8. Compiler les assets pour la production

En développement, AssetMapper sert les fichiers à la volée. En production, il faut les compiler dans `public/assets/` :

```bash
# Compiler tous les assets
php bin/console asset-map:compile
```

Cette commande :
- Copie tous les assets dans `public/assets/`
- Ajoute un **hash de contenu** dans le nom de chaque fichier (`app.a1b2c3d4.js`)
- Génère le fichier `importmap` final pour les navigateurs

### Intégration dans le déploiement

```bash
# Exemple de script de déploiement
composer install --no-dev --optimize-autoloader
php bin/console asset-map:compile
php bin/console cache:clear --env=prod
```

---

## 9. Inspecter les assets

```bash
# Lister tous les assets détectés par AssetMapper
php bin/console debug:asset-map

# Exemple de sortie :
# assets/app.js              → /assets/app.js
# assets/styles/app.css      → /assets/styles/app.css
# assets/images/logo.png     → /assets/images/logo.png
```

---


