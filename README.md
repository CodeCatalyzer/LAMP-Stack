# LAMP-Stack
In this tutorial i will teach you how to make a (Linux-Apache-MySQL-PHP) stack in Docker. 

At first you start with installing Docker Desktop from https://www.docker.com/
If you have never used docker before it's a good practice to first finish the docker tutorial.
I will use terms like docker-image that are explained in the tutorial
You can do this by typing the following command in your terminal:  
  
`docker run -dp 80:80 docker/getting-started`  
  
If done right, you will now see the following when you go to localhost in your browser.

# PHP
To start off the tutorial, go to your file explorer and create a folder.
In this folder you will want to add an index.php and a Dockerfile. (The 'D' in Dockerfile has to be Capitalized)
Inside the index.php file you will add the following code:

```php
<?php

echo "<h1> Hello World </h1>";
```

When you run this in your browser, it will return the source code. The reason for this is that the browser can't do anything with this php file. 
The HTML inside the file can only be obtained if php is configured. We can easily fix this. In your Dockerfile, add the following lines of code.  

```php
# Get information from the latest image version of php:apache
FROM php:apache

# Copy the contents of the current directory to the /var/www/html directory of the container
COPY . /var/www/html
```

Now you can run the following command in your terminal to see if it works:  

```php
# Docker --> runs the docker binary
# build --> builds an image from a Dockerfile
# -t (name) --> tags the image with a name (here we used example-app)
# ./ --> builds the image from all of the files in this directory

docker build -t myapp ./


# Docker --> runs the docker binary
# run --> creates a container from an image and runs the container ~ -p --> Publish a container’s port(s) to the host ~ 8080:80 --> the port where the site will be hosted
# name (myapp) --> use the image named myapp

docker run -p 8080:80 myapp
```

