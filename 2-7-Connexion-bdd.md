# Connexion à la base de données


La connexion à la base de données se fait par le biais d'un fichier de configuration et d'une variable d'environnement. un fichier de configuration indique quelle variable d'environnement utilisé, et celle-ci se configure dans votre fichier d'environnement, par défaut `.env`

## Fichier `.env`
Les fichiers d’environnement servent à configurer l’application selon le contexte d’exécution (développement, production, test…). Ils permettent d’éviter de mettre des informations sensibles ou spécifiques à une machine directement dans le code.

Les informations de connexion à la base de données sont stockées dans une variable d'environnement appelée ``DATABASE_URL``. Pour le développement, vous pouvez la trouver et la modifier si besoin dans le fichier ``.env``.

Variable d'environnement pour MySql :
```d
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name?serverVersion=8.0.37"
```

- **`db_user`** : utilisateur de la base
- **`db_password`** : mot de passe
- **`127.0.0.1`** : adresse du serveur
- **`3306`** : port par défaut de MySQL
- **`nom_base`** : nom de la base de données
- **`serverVersion`** : version du serveur

## Test de la connexion

Cette commande permet de créer la base de données
```bash
php bin/console doctrine:database:create
```

> Si ça plante, vérifier les informations de votre variable et si tout est bien installé (BDD, extensions php)

## Mettre à jour la base de données en rapport avec les entités

Vous avez 2 possibilités pour mettre à jour votre base de données :
1. Les migrations
2. schema:update

### 1. Les migrations

Les migrations permettent de générer un fichier décrivant les changements à appliquer à la base de données.

```bash
php bin/console make:migration
```

Doctrine va comparer les entités et la structure actuelle de la base, puis générer un fichier dans `/migration`

Pour exécuter la migration et ainsi mettre à la base :
```bash
php bin/console doctrine:migrations:migrate
```

**Avantages**
- Historique des modifications
- traçabilité
- reproductible sur d'autres environnements

### 2. schema:update

Cette commande met directement à jour la base de données à partir des entités
```bash
php bin/console doctrine:schema:update
```
Doctrine compare les entités et la structure et applique immédiatement les changements

**Inconvénients**

- Aucun historique
- impossible de revenir en arrière
