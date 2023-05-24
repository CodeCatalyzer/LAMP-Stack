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

docker build -t example-app ./


# Docker --> runs the docker binary
# run --> creates a container from an image and runs the container ~ -p --> Publish a container’s port(s) to the host ~ 8080:80 --> the port where the site will be hosted
# name (example-app) --> use the image named example-app 

docker run -p 8080:80 example-app
```

Now you will be able to see the following when visiting localhost:8080:  
![afbeelding](https://github.com/CodeCatalyzer/LAMP-Stack/assets/112801788/7a0a99ca-ce1e-482e-9947-19d57dc19675)  

Now you can start your new awesome programs using PHP!  
.  
.  
Just kidding.. When you change something in your php script, you will see that the browser doesn't automatically update. This is because you have to build the container when you made a change. Imagine a classroom. Students walk in the classroom and take a seat. The teacher only knows how many students are in the classroom the moment he counts them. This is the build command in docker. If a student walks in after the teacher had counted the amount of students, he has to count again because otherwise he won't have the latest amount of students.



