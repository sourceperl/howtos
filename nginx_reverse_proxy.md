# Setup a reverse proxy with Nginx


## Setup

```bash
# install nginx
sudo apt install nginx
# remove default conf
sudo rm /etc/nginx/sites-enabled/default
# add configuration file for reverse proxy 
cat <<EOF | sudo tee -a /etc/nginx/proxy.conf
# this is upstream grafana
upstream backend_grafana {
  server 127.0.0.1:3000;
}

# redirect HTTP to HTTPs
server {
    listen 80 default_server;
    #listen [::]:80 default_server;
    return 301 https://$host$request_uri;
}

# define HTTPs end-point
server {
  listen 443 ssl;
  #listen [::]:443 ssl;
  server_tokens off;
  ssl_certificate /etc/nginx/ssl/https.crt;
  ssl_certificate_key /etc/nginx/ssl/https.key;

  location / {
    include proxy_params;
    proxy_pass http://backend_grafana;
  }
}
EOF
# activate proxy configuration
sudo ln -s /etc/nginx/sites-availables/proxy.conf /etc/nginx/sites-enabled/
# create certificates directories
sudo mkdir /etc/nginx/ssl/
# populate it with self-signed HTTPs cert
sudo openssl -x509 -nodes -days 3650 -newkey rsa:4096 -keyout /etc/nginx/ssl/https.key -out /etc/nginx/ssl/https.crt
# restart nginx
sudo systemctl restart nginx.service
```
