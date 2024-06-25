---
title: How to Dockerize a Laravel 11 and React 18 Application for Development
description: ""
date: 2024-06-25T13:03:26.187Z
preview: ""
draft: false
tags: []
categories:
    - Laravel
    - React.js
    - Devops
hero: "/images/posts/2024-06-25-how-to-dockerize-a-laravel-11-and-react-18-application-for-development.png"
type: posts
slug: dockerize-laravel-11-react-18-application-development
---

### Introduction
Docker has become a popular choice for development environments due to its ability to containerize applications and simplify deployment across different systems. This guide will walk you through dockerizing a Laravel 11 backend and React 18 frontend application, using Docker Compose to orchestrate multiple services. We wil assume that your laravel and react apps live in seperate folders and will be served through different domain. Suffice to say that the react app will not be served from the laravel backend but will use the laravel backend only as an api service.

#### Docker Compose Configuration
First, create a `docker-compose.yml` file in your project's root directory:

```
version: '3'

services:
  app:
    build:
      context: ./laravel-11-backend
      dockerfile: Dockerfile
    ports:
      - "8000:9000"
    volumes:
      - ./laravel-11-backend:/var/www/html
      - ./laravel-11-backend/storage:/var/www/html/storage
      - ./laravel-11-backend/bootstrap/cache:/var/www/html/bootstrap/cache
    networks:
      - laravel-network
    depends_on:
      - mysql
    environment:
      - DB_HOST=mysql

  mysql:
    image: mysql:8.0
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: laravel_react_blog_user_permission
      MYSQL_USER: app
      MYSQL_PASSWORD: password
      MYSQL_ROOT_PASSWORD: vicecity
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - laravel-network

  nginx:
    build:
      context: ./nginx
    ports:
      - "80:80"
    volumes:
      - ./laravel-11-backend:/var/www/html
    networks:
      - laravel-network
    depends_on:
      - app

  frontend:
    build:
      context: ./react-18-frontend
      dockerfile: Dockerfile
    ports:
      - "5173:5173"
    volumes:
      - ./react-18-frontend:/usr/src/app
      - /usr/src/app/node_modules
    networks:
      - laravel-network
    depends_on:
      - app

networks:
  laravel-network:
    driver: bridge

volumes:
  mysql_data:
```

Then in the root folder where the `docker-compose.yml` file is, create three folders if they dont exist already.`laravel-11-backend` for your laravel folder. `react-18-frontend` for the react app and `nginx` for the nginx files.

**Breakdown of `docker-compose.yml`:**

1. `app` Service (Laravel Backend):

    Builds from `./laravel-11-backend` using `Dockerfile`.
    Exposes port `9000` internally, mapped to `8000` externally on the local machine.
    Mounts Laravel directories for code and storage.
    Depends on mysql service and connects to by setting `DB_HOST` environment variable.

2. `mysql` Service:

    Uses MySQL 8.0 image with predefined environment variables for database setup.
    Exposes port `3306` for database connections.
    Persists MySQL data to `mysql_data` volume.

3. `nginx` Service:

    Builds from ./nginx folder using its Dockerfile.
    Forwards port 80 to serve Laravel's public directory.
    Mounts Laravel directory to serve PHP files through PHP-FPM.

4. `frontend` Service (React Frontend):

    Builds from ./react-18-frontend using Dockerfile.
    Exposes port 5173 for React development server.
    Mounts React source code and node_modules for live updates.

5. Networks and Volumes:

    Defines a laravel-network for communication between services.
    Creates mysql_data volume for MySQL data persistence.

#### Dockerfiles and Configuration Files
Here are the Dockerfiles and configuration files used in the setup:

<br/>

* **Laravel Backend Dockerfile (laravel-11-backend/Dockerfile):**

```
FROM php:8.2-fpm

RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    zip \
    unzip \
    libonig-dev \
    libxml2-dev \
    netcat-openbsd \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd pdo pdo_mysql mbstring exif pcntl bcmath xml

# Install Composer
COPY --from=composer:2.2 /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www/html

# Copy existing application directory contents
COPY . .

# Copy entrypoint script
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

# Install Laravel dependencies
RUN composer install --optimize-autoloader --no-dev

# Expose port 9000
EXPOSE 9000

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
```

**Breakdown of Laravel Backend Dockerfile:**

1. Base Image: `php:8.2-fpm`

    Uses PHP 8.2 with FPM (FastCGI Process Manager) for handling PHP requests efficiently.

