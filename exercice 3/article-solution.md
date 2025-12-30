# Setting Up a High-Performance LEMP Stack with Nginx Using Docker Containers



his guide demonstrates how to set up and deploy a fully functional LEMP stack (Linux, Nginx, MySQL/MariaDB, PHP) using Docker containers. Docker provides portability, making deployment faster with isolated environments.
Prerequisites

Before we begin, ensure the following:

- Docker and Docker Compose are installed on your system.
- A basic understanding of Docker and containerized applications.
- Your system is updated:


[Docker and docker compose installation commands](https://docs.docker.com/engine/install/debian/) (if not already installed for debian based distros)



```bash
sudo apt update && sudo apt upgrade -y
```

## 1. Prepare the Docker Compose File

We’ll use Docker Compose to orchestrate the LEMP stack components: Nginx, PHP-FPM, and MariaDB.
### Steps:


## get wordpress 

```bash

wget https://wordpress.org/latest.zip
```

## unzip wordpress

```bash
unzip latest.zip -d public_html/ && rm latest.zip
```
- Create the project directory:

```bash
mkdir lemp-docker && cd lemp-docker
```

- Create a `docker-compose.yml` file:

```bash
nano docker-compose.yml
```

- Add the following content:

```yaml

services:
  nginx:
    image: nginx:latest
    container_name: nginx_server
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./public_html:/var/www/html
      - ./logs/nginx:/var/log/nginx
    depends_on:
      - php
    networks:
      - lemp_network

  php:
    image: php:8.1-fpm
    container_name: php_server
    volumes:
      - ./public_html:/var/www/html
    networks:
      - lemp_network

  db:
    image: mariadb:latest
    container_name: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: my_database
      MYSQL_USER: user
      MYSQL_PASSWORD: userpassword
    volumes:
      - ./data:/var/lib/mysql
    networks:
      - lemp_network

networks:
  lemp_network:
    driver: bridge
```

## 2. Set Up the Nginx Configuration

For this setup, Nginx acts as the webserver to serve static resources and proxy PHP requests to the PHP-FPM container.
### Steps:

- Create an `nginx/` directory inside your project:

```bash
mkdir -p nginx
```

- Create the Nginx configuration file:

```bash
nano nginx/nginx.conf
```

- Add the following configuration:

```nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        server_name 10.0.60.236;  # replace with your container/host IP (currently 10.0.60.236)

        root /var/www/html;
        index index.php index.html index.htm;

        location / {
            try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
            include fastcgi_params;
            fastcgi_pass php_server:9000;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_index index.php;
        }

        location ~ /\.ht {
            deny all;
        }
    }
}
```

## 3. Add PHP Application Code

We’ll create a simple PHP application to test the stack.

- Create a `public_html/` directory to hold application files:

```bash
mkdir public_html
```

- Create a `public_html/index.php` file:

```bash
nano public_html/index.php
```

Add the following content:

```php
<?php phpinfo(); ?>
```

## 4. Launch the LEMP Stack

Now that we have the docker-compose.yml, Nginx configuration, and PHP application ready, we can bring the stack online.
### Steps:

Inside your project directory, run:

```bash
docker-compose up -d
```

Verify that the containers are running:

```bash
docker-compose ps
```

## 5. Test the Setup

- Open your browser and go to:

http://10.0.60.236

You should see the PHP information page displaying your PHP configuration.

If you encounter any issues, check the logs:

- Nginx logs: ./logs/nginx
- Docker container logs:

```bash
docker compose logs nginx_server
docker compose logs php_server
docker logs mariadb
```


## Clean Up

To stop and remove containers:

```bash
docker compose down
```

To completely remove containers, networks, and volumes:

```bash
docker compose down --volumes
```


# Sources: 

[DigitalOcean - How To Set Up a LEMP Stack with Docker on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-lemp-stack-with-docker-on-ubuntu-20-04)


[Setting Up a High-Performance LEMP Stack with Nginx Using Docker Containers](https://medium.solvytech.com/setting-up-a-high-performance-lemp-stack-with-nginx-using-docker-containers-d8c2f4db174c)
