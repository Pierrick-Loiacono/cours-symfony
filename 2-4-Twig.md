# Twig


Twig est le moteur de templates utilisé par défaut dans Symfony. Son rôle est de générer du HTML dynamique en affichant des données envoyées par les contrôleurs. Twig à des avantages, syntaxe lisible, séparation logique / affichage. sécurisé par défaut (escape HTML).

---
## 1. Template principal : base.html.twig (héritage)

Dans le dossier `/templates` vous pouvez retrouver le template principal dont la quasi totalité des templates que vous allez créer vont hériter, pourquoi ? Parce que plutot que de tout copier dans chaque page, Twig permet d'utiliser l'héritage de template.

Dans une app web, de nombreuses pages partagent la même structure HTML :
- le `<head>`
- un menu de navigation
- le footrer
- les scripts / style
- une barre de recherche ...

Donc le principe est simple :
- Un template parent contient la structure générale
- Les templates enfants remplissent certaines parties

Ces parties sont appelées **blocs**. Ce sont des zzone du template qui peut être remplacée ou complétée. Vous pouvez créer autant de bloc que vous voulez

Par défaut, Symfony a générer celui-ci : **base.html.twig**

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
        <link rel="icon" href="data:image/svg+xml, ...">
        {% block stylesheets %}
        {% endblock %}

        {% block javascripts %}
            {% block importmap %}{{ importmap('app') }}{% endblock %}
        {% endblock %}
    </head>
    <body>
        {% block body %}{% endblock %}
    </body>
</html>
```

Dans mon index/index.html.twig
```html
{% extends 'base.html.twig' %}

{% block title %}Hello IndexController!{% endblock %}

{% block body %}
    <p> Texte par défaut blabla </p>
{% endblock %}
```

Dans ce fichier Twig on `extends` le template de base, donc il héritera de tout le fichier parent et pourra soit redéfinir entiérement les blocs, soit les compléter

Ici dans l'exemple `{% block title %}Hello IndexController!{% endblock %}` écrase complétement le contenu du bloc parent

Si on avait voulu conserver ce qu'il y avait dans le bloc parent, tout en ajoutant notre texte, il aurait fallu utiliser la fonction que Twig met à disposition : `{{ parent() }}`

## 2. Inclusion de template

Il est tout à fait possible d'inclure un morceau de template dans un autre. Ces morceaux de templates sont souvent mis dans un dossier `/templates/partials` et le nom du fichier comme par `_`
```twig
{% include 'header.html.twig' %}
```

Par défaut, un template inclus hérite automatiquement des variables du template initial. En revanche il vous est possible de lui en passer explicitement ou alors de ne pas le faire hériter des variables

```twig
{% include 'produits/_card.html.twig' with { produit: produit } %}
```

```twig
{% include 'produits/_card.html.twig' with { produit: produit } only %}
```
## 3. Génération d'URL

**Sans paramètres**
Vous pouvez générer des URL dans le Twig grâce à la fonction ``path()`` et ainsi préciser le **nom** d'une route 

```twig
<a href="{{ path('produits_index') }}">
    Liste des produits
</a>
```

**Avec paramètres**

```twig
<a href="{{ path('produit_show', {'id': produit.id}) }}">
    Voir le produit
</a>
```


## 4. Les variables

### 4.1 Envoie de données depuis le contrôleur
Pour rappel, vous pouvez transmettre des données dans des variables via le contrôleur.

```php
    return $this->render('index/index.html.twig', [
            'nom' => 'Loiacono',
            'prenom' => 'Pierrick',
        ]);
```

Ici on envoie dans la vue un tableau de variable que la vue Twig pourra utiliser directement dans son fichier. On peut afficher le contenu d'une variable avec des doubles accolades ``{{ }}``

```twig
    <p> Salut {{ nom }} {{prenom}} </p>
```

> Si dans la vue twig vous tenter d'utiliser une variable qui n'ait pas créée, ça plante

### 4.2 Création de variable locale dans une vue

```html
{% set prenom = "Pierrick" %}

