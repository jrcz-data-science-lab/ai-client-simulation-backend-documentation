# Code for the nginx setup

## Structure
```tree
.
├── fastcgi.conf 📄
├── fastcgi_params 📄
├── koi-utf 📄
├── koi-win 📄
├── mime.types 📄
├── nginx.conf 📄
├── proxy_params 📄
├── scgi_params 📄
├── sites-available 📂
│   └── aifrontend.com 📄
├── sites-enabled 📂
│   └── aifrontend.com -> /etc/nginx/sites-available/aifrontend.com 📄
├── ssl 📂
│   ├── servercert.pem 📄
│   └── serverkey.pem 📄
├── uwsgi_params 📄
└── win-utf 📄

```

## nginx.conf

Creating the configuration under which nginx will run

```nginx
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    include       sites-enabled/*;
    server_names_hash_bucket_size 64;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       8222;
        server_name  localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
}
```
## Proxy params

Creating proxy parameters

```nginx
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

## Sites available

Creating connection inside the proxies.
Here you should also create a certificate, that you put in the ssl folder. (I've used mkcert)

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name {your_ip};

    ssl_certificate /etc/nginx/ssl/servercert.pem;
    ssl_certificate_key /etc/nginx/ssl/serverkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256';

    location / {
        proxy_pass https://localhost:8888;
        include proxy_params;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_http_version 1.1;
    }

    location /api/ {
        proxy_pass http://localhost:8502/;
        include proxy_params;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_http_version 1.1;
    }
    
    location /ollama/ {
    proxy_pass http://localhost:11434/;
    include proxy_params;
    }

    
    location /streaming/ {
    proxy_pass http://localhost:8503/;
    include proxy_params;
   
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_http_version 1.1;
    }
    
    location /tts/ {
    proxy_pass http://localhost:8504/;
    include proxy_params;
    }

}

server {
    listen 80;
    listen [::]:80;

    server_name {your_ip};

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

```
When it's done then run:

```bash
sudo ln -s /etc/nginx/sites-available/aifrontend.com /etc/nginx/sites-enabled/
```
Then restart nginx:

```bash
sudo systemctl restart nginx
```