# Challenge 

## Sloution Strategy 
**Step 1**: Clone the Challenge repository

**Step 2**: Create Dockerfile for the PHP Laravel API service 

**Step 3**: Create Dockerfile for the Nuxt.js Client service 

**Step 4**: Write Docker Compse file and nginx.conf

**Step 5**: Generate a self-signed certificate for the Nginx web server.

**Step 6**: Trouble Shooting 

**Step 7**: Run and Test the services
_____________________________________________________________________________________________________________________________________________________

## Step 1: Clone the Challenge Repository 

```
git clone https://github.com/only-for-testing/Challenge
```
```
cd Challenge
```
## Step 2: Create Dockerfile for the PHP Laravel API service 

1. Navigate to the API directory.
2. Create a Dockerfile.
```
FROM php:8.1-fpm

WORKDIR /var/www/html

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    libzip-dev \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    curl

# Install PHP extensions
RUN docker-php-ext-install pdo pdo_mysql mbstring exif pcntl bcmath gd

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

COPY . /var/www/html

RUN composer install

EXPOSE 8000
CMD ["php-fpm"]
```
![image](https://github.com/user-attachments/assets/16199f89-d699-4987-9be7-bca5ea54eecb)

## Step 3: Create Dockerfile for the Nuxt.js Client service 

1. Navigate to the Client directory.
2. create a Dockerfile.
   
```
FROM node:18

WORKDIR /app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install --legacy-peer-deps 

COPY . .

RUN npm run build

EXPOSE 3000
CMD ["npm", "run", "start"]
```
![image](https://github.com/user-attachments/assets/44746afc-ee83-46cb-a8b6-1389a2f9c60f)


## Step 4: Write Docker Compse file and nginx.conf

- Write a Docker Compose file at the root directory.
  
  Include services for:

    - db (MySQL database)

    - api (PHP Laravel API)

    - client (Nuxt.js Client)

    - nginx (Nginx web server acting as a proxy)
 
```
version: '3.8'

services:

#MYSQL database
  db:
    image: mysql:8
    environment:
      MYSQL_DATABASE: bookapi
      MYSQL_USER: app
      MYSQL_PASSWORD: password
      MYSQL_ROOT_PASSWORD: rootpassword
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - app-network

#PHP Laravel API
  api:
    build:
      context: ./api
    environment:
      DB_CONNECTION: mysql
      DB_HOST: db
      DB_PORT: 3306
      DB_DATABASE: bookapi
      DB_USERNAME: app
      DB_PASSWORD: password
    volumes:
      - ./api:/var/www/html
    depends_on:
      - db
    networks:
      - app-network

#Nuxt.js Client
  client:
    build:
      context: ./client
    volumes:
      - ./client:/app
    networks:
      - app-network

#Nginx web server
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - api
      - client
    networks:
      - app-network

volumes:
  db_data:

networks:
  app-network:
```
![image](https://github.com/user-attachments/assets/1d4d9f76-39f4-4e83-93f0-14f2372a1a26)

![image](https://github.com/user-attachments/assets/f22223f8-28d9-4237-a86b-a2ddbcaf45cb)


- Create an nginx.conf file for the Nginx reverse proxy.

```
# Global settings, if any
user  nginx;
worker_processes  auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # Server block begins here
    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://client:3000;  # Forward requests to Nuxt.js client
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /api {
            proxy_pass http://api:8000;  # Forward requests to Laravel API
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    }
}
```
![image](https://github.com/user-attachments/assets/9ec3a6ae-e3a8-4922-a976-8f256387a9f5)

## Step 5: Generate a self-signed certificate for the Nginx web server.

```
mkdir ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ssl/nginx.key -out ssl/nginx.crt -subj "/CN=localhost"
```

## Step 6: Trouble Shooting 

- Here's some problems may face you when you try to compose up the containers:
  
- When Compose up the containers and found the client and nginx container exit
- after Checking Clients' logs may got that error:
```
npm error A complete log of this run can be found in: /root/.npm/_logs/2024-09-29T23_28_39_006Z-debug-0.log
challenge_client_1 exited with code 1"
```
- Loginto the Container by running the following Command:
```
docker-compose run --rm client sh
```
- check if Nuxt is installed:
```
npm list nuxt
```
- if nuxt isn't installed, install and start it 
```
npm install nuxt
npm run build
npm run start
```

## Step 7: Run and Test the services

- Run the service :
```
docker-compose up --build 
```
![image](https://github.com/user-attachments/assets/aa500f4c-8b76-4b98-a982-a7b4ab45c2fa)

![image](https://github.com/user-attachments/assets/ffd3919f-f40b-46ed-ba8b-57f936acb7d1)

- Go to the browser and access nginx sevice at port 80:
```
http://localhost
```

- OR acess nginx through Incognito or private tab using ssl port 443:
  
```
https://localhost
```
![image](https://github.com/user-attachments/assets/48f333d6-f189-4669-9f38-77e751a77aa3)