<p> Salut {{ prenom }} </p>
```

### 4.3 Les commentaires

Les commentaires n'apparaitront pas dans le rendu HTML. Les commentaires se font avec ``{# #}``
```twig
{# Je suis un commentaire tout mignon #}
```

### 4.4 Conditions

Pour les conditions on utilise : `{% if %}`, `{% elseif %}`, `{% else %}` et `{% endif %}`

```twig
{% if note >= 18 %}
    <p>Très bien ! GG</p>
{% elseif  note >= 10 %}
    <p>Pas mal pour un mortel</p>
{% else %}
    <p>Avada Kedavra !!!</p>
{% endif %}
```

**Opérateurs logiques**

On les oublie pas ceux la
- and
- or
- not

```twig
{% if user.isAdmin and user.isActive %}
    <p>Administrateur actif</p>
{% endif %}
```

**Tester si une variable existe**

```twig
{% if prenom is defined %}
    <p>Hello {{ prenom }}</p>
{% endif %}
```

**Tester si une variable existe**
```twig
{% if articles is empty %}
    <p>Aucun article disponible</p>
{% endif %}
```

Et l'inverse 

```twig
{% if articles is not empty %}
```

**Tester si une valeur appartient à une liste**

```twig
{% if user.role in ['admin', 'moderateur'] %}
    <p>Accès autorisé mon gaté</p>
{% endif %}
```

**Vérifier un booléen**
```twig
{% if user.isVerified %}
    <p>Compte vérifié</p>
{% endif %}
```

### 4.5 Boucles

Pour boucler : `{% for %}` 
Elles permettent de parcourir une collection de données :
- tableau / liste
- collection d'objets

```twig
{% set fruits = ['Pomme', 'Banane', 'Orange'] %}

{% for fruit in fruits %}
    <li>{{ fruit }}</li>
{% else %}
    <p> Aucun fruit, c'est bien vide </p>
{% endfor %}
```

**Clé valeur**

```twig
{% set notes = {
    "math": 15,
    "anglais": 12,
    "info": 18
} %}

{% for key, value in tableau %}
    {{ key }} : {{ value }}
{% endfor %}
```

**La variable ``loop``**

| Variable | Description |
| -------- | ------- |
| **loop.index** | position (commence à 1) |
| **loop.index0** | position (commence à 0) |
| **loop.first** | premier élément |
| **loop.last** | dernier élément |
| **loop.length** | nombre total |
```twig
{% for produit in produits %}
    <p>{{ loop.index }} - {{ produit.nom }}</p>
{% endfor %}
```

Résultat :
```
1 - Produit A
2 - Produit B
3 - Produit C
```

Autre exemple
```twig
{% for produit in produits %}

    {% if loop.first %}
        <p>Premier produit :</p>
    {% endif %}

    <p>{{ produit.nom }}</p>

    {% if loop.last %}
        <p>Fin de la liste</p>
    {% endif %}

{% endfor %}
```

**`for` avec condition**
```twig
{% for produit in produits if produit.isValid %}
    <p>{{ produit.nom }}</p>
{% endfor %}
```

### 4.6 Les filtres

Syntaxe : `{{ variable | filtre }}`

| Filtre | Exemple | Résultat |
| -------- | ------- | ------- |
| **upper** | {{ prenom \| upper }} | PIERRICK |
| **lower** | {{ prenom \| lower }} | pierrick |
| **capitalize** | {{ prenom \| capitalize  }} | Pierrick |
| **length** | {{ prenom \| length  }} | 8 |
| **date** | {{ produit.date \| date("d/m/Y") }} | 01/01/2025 |
| **date** | {{ produit.date \| date("d/m/Y H:i") }} | 01/01/2025 14:30 |
| **default** | {{ utilisateur.nom \| default("Anonyme") }} | Anonyme |
```twig

```

```twig

```

### 4.7 Autres

**Concaténation**

On concaténe avec le tilde `~`
```twig
{% set prenom = "Pierrick" %}
{% set nom = "Loiacono" %}
{% set nom_complet = nom ~ " " ~ prenom %}

<p> Salut {{ nom_complet }} </p>
```

**Calcul**

Tous les opérateurs sont disponibles 

| Oéprateur | Signification |
| -------- | ------- |
| **+** | Addition |
| **-** | Soustraction |
| **\*** | Multiplication |
| **/** | Division |
| **%** | Modulo |

Exemple :

```twig
<p>Total : {{ prix * quantite }} €</p>
```

**Test des types / états**
```twig
{% if articles is empty %}
{% if utilisateur is null %}
{% if age is even %}
{% if age is odd %}
```

**Opérateur ternaire**

```twig
{{ age >= 18 ? "Majeur" : "Mineur" }}
```

**Opérateur Elvis**

```twig
{{ utilisateur.nom ?? "Anonyme" }}
```

**Dump une variable**

```twig
{{ dump(variable) }}
```