Now you will be able to see the following when visiting localhost:8080:  
![afbeelding](https://github.com/CodeCatalyzer/LAMP-Stack/assets/112801788/73f84b72-489e-4619-9e6b-f58a0aef4306)


Now you can start your new awesome programs using PHP!  
.  
.  
Just kidding.. When you change something in your php script, you will see that the browser doesn't automatically update. This is because you have to build the container when you made a change. Imagine a classroom. Students walk in the classroom and take a seat. The teacher only knows how many students are in the classroom the moment he counts them. This is the build command in docker. If a student walks in after the teacher had counted the amount of students, he has to count again because otherwise he won't have the latest amount of students. 

To see change in your browser, you will have to stop the container in the docker desktop and afterwards execute the `docker build` and `docker run` commands again.  

![afbeelding](https://github.com/CodeCatalyzer/LAMP-Stack/assets/112801788/aa5e927a-6450-441e-9794-974e9c04da1a)


## Volumes 
A Docker volume is like a special storage place for containers. It keeps important data safe even if the containers are turned off or taken away You can easily share this data between different containers or even on different computers. It helps make sure everything works smoothly and keeps your information organized. 
There is more information about bind mounts on the following website: https://docs.docker.com/storage/bind-mounts/

There are multiple ways to sync the container with your folders.  

### Way 1: With Dockerfile  
This is the best if you also want to use another docker-image in your application. For example a mysql database.
In your `Dockerfile` remove the `COPY` line. 
You can now run a new container with the following commands: 

```php
docker build -t myapp ./

# -v --> bind mount a volume
# ${PWD}:/var/www/html --> sync the current directory with /var/www/html
# myapp --> use the image named 'myapp'

docker run -p 8080:80 -v ${PWD}:/var/www/html myapp
```

## Way 2: Without Dockerfile
This way is easy if you only want to run php in your application
Just add the following command to your terminal: 

```php
docker build -t myapp ./

# -v --> bind mount a volume
# ${PWD}:/var/www/html --> sync the current directory with /var/www/html
# php:apache --> use the php:apache image from docker

docker run -v ${PWD}:/var/www/html php:apache
```

If you did it right, the code you add to your php file will automatically update when you refresh your browser:  
![afbeelding](https://github.com/CodeCatalyzer/LAMP-Stack/assets/112801788/e7527485-a930-4994-882f-19a7df615fe5)

# MySQL

Next up we want to be able to run a database with the php code. Therefore we have to configure MySQL into Docker. You can ofcourse also decide
to use DBMS like MariaDB. To check if mySQL works, add the following code to `index.php`:  
```php
<?php
$servername = "aServer";
$username = "root";
$password = "aPassword";
$database = "aDatabase";

try {
    $conn = new PDO("mysql:host=$servername;dbname=$database", $username, $password);
    $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    echo "Connected successfully";
} catch(PDOException $e) {
    echo "Connection failed: " . $e->getMessage();
}
```
Now if done correctly, the browser will give the following message: `connection failed: could not find driver`.
Just like we did with php we have to make a new custom image. In your `rootfolder` create a subfolder called `database`. In this folder
add a new `Dockerfile`

In the `Database Dockerfile` add the following code:
```php
FROM mysql:8.0
```

In the `PHP Dockerfile` add the following code: 
```php
# RUN --> runs a command for example (docker-php-ext-install)
# docker-php-ext-install --> installs a extension for php, here it is pdo and pdo_mysql

RUN docker-php-ext-install pdo pdo_mysql
```

Now we can `build` the images with the following commands: 
```php
docker build -t myapp ./
docker build -t mydatabase ./database
```
And now `run` the images:
```php
#--name aServer --> assigns the name aServer to the image
# -e MYSQL_ROOT_PASSWORD=aPassword --> sets the variable value of MYSQL_ROOT_PASSWORD to aPassword
# database --> name of the image

docker run --name mydatabase -e MYSQL_ROOT_PASSWORD=aPassword mydatabase
docker run -p 8080:80 -v ${PWD}:/var/www/html myapp
```
If everything is done correctly, refreshing your browser will give the following error code:  
![afbeelding](https://github.com/CodeCatalyzer/LAMP-Stack/assets/112801788/ade5ad44-d619-4d48-b827-d4b66246d006)

This is pretty logical. We did in fact install the mysql extension to PHP but the database image and php image are not linked together.
They are now just 2 seperate images being run in different containers. To fix this we will introduce networks

## Networks
A Docker network is like a magic road that connects different containers together. It lets them talk to each other and share information. Just like how friends can send messages or talk on the phone, containers can communicate through the Docker network. It helps them work together as a team and share things they need, like toys or snacks. The Docker network makes sure everyone can connect and play nicely together.

Execute the following commands to kill all the existing containers and create a network:

```php
# kills all existing containers 
docker system prune

# Creates a network with the name aNetwork
# -d(detached) -->  will make the container run in the background without showings its output
docker network create -d bridge aNetwork
```

Instead of having to add -e variables to your run command you can just add them in your MySQL dockerfile:
```php
FROM mysql:8.0

ENV MYSQL_ROOT_PASSWORD=aPassword
ENV MYSQL_DATABASE=aDatabase
```

Now run the php and database images again, but add the network to it: 

```php
docker build -t mydatabase ./database
docker run --network myNetwork --name aServer -p 3306:3306 mydatabase
docker run --network myNetwork -p 8080:80 -v ${PWD}:/var/www/html myapp
```
Now you will get `Connected succesfully`

![afbeelding](https://github.com/CodeCatalyzer/LAMP-Stack/assets/112801788/e12cddd1-0923-47a6-8d2f-232b1375f6cb)


# Laravel application via Docker

To run a laravel application in Docker you will need the following things, php, mysql, node and composer, as laravel needs you to do the npm install and composer install commands to work. Inside your folder create a new laravel project. 

## Node and Composer

Installing node and composer is very easy to do. Just like we did with the mysql dockerfile, create a folder named the way you want. I named my folders composer and node. Inside the folders create a new Dockerfile.

Inside the composer Dockerfile type the following: 

```php
# Uses the latest composer image
FROM composer:latest
# This is the work directory where the application code gets stored
WORKDIR /var/app
# Executes the composer install command
CMD ["composer",  "install"]
```

Inside the node Docker file type the following:

```php
# Uses node:20-alpine image
FROM node:20-alpine

WORKDIR /var/app/
# Execute the command npm install
CMD [ "npm", "install" ]
```

The extra build and run commands will be:
```php
docker build -t mycomposer ./composer
docker build -t mynode ./node

docker run --name mycomposer -d -v ${PWD}/src/:/var/app/ mycomposer
docker run --name mynode -d -v ${PWD}/src/:/var/app/ myNode
```

Running all the build and run command will not do the trick yet. If we run this the program will have a couple problems. 

Because this time the index.php file is not directly in the root folder, but inside the public folder of our project we have to make some tweaks to the program. This is what the changed dockerfile will look like: 

```php
FROM php:8.2-apache

RUN docker-php-ext-install pdo pdo_mysql
# Sets the document root to the public folder in your laravel application
ENV APACHE_DOCUMENT_ROOT=/var/www/html/(projectfolder)/public

#These RUN commands modify the Apache configuration files in the Docker image to dynamically replace certain file path references with the value of the ${APACHE_DOCUMENT_ROOT} environment variable.

RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

RUN sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf
RUN a2enmod rewrite

WORKDIR /var/www/html/
```

Et Voilà, you are done! This are the commands i run now to run my laravel application:
```php
# Build the images
docker build -t myapp ./
docker build -t mydatabase ./database
docker build -t mycomposer ./composer
docker build -t mynode ./node

# Create the network
docker network create -d bridge myNetwork

# Run the containers
docker run --name mydatabase --network myNetwork -d -v ${PWD}/data/:/var/lib/mysql/ mydatabase
docker run --name myapp --network myNetwork -d -p 8080:80 -v ${PWD}/src/:/var/www/html/ myapp
docker run --name mycomposer -d -v ${PWD}/src/:/var/app/ mycomposer
docker run --name mynode -d -v ${PWD}/src/:/var/app/ mynode
```
![afbeelding](https://github.com/CodeCatalyzer/LAMP-Stack/assets/112801788/96dc8577-e18a-4b8f-b884-85890cc789d1)


# Your laravel application is ready to go!
