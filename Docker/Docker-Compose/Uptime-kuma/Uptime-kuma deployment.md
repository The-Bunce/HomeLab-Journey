#  Homelab Issue
# <img src="/Images/Docker-Images/Uptime-kuma.png" width="800" height="500"/>
The issue I have identified in my homelab is the ability to monitor hosts to identify when issues occur, there was a choice between checkmk and Uptime-Kuma, I decided on Uptime-Kuma as its simplier and will provide the services i require. The display/dashboard created from Uptime-Kuma will be displayed on the LRM-7521 Displays within Main-Rack 2 

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step By Step Guide

Here is a step by step breakdown on how to get Uptime-Kuma up and running and how to add services for monitoring

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 1

First we need to create the docker-compose folder location (if you have already made one cd into it)
```bash
mkdir docker-compose && cd docker-compose
```

now we need to make the repository where the docker-compose file will be stored
```bash
mkdir Uptime-kuma && cd Uptime-kuma
```

now we need to create the docker compose file 
```bash
nano docker-compose.yaml
```
```bash
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    user: 1000:1000
    volumes:
      - /home/${VOLUMES_USER}/docker/uptime-kuma:/app/data
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      proxy:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.${TRAEFIK_ROUTER_NAME}.entrypoints=http"
      - "traefik.http.routers.${TRAEFIK_ROUTER_NAME}.rule=Host(`{DOMAIN_NAME}`)"
      - "traefik.http.middlewares.${TRAEFIK_ROUTER_NAME}-https-redirect.redirectscheme.scheme=https"
      - "traefik.http.routers.${TRAEFIK_ROUTER_NAME}.middlewares=${TRAEFIK_ROUTER_NAME}-https-redirect"
      - "traefik.http.routers.${TRAEFIK_ROUTER_NAME}-secure.entrypoints=https"
      - "traefik.http.routers.${TRAEFIK_ROUTER_NAME}-secure.rule=Host(`{DOMAIN_NAME}`)"
      - "traefik.http.routers.${TRAEFIK_ROUTER_NAME}-secure.tls=true"
      - "traefik.http.routers.${TRAEFIK_ROUTER_NAME}-secure.service=${TRAEFIK_ROUTER_NAME}"
      - "traefik.http.services.${TRAEFIK_ROUTER_NAME}.loadbalancer.server.port=3001"
      - "traefik.docker.network=proxy"
  env_file:
    - .env
networks:
  proxy:
    external: true
```
Ctrl + X && Y

now we need to create the .env file (this will be where we give the variables listed above values)
```bash
nano .env
```
```bash
VOLUME_USER=
DOMAIN_NAME=
TRAEFIK_ROUTER_NAME=
```
Ctrl + X && Y

now you will need to go to the URL Uptime-kuma is hosted on, it will ask you to create a user 
once that is done you should now be able to add hosts for monitoring 


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 2

To add a host for monitoring click the add host button on the dashboard 