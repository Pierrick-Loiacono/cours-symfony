# Les fichiers .env



## Les principaux fichier .env

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

## Ordre de priorité

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

## Optimisation

Vous pouvez optimiser vos fichiers d'environnement en créant un seul fichier.

La commande `php bin/console dump-env prod` par exemple, va lire toutes les variables dans les différents fichiers et génére un `.env.local.php`. Ce fichier sera lu à la place de tous les autres