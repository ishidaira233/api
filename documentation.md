Docker : Nginx + PHP-FPM + Postgresql

Installation de cet architecture :
```
#----------1. Installation de Docker----------------#
sudo apt-get remove docker docker-engine docker.io containerd runc #uninstall old versions
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common
#---------2. Creer des fichiers comme suivant------#
.
├── bdd
│   ├── data1
│   ├── data2
│   └── data3
├── docker-compose.yml
├── html
│   ├── index.php
│   └── style.css
├── html2
│   └── index.php
├── html3
│   └── index.php
└── nginx
    ├── Dockerfile
    ├── proxy.conf
    ├── site1.conf
    ├── site2.conf
    └── site3.conf
#--------3. Ajouter le contenu des fichiers comme ci-dessous-----#
```

Le contenu de docker-compose.yml :

```
version: '3'
services:
  proxy:
    image: nginx:latest
    container_name: proxy
    ports:
        - "80:80"
    volumes:
        - ./nginx/proxy.conf:/etc/nginx/conf.d/default.conf
    networks:
      - front
      - backend
      - backend2
      - backend3
  web:
    image: nginx:latest
    container_name: web
    volumes:
        - ./html:/var/www/html
        - ./nginx/site1.conf:/etc/nginx/conf.d/default.conf
    networks:
      - backend
  php:
    image: pichouk/php
    container_name: php
    volumes:
        - ./html:/var/www/html
    networks:
      - backend
  postgresql:
    image: postgres:10
    container_name: postgresql
    environment:
      POSTGRES_DB: prism
      POSTGRES_USER: snowden
      POSTGRES_PASSWORD: nsa
    volumes:
      - ./bdd/data1:/var/lib/postgresql/data
    networks:
      - backend
  web2:
    image: nginx:latest
    container_name: web2
    volumes:
        - ./html2:/var/www/html
        - ./nginx/site2.conf:/etc/nginx/conf.d/default.conf
    networks:
      - backend2
  php2:
    image: pichouk/php
    container_name: php2
    volumes:
        - ./html2:/var/www/html
    networks:
      - backend2
  postgresql2:
    image: postgres:10
    container_name: postgresql2
    environment:
      POSTGRES_DB: prism
      POSTGRES_USER: snowden
      POSTGRES_PASSWORD: nsa
    volumes:
      - ./bdd/data2:/var/lib/postgresql/data
    networks:
      - backend2
  web3:
    image: nginx:latest
    container_name: web3
    volumes:
        - ./html3:/var/www/html
        - ./nginx/site3.conf:/etc/nginx/conf.d/default.conf
    networks:
      - backend3
  php3:
    image: pichouk/php
    container_name: php3
    volumes:
        - ./html3:/var/www/html
    networks:
      - backend3
  postgresql3:
    image: postgres:10
    container_name: postgresql3
    environment:
      POSTGRES_DB: prism
      POSTGRES_USER: snowden
      POSTGRES_PASSWORD: nsa
    volumes:
      - ./bdd/data3:/var/lib/postgresql/data
    networks:
      - backend3
networks:
  front:
    external: true
  backend:
    external: true
  backend2:
    external: true
  backend3:
    external: true
```

Le contenu dans site1.conf

```
# site1.conf
server{
        listen 80;
        server_name shenlika1.apicasoft.fr;
        root /var/www/html;
        index index.php index.html;
        location / {
                try_files $uri $uri/ =404;
        }
        location ~ \.php$ {
          try_files $uri =404;
          fastcgi_split_path_info ^(.+\.php)(/.+)$;
          fastcgi_pass php:9000;
          fastcgi_index index.php;
          include fastcgi_params;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
          fastcgi_param PATH_INFO $fastcgi_path_info;
        }
}
```

Le contenu dans site2.conf

```
# site2.conf
server{
        listen 80;
        server_name shenlika2.apicasoft.fr;
        root /var/www/html;
        index index.php index.html;
        location / {
                try_files $uri $uri/ =404;
        }
        location ~ \.php$ {
          try_files $uri =404;
          fastcgi_split_path_info ^(.+\.php)(/.+)$;
          fastcgi_pass php2:9000;
          fastcgi_index index.php;
          include fastcgi_params;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
          fastcgi_param PATH_INFO $fastcgi_path_info;
        }
}
```

