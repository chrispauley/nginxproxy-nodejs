# nginxproxy-nodejs

This is a bare minimum nodejs server app that is wrapped in a Docker image.

## Installation
Install Docker

Node JS is not required on your development machine, that is part of this image.

## Create the nodejs image

First create your site.
```
mkdir nginxproxy-nodejs
cd nginxproxy-nodejs
mkdir nodejs nginx
cd nodejs
```

Here we will use index.js to server 'hello world' text message from our express server.
```
var http = require('http');

var server = http.createServer(function (request, response){
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.end("Hello World from nodejs in a Docker container.");
});

server.listen(3000);

console.log("Server running at http://127.0.0.1:3000/");
```

### Create the Dockerfile
The contents of the Dockerfile build script will reference

```
# Use the base image created and maintained by mhart
# It has a bare minimum installation the server and the libraries needed for an expressjs server.
FROM mhart/alpine-node
COPY index.js .
EXPOSE 3000
CMD node index.js
```

Create and run a container from the image. This will build a container with the tag "chrispauley/node-app" and the dot indicates the Dockerfile location.
```
cd nodejs
docker build -t chrispauley/node-app .
```

Next, switch to the nginx directory.
```
cd ../nginx
touch default.conf
```

Create the nginx server configuration file for the proxy.
default.conf

```
server {
  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_pass http://app:3000;
  }
}
```

Create the Dockerfile for nginx
```
FROM nginx
COPY default.conf /etc/nginx/conf.d/
```

Then build and view the image on your machine
```
docker build -t chrispauley/nginx-proxy .
docker images
```

## Run and test your image in a container
```
docker run -ti -p 3000:3000 --name node-app chrispauley/node-app
docker run -d -p 8000:80 --link node-app:app --name nginx-proxy chrispauley/nginx-proxy
```
Switch to your browser and get: [http://127.0.0.1:8000/](http://127.0.0.1:8000)


## Clean up
Kill the running process
```
docker kill nginx-proxy
docker kill node-app
```

List container instances to be removed
```
docker ps -a
```

Remove container instances
docker rm <container id>
```
docker rm ea0ea
```

Remove images
```
docker rmi chrispauley/node-app
docker rmi chrispauley/nginx-proxy
docker rmi mhart/alpine-node
```

## Bonus
Add to your github repo
```
git init
git add .
## create your github repo
git remote add origin https://github.com/chrispauley/nginxproxy-nodejs.git
git push origin master
```

### Deploy to a Docker Host
Login to your server host. At a terminal window
```
git clone https://github.com/chrispauley/nginxproxy-nodejs.git
cd nginxproxy-nodejs
docker build -t express-helloworld .
docker run -ti -p 8000:8000 express-helloworld
```
Make sure your server host port 8000 is accessible and test it:
 [http://localhost:8000/](http://localhost:8000/)
Update a cname record on your domain and your up.

## Clean Up
Remember to clean up your containers and images!