2. Dependencies Installation:

    Installs essential packages like `git`, `curl`, `zip`, `unzip`, etc., necessary for Laravel and Composer operations.
    Installs specific PHP extensions (`gd`, `pdo_mysql`, `mbstring`, etc.) required by Laravel for image processing, database connectivity, and string manipulation.

3. Composer Installation:

    Copies Composer from the Composer image (`composer:2.2`) into the PHP image, making it globally accessible.
    Allows dependency management and package installation within the container.

4. Working Directory and Application Setup:

    Sets the working directory within the container to `/var/www/html`, the standard directory for web applications in many PHP environments.
    Copies all application files into the container, including Laravel's codebase.

5. Permissions and Laravel Setup:

    Adjusts file ownership (`www-data:www-data`) to ensure Laravel can read and write necessary files.
    Installs Laravel dependencies using Composer, optimizing the autoloader and excluding development packages (`--optimize-autoloader --no-dev`).

6. Port Exposition:

    Exposes port `9000` to allow PHP-FPM to communicate with Nginx for processing PHP scripts.

7. Entrypoint Script:

    Copies an entrypoint script (`entrypoint.sh`) that waits for MySQL to be ready before executing commands like database migrations and seeding.
    Executes PHP-FPM to serve PHP applications.

<br/>

* **Nginx Dockerfile (nginx/Dockerfile):**

```
FROM nginx:1.21.6

# Copy the Nginx configuration file
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 80
EXPOSE 80
```

**Breakdown of Nginx Dockerfile:**

1. Base Image: `nginx:1.21.6`

    Uses the official Nginx image version `1.21.6` from Docker Hub as the base.

2. Custom Configuration:

    Copies a custom Nginx configuration file (`nginx.conf`) to replace the default configuration.
    Configures Nginx to serve the Laravel application located in `/var/www/html/public`, directing traffic to PHP-FPM for PHP script processing.

3. Port Exposition:

    Exposes port `80` to allow external HTTP traffic to access the web application served by Nginx.

<br/>

* **Nginx Configuration (nginx/nginx.conf):**
```
server {
    listen 80;
    server_name localhost;

    root /var/www/html/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

<br/>

* **React Frontend Dockerfile (react-18-frontend/Dockerfile):**
```
FROM node:18-alpine

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5173

CMD ["npm", "run", "dev"]
```

**Breakdown of React Frontend Dockerfile:**

1. Base Image: `node:18-alpine`

    Uses Node.js version `18` based on Alpine Linux, a lightweight and secure base image for Node.js applications.

2. Working Directory and Dependencies:

    Sets the working directory within the container to `/usr/src/app`, a common practice for Node.js applications.
    Copies `package.json` and `package-lock.json` to install dependencies separately from application code to leverage Docker's layer caching.

3. Dependency Installation:

    Runs `npm install` to install all dependencies listed in `package.json`, optimizing for Docker builds.

4. Application Setup:

    Copies all application code (`COPY . .`) into the container after installing dependencies.
    This step ensures that changes in the application code trigger rebuilds of Docker layers that depend on it, speeding up development workflows.

5. Port Exposition:

    Exposes port `5173`, which is typically used by React's development server (`npm run dev`), allowing external access during development.

6. Command to Run Application:

    Specifies the default command (`CMD`) to start the application using `npm run dev`, which runs the React development server.

#### Vite Configuration
If you happen to be usng `vite` for building your react app, make sure your Vite configuration (vite.config.js) is set up to allow the server to be accessed from outside the container. Update your vite.config.js as follows:
```
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    host: true, // This allows Vite to accept connections from outside the container
    port: 5173, // The port Vite will run on
    strictPort: true, // If the port is already used, Vite will fail instead of picking another port
    watch: {
      usePolling: true, // This is needed to work well inside Docker
    },
  },
})
```

#### Conclusion
Each Dockerfile serves a crucial role in containerizing different components of your application stack. The Laravel backend Dockerfile configures PHP and Composer dependencies, the Nginx Dockerfile customizes Nginx for serving PHP applications, and the React frontend Dockerfile sets up Node.js for developing and serving React applications. Together with Docker Compose, these files orchestrate a unified development environment, ensuring consistency across different machines and simplifying the setup for your Laravel 11 and React 18 application development.

This setup ensures that your development environment closely mirrors your production setup, facilitating smoother deployments and collaboration.
