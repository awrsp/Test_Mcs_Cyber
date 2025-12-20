# Setting Up a High-Performance LEMP Stack with Nginx Using Docker Containers



his guide demonstrates how to set up and deploy a fully functional LEMP stack (Linux, Nginx, MySQL/MariaDB, PHP) using Docker containers. Docker provides portability, making deployment faster with isolated environments.
Prerequisites

Before we begin, ensure the following:

- Docker and Docker Compose are installed on your system.
- A basic understanding of Docker and containerized applications.
- Your system is updated:

```bash
sudo apt update && sudo apt upgrade -y
```

## 1. Prepare the Docker Compose File

We’ll use Docker Compose to orchestrate the LEMP stack components: Nginx, PHP-FPM, and MariaDB.
### Steps:

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
version: "3.8"

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
        server_name localhost;

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
docker ps
```

## 5. Test the Setup

- Open your browser and go to:

http://localhost

You should see the PHP information page displaying your PHP configuration.

If you encounter any issues, check the logs:

- Nginx logs: ./logs/nginx
- Docker container logs:

```bash
docker logs nginx_server
docker logs php_server
docker logs mariadb
```

## 6. Performance Optimization

To ensure the LEMP stack is optimized for high traffic, consider the following:

### A. Enable Gzip Compression in Nginx

Update the nginx.conf file to include compression settings:

```nginx
gzip on;
gzip_types text/plain application/json application/javascript text/css application/xml;
gzip_min_length 256;
```

Restart the stack:

```bash
docker-compose restart
```

### B. Tune MySQL/MariaDB

Optimize the MariaDB database by editing the my.cnf file. You can mount a custom configuration file for MariaDB if needed.

### C. Scale Services

If you expect high traffic:

- Use Docker’s built-in scaling for the PHP-FPM service:

```bash
docker-compose up -d --scale php=3
```

- Implement load balancing in Nginx (e.g., round-robin or least connections) to distribute traffic across multiple PHP containers.

## 7. Add SSL with Certbot (Optional)

To secure the stack with HTTPS:

- Add the Certbot container to your `docker-compose.yml` for SSL certificates:

```yaml
certbot:
  image: certbot/certbot
  container_name: certbot
  volumes:
    - ./certbot/conf:/etc/letsencrypt
    - ./certbot/logs:/var/log/letsencrypt
    - ./certbot/www:/var/www/certbot
  networks:
    - lemp_network
```

- Use Certbot with the Nginx container to generate an SSL certificate:

```bash
docker-compose run certbot certonly --webroot --webroot-path=/var/www/html -d your_domain -d www.your_domain
```

## Clean Up

To stop and remove containers:

```bash
docker-compose down
```

To completely remove containers, networks, and volumes:

```bash
docker-compose down --volumes
```


# Sources: 

[DigitalOcean - How To Set Up a LEMP Stack with Docker on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-lemp-stack-with-docker-on-ubuntu-20-04)


[Setting Up a High-Performance LEMP Stack with Nginx Using Docker Containers](https://medium.solvytech.com/setting-up-a-high-performance-lemp-stack-with-nginx-using-docker-containers-d8c2f4db174c)
