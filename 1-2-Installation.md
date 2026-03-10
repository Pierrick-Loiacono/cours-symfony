# Installation

### Contexte

Selon la version de Symfony utilisée, la version de PHP a posséder peut être différente. 
- Pour Symfony 7.4 il faut PHP >= 8.2, mais pour Symfony 8.0 il faut PHP >= 8.4.
- Composer : c'est le gestionnaire de dépendances PHP
- Symfony CLI : Outil en CLI officiel qui facilite la création et la gestion d'un projet Symfony

---

## Installation de PHP

**Linux**

```bash
sudo apt update
sudo apt install -y lsb-release ca-certificates apt-transport-https software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install -y php8.3 php8.3-cli php8.3-common php8.3-mbstring php8.3-xml php8.3-curl php8.3-intl php8.3-pdo php8.3-mysql php8.3-zip
```
---

**Windows**

Vous pouvez téléchargez la version de PHP souhaité sur le site officiel https://windows.php.net/download/ 

Vous pouvez aussi tout à fait installer WAMP ou XAMPP, qui est package de tout le nécessaire (PHP, MySQL, Apache), mais qui peut nécessiter d'autres installations

!!! - Oubliez pas de mettre le necessaire dans vos variables d'environnement

---

**Mac**

---

## Composer

Vous devez avoir PHP pour pouvoir installer et utiliser Composer. Il permettra de gérer les dépendances des projets PHP

**Linux / mac**
```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'c8b085408188070d5f52bcfe4ecfbee5f727afa458b2573b8eaaf77b3419b0bf2768dc67c86944da1544f06fa544fd47') { echo 'Installer verified'.PHP_EOL; } else { echo 'Installer corrupt'.PHP_EOL; unlink('composer-setup.php'); exit(1); }"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

**Windows**

Téléchargez directement via le site : https://getcomposer.org/download/

Vous pouvez vérifier l'installation : 
```bash
composer -V
```

## Symfony CLI
**Linux / Mac**
```bash
wget https://get.symfony.com/cli/installer -O - | bash
mv ~/.symfony/bin/symfony /usr/local/bin/symfony
```

**Windows**

Téléchargez l'install sur leur Github : https://github.com/symfony/cli/releases

Vous pouvez vérifier l'installation : 
```bash
symfony -V
```

### Pourquoi Symfony CLI ?

- Vérifier si l'environnement est compatible : `symfony check:requirements`
    - Si des extensions / packages sont manquants, la CLI vous le dira
- Lancer un serveur local : `symfony server:start`
- Créer un projet : `symfony new nom_projet --webapp`
- Voir les logs : `symfony server:log`

### Création du projet

1. `symfony new nom_projet --webapp --version=7.4`
2. `cd nom_projet`
3. `symfony server:start`
4. Dans un navigateur : http://localhost:8000

