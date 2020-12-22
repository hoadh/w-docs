# Cài đặt VPS

## Docker & Docker-compose

### Docker

```
# Làm theo các bước ở link: https://docs.docker.com/engine/install/ubuntu/
# Xóa phiên bản cũ: sudo apt-get remove docker docker-engine docker.io containerd runc

$ sudo apt-get update

$ sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io -y



# Bước cuối cùng
sudo usermod -aG docker $(whoami)

# Logout và đăng nhập lại
sudo reboot
```

### Docker-composer

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

```


## Caddy

```bash
echo "deb [trusted=yes] https://apt.fury.io/caddy/ /" \
    | sudo tee -a /etc/apt/sources.list.d/caddy-fury.list
sudo apt update
sudo apt install caddy
```

Cập nhật lại Caddyfile

```
sudo nano /etc/caddy/Caddyfile
```


```
sudo systemctl restart caddy
sudo systemctl status caddy
```



## Cài đặt WP qua Docker


Bổ sung các file dưới đây vào thư mục home

### uploads.ini

```
file_uploads = On
memory_limit = 64M
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 600
```

### .env

```
WEB_SOCK=127.0.0.1:8000

DB_USER=wordpress
DB_PASSWORD=my-db-pwd
DB_NAME=wordpress
```

### docker-compose.yaml

```
version: '3.1'

services:

  app:
    image: wordpress
    restart: unless-stopped
    depends_on:
      - db
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: $DB_USER
      WORDPRESS_DB_PASSWORD: $DB_PASSWORD
      WORDPRESS_DB_NAME: $DB_NAME
    ports:
      - $WEB_SOCK:80
    volumes:
      - ./app:/var/www/html
      - ./uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
 
  db:
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: $DB_NAME
      MYSQL_USER: $DB_USER
      MYSQL_PASSWORD: $DB_PASSWORD
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - ./db:/var/lib/mysql

volumes:
  app: {}
  db: {}
```

### Tạo các thư mục để mount volumne trong containers

```
mkdir app
sudo chown -R 1001:1001 app

mkdir db
sudo chown -R 1001:1001 db

```

### Up
```
docker-compose up -d
```