Le contenu dans site3.conf :

```
# site3.conf
server{
        listen 80;
        server_name shenlika3.apicasoft.fr;
        root /var/www/html;
        index index.php index.html;
        location / {
                try_files $uri $uri/ =404;
        }
        location ~ \.php$ {
          try_files $uri =404;
          fastcgi_split_path_info ^(.+\.php)(/.+)$;
          fastcgi_pass php3:9000;
          fastcgi_index index.php;
          include fastcgi_params;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
          fastcgi_param PATH_INFO $fastcgi_path_info;
        }
}
```

Le contenu dans proxy.conf :

```
server {
    listen 80;
    server_name shenlika1.apicasoft.fr;
    location / {
        proxy_pass http://web:80;
    }
}
server {
    listen 80;
    server_name shenlika2.apicasoft.fr;
    location / {
        proxy_pass http://web2:80;
    }
}
server {
    listen 80;
    server_name shenlika3.apicasoft.fr;
    location / {
        proxy_pass http://web3:80;
    }
}
```
```
#--------4. Execution de Docker-----#
docker-compose up

# Donc j'ai 10 conteneurs suivants :

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
32defe3610f5        nginx:latest        "nginx -g 'daemon of…"   28 hours ago        Up 28 hours         0.0.0.0:80->80/tcp   proxy
95e549b2a4c1        postgres:10         "docker-entrypoint.s…"   28 hours ago        Up 28 hours         5432/tcp             postgresql2
6724b4cab992        postgres:10         "docker-entrypoint.s…"   28 hours ago        Up 28 hours         5432/tcp             postgresql
cceb7ece960b        postgres:10         "docker-entrypoint.s…"   28 hours ago        Up 28 hours         5432/tcp             postgresql3
5a2d1323c099        pichouk/php         "docker-php-entrypoi…"   28 hours ago        Up 28 hours         9000/tcp             php2
e123fe9a5a6a        nginx:latest        "nginx -g 'daemon of…"   28 hours ago        Up 28 hours         80/tcp               web
4ed025b6ac0f        pichouk/php         "docker-php-entrypoi…"   28 hours ago        Up 28 hours         9000/tcp             php3
5defdf7f8207        nginx:latest        "nginx -g 'daemon of…"   28 hours ago        Up 28 hours         80/tcp               web2
26b3a5818caf        nginx:latest        "nginx -g 'daemon of…"   28 hours ago        Up 28 hours         80/tcp               web3
0b82b87af69c        pichouk/php         "docker-php-entrypoi…"   28 hours ago        Up 28 hours         9000/tcp             php
```

Avec web, php et postgresql, je crée un site avec nginx+php+postgresql, des trois conteneurs sont dans le même réseau, du coup ils peuvent se communiquer. Pareil, je crée deux autres sites avec chaque trois conteneurs et un réseau. Ensuite, pour reverse proxy vers l'application, j'ajoute un proxy. Le conteneur Nginx applicatif n'a plus besoin d'exposer son port sur l'hôte Docker, c'est le reverse proxy qui va s'en charger. 

Certains commands utiles dans Docker

```
#Pour Image :
docker image ls 
docker pull NOM_IMGAE #telecharger une image
docker image rm NOM_IMAGE 
#Pour Conteneur :
docker run -it nom_con
docker container ls -a
docker container rm NOM_CONTENEUR #移除
docker start NOM_CONTENEUR
docker stop NOM_CONTENEUR
docker pause #suspendu le processus
docker unpause #relancer le processus 
docker exec -it postgresql psql -U postgres
# create new network ：
sudo docker network create myfirstnet
#supprimer tout les conteneurs existants, qui ne servent plus.
docker rm -vf $(docker ps -a -q)
#Entrer un postgresql conteneur： 
1. docker exec -it postgresql2 bash
2. su postgres
3. psql -U snowden -d prism
```



