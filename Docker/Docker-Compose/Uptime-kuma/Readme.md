#  Homelab Monitoring Services & Hosts Issue
# <img src="/Images/Docker-Images/Uptime-kuma.png" width="800" height="500"/>
The issue I have identified in my homelab is the ability to monitor hosts to identify when issues occur, there was a choice between checkmk and Uptime-Kuma, I decided on Uptime-Kuma as its simplier and will provide the services i require. The display/dashboard created from Uptime-Kuma will be displayed on the LRM-7521 Displays within Main-Rack 2 

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step By Step Guide

Here is a step by step breakdown on how to get Uptime-Kuma up and running and how to add services for monitoring

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 1

First we need to create the docker-compose folder location (if you have already made one cd into it)
```bash
mkdir docker-compose && cd docker-compose
```

Now we need to make the repository where the docker-compose file will be stored
```bash
mkdir Uptime-kuma && cd Uptime-kuma
```

Now we need to create the docker compose file 
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
Ctrl + X && Y (this is to save the docker compose file)

Now we need to create the .env file (this will be where we give the variables listed above values)
```bash
nano .env
```
```bash
VOLUME_USER=
DOMAIN_NAME=
TRAEFIK_ROUTER_NAME=
```
Ctrl + X && Y

Now you will need to go to the URL Uptime-kuma is hosted on, it will ask you to create a user 
Once that is done you should now be able to add hosts for monitoring 


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 2

You will now need to gather all the hosts you want to monitor, once you have the list of the hosts you want to monitor (you need either the IP or Hostname of the service/device for example 192.168.1.23)

# <img src="/Images/Docker-Images/Uptime-kuma-adding-new-monitor.png" width="800" height="500"/>
Adding a new monitor
You will need to click on add new monitor
You will then need to fill in the following details 
Monitor type (if its a webpage use HTTPs if not you can use ping)
There will be a few choices when it comes to the monitoring types 
# <img src="/Images/Docker-Images/Uptime-kuma-monitor-list.png" width="800" height="500"/>
I will only be using Ping & HTTP(s) but feel free to use what fits for your use case
Friendly Name (this will be used for you to easily be able to identify the service/host)
URL or ip (depending on what monitoring type you chose)
Heartbeat interval (how often you want the check to occur)

!Important if you do not have signed certs ensure you tick the ignore TLS/SSL error for HTTPS websites
!Change max redirects to 0 to make this more secure

# <img src="/Images/Docker-Images/Uptime-kuma-adding-groups-and-Tags.png" width="800" height="500"/>

Now you can create groups to bundle your services and hosts together I just made three groups Production, Development and Backup 

You can also assign tags which makes it easier to identify the host, I used the tags to tag criticallity of the service 

# <img src="/Images/Docker-Images/Uptime-kuma-Slug.png" width="800" height="500"/>

you can add slug/status pages which allow you to only display hosts you need to monitor or consider to be crtical

This is how my Uptime-Kuma currently looks, it will be getting updated when I add more services into my environment
# <img src="/Images/Docker-Images/Uptime-kuma-final example.png" width="800" height="500"/>