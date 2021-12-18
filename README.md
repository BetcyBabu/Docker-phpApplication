# Dockerizing PHP Application

## 1. Create a directory for your project and switch into the directory

mkdir phpApplication
cd phpApplication

## 2. Download your PHP website's file to a folder, say website

git clone https://github.com/BetcyBabu/sample-website.git website

## 3. Copy SSL certificates, private for your PHP application into cert.crt and  files.

## 4. Create a new network

docker network create mynetwork

## 5. Create PHP container

docker container run --name php -d -v $(pwd)/website/:/var/www/html/ --network mynetwork --restart always php:fpm-alpine3.15

## 6. Create default.conf using the below contents

server {
    listen 80;
    return 301 https://seblog.tech;
}

server {

    listen 443 ssl;
    server_name seblog.tech;

    ssl_certificate           /etc/nginx/cert.crt;
    ssl_certificate_key       /etc/nginx/cert.key;


    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;

    root /var/www/html;
    index index.php;
    server_tokens off;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param SCRIPT_NAME $fastcgi_script_name;
  }
}

Replace seblog.tech with your domain name

## 7. Create nginx container

docker container run -d --name nginx -v $(pwd)/default.conf:/etc/nginx/conf.d/default.conf -v $(pwd)/website/:/var/www/html/ -v $(pwd)/cert.crt:/etc/nginx/cert.crt -v $(pwd)/cert.key:/etc/nginx/cert.key -p 80:80 -p 443:443 --network mynetwork --restart always nginx:alpine


