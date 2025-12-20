# Solution Exercice 3 : Hébergement WordPress sur Proxmox

Ce tutoriel est divisé en plusieurs sections pour vous guider à travers le processus de résolution de l'exercice.

Nous allons héberger notre projet sur Proxmox local en utilisant un conteneur LXC basé sur [Ubuntu 25.04 standard](https://releases.ubuntu.com/plucky/) (Plucky).

Nous allons installer WordPress sur une stack [LEMP](https://www.geeksforgeeks.org/websites-apps/what-is-lemp-stack/) (Linux, Nginx, MySQL, PHP).

## Étape 1 : Préparation du conteneur LXC

1. Créez un nouveau conteneur LXC dans Proxmox en utilisant l'image Ubuntu 25.04.

2. Allouez les ressources nécessaires (CPU, RAM, stockage) en fonction des besoins de votre projet :
   - 32 gb de stockage devraient suffire pour ce projet.
   - 2 coueurs CPU et 4 gb de RAM devraient être suffisants.

3. Réseau et DNS :
   - Nous allons utiliser une ip statiquedynamique pour ce conteneur ou nous allons faire une reservation DHCP sur notre routeur.
   - Un dns cloudflare sera utilisé pour resolver.

4. Le conteneur :
   - Il sera unprivileged.

## Étape 2 : Installation de la stack LEMP

1. Mettez à jour les paquets du système :

```bash
sudo apt update && sudo apt upgrade -y
```

2. Installez docker et docker-compose :

```bash
sudo apt install  docker.io docker-compose -y
```

3. Créez un fichier `docker-compose.yml` pour configurer les services Nginx, MySQL et PHP-FPM :

```yaml
version: '3.1'

services:

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: examplepassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpressuser
      MYSQL_PASSWORD: wordpresspassword
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpressuser
      WORDPRESS_DB_PASSWORD: wordpresspassword
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress_data:/var/www/html

volumes:
  db_data:
  wordpress_data:
```

4. Démarrez les services avec Docker Compose :
