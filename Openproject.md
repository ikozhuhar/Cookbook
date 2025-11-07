#### :white_check_mark: _NGINX_

```ruby
server {
	listen 443 ssl;
	ssl_certificate     /etc/nginx/ssl/server.crt;
	ssl_certificate_key /etc/nginx/ssl/server.key;
	server_name openproject.mip.ru;
	location / {
		proxy_pass http://127.0.0.1:8080;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP  $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_set_header X-Forwarded-Proto https;
		proxy_redirect    off;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";
	}
    client_max_body_size 100M;
    proxy_connect_timeout 300;
    proxy_send_timeout 300;
    proxy_read_timeout 300;
}
server {
	listen 80;
	listen [::]:80;
	server_name openproject.mip.ru;
	return 301 https://$host$request_uri;
}
```



#### :white_check_mark: _.env_

```ruby
# OpenProject Settings
OPENPROJECT_HOST__NAME=openproject.example.com
SECRET_KEY_BASE=83abe353bcf724db06460e34938ea09d5a2c0d268252074ffbadc8c82a53790e77ae316c98225079adae71fa178d2a933fc5c7edf5637585f016247c47e01c22

# PostgreSQL Settings
POSTGRES_DB=openproject
POSTGRES_USER=openproject
POSTGRES_PASSWORD=TrvsJ4pWHN7uq3U
```


#### :white_check_mark: _docker-compose.yml_

```ruby
services:

  postgres:
    image: mrcojuhari/postgres:17
    container_name: openproject_postgres
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - openproject_pgdata:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - openproject-net

  openproject:
    image: mrcojuhari/openproject:latest
    container_name: openproject
    depends_on:
      - postgres
    ports:
      - "8080:80"
    environment:
      - OPENPROJECT_HOST__NAME=${OPENPROJECT_HOST__NAME}
      - SECRET_KEY_BASE=${SECRET_KEY_BASE}
      - OPENPROJECT_HTTPS=true
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
    volumes:
      - openproject_assets:/var/openproject/assets
    restart: unless-stopped
    networks:
      - openproject-net

volumes:
  openproject_pgdata:
  openproject_assets:

networks:
  openproject-net:
    driver: bridge
```

```ruby
# Проверяем docker-compose.yml
sudo docker-compose config
sudo docker-compose config --help
sudo docker-compose config --service
sudo docker-compose config --volumes
```

<img width="1774" height="239" alt="image" src="https://github.com/user-attachments/assets/a981afbe-570c-4566-a0ec-662d370918ac" />

<img width="1788" height="147" alt="image" src="https://github.com/user-attachments/assets/4834db82-7c05-4c89-8061-2c4ad2a1fd48" />





















