# Build CICD Pipeline with AWS and Github Actions

## Prepare the project
Example of Dockerfile:
```dockerfile
From golang:1.22

WORKDIR /go/src/app

COPY . .

RUN go build -o main src/main.go

CMD ["./main"]
```

## Build Docker Image
```shell
docker build -t ryqdev/xxxxx .
```

## Run Docker Image
```shell
docker run -d -p 8080:8080 --name cicd-pipeline-container ryqdev/xxxxx
```

## Build CI Pipeline
Under `.github/workflows` directory, add the ci file:
```yaml
# ci.yml
name: CI Pipeline

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Login Dockerhub
      env:
        DOCKER_USERNAME: ${{secrets.DOCKER_USERNAME}}
        DOCKER_PASSWORD: ${{secrets.DOCKER_PASSWORD}}
      run: docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
      
    - name: Build the Docker image
      run: docker build -t ryqdev/cicd-pipeline .
    - name: Push to Dockerhub
      run: docker push ryqdev/cicd-pipeline:latest
```

## Build CD Pipeline
Under `.github/workflows` directory, add the cd file:
```yaml
# cd.yml
name: CD Pipeline
on:
  workflow_run:
    workflows: ["CI Pipeline"]
    types:
      - completed
jobs:
  build:
    runs-on: self-hosted

    steps:
    - name: Pull Docker image
      run: sudo docker pull ryqdev/cicd-pipeline:latest
    - name: Delete Old docker container
      run: sudo docker rm -f cicd-pipeline-container || true
    - name: Run Docker Container
      run: sudo docker run -d -p 8080:8080 --name cicd-pipeline-container ryqdev/cicd-pipeline
```

## Setup AWS EC2
### Add Github Runner
<img width="2555" alt="image" src="https://github.com/user-attachments/assets/a128ffe2-bb3f-4845-92c9-e66ae162b2a3">

Run the commands in EC2:
<img width="1382" alt="image" src="https://github.com/user-attachments/assets/09425d6c-10ef-46a4-9d37-205a320fcf86">


### Install docker in EC2
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04

### Install nginx in EC2
```shell
sudo apt install nginx
```

Get the docker container's ip address:
```shell
docker inspect \
  -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <containter id>
```

Modify `location / {` in `/etc/nginx/sites-available`:
```text
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	# SSL configuration
	#
	# listen 443 ssl default_server;
	# listen [::]:443 ssl default_server;
	#
	# Note: You should disable gzip for SSL traffic.
	# See: https://bugs.debian.org/773332
	#
	# Read up on ssl_ciphers to ensure a secure configuration.
	# See: https://bugs.debian.org/765782
	#
	# Self signed certs generated by the ssl-cert package
	# Don't use them in a production server!
	#
	# include snippets/snakeoil.conf;

	root /var/www/html;

	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		proxy_pass http://172.17.0.2:8080;
	}

	# pass PHP scripts to FastCGI server
	#
	#location ~ \.php$ {
	#	include snippets/fastcgi-php.conf;
	#
	#	# With php-fpm (or other unix sockets):
	#	fastcgi_pass unix:/run/php/php7.4-fpm.sock;
	#	# With php-cgi (or other tcp sockets):
	#	fastcgi_pass 127.0.0.1:9000;
	#}

	# deny access to .htaccess files, if Apache's document root
	# concurs with nginx's one
	#
	#location ~ /\.ht {
	#	deny all;
	#}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#	listen 80;
#	listen [::]:80;
#
#	server_name example.com;
#
#	root /var/www/example.com;
#	index index.html;
#
#	location / {
#		try_files $uri $uri/ =404;
#	}
#}
```

Restart nginx
```shell
sudo systemctl restart nginx
```

### Access the Public IPv4 DNS
Get the Public IPv4 DNS from AWS console: `ec2-13-60-74-205.eu-north-1.compute.amazonaws.com`
<img width="893" alt="image" src="https://github.com/user-attachments/assets/9f6f2676-5520-4f5c-95ab-df5c3f7784fd">


If you cannot access the server, check if you enable the http port to all ips in `Security Group`
<img width="1666" alt="image" src="https://github.com/user-attachments/assets/ccf8068c-1f3b-4d96-80bf-1784f472b59e">
