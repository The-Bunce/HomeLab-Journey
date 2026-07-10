#  Homelab Issue
# <img src="/Images/Docker-Images/Netbox-Image.png" width="800" height="500"/>
I have noticed that I have started to use AI models quite a lot recently, so I have decided to deploy my own AI system

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Requirements
GPU
Virtual Machine
GPU Passthrough enabled on Proxmox (follow Guide)
Supported GPUs
# <img src="/Images/Docker-Images/AISetup1.png" width="800" height="500"/>

GPU I am using will be an RTX 3090 

# <img src="/Images/Docker-Images/AISetup2-GPU.png" width="800" height="500"/>

OS 
The OS I will be using is Ubuntu 

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 1

Creating and setting up the Virtual Machine

Go to Proxmox (you will need to ensure the GPU in your system is assigned to the vm that will be hosting the AI)


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 2

Installing drivers that are required

first as its a new machine 


```bash
sudo apt update && sudo apt upgrade -y
```
after this reboot the machine

***if you get an error saying there is no ubuntu-drivers run the following command
```bash
sudo apt install ubuntu-drivers-common
```

now do a 
```bash
sudo ubuntu-drivers install
```
This will install the recommended drivers
once it is installed reboot the machine 

To Check to see if it is all working correctly use the following command
```bash
nvidia-smi
```
you should see your GPU listed


Check driver versions 
```bash
cat /proc/driver/nvidia/version
```


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 3  Installing the Nvidia toolkit

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
```bash
sudo apt-get update
```
```bash
sudo apt-get install -y nvidia-container-toolkit
```

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step  4 Install docker-compose
```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
sudo systemctl status docker
```
```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```
```bash
sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```
test to see if all is configured correctly


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 4 Folder Structure

please create the following folders on your system you can either create them on vscode right click create or on the cli by the following command 
```bash
mkdir "name"
```

-docker-compose
    -ai
        -.env
        -docker-compose.yaml
        -ollama
        -open-webui
        -searxng
        -whisper


you will want to create these folders where you would want the files located





# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 5 Folder Permission

ensure you change the below to be the user running the system ie you and the folder path of the compose file
```bash
sudo chown serveradmin:serveradmin -R /opt/stacks
```

to check ID of the user you can just run the following: 
```bash
id
```

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 6 Creating the Compose file 

on the host you can either create the new compose file using vscode -> right click create or via the cli using the following 
```bash
sudo nano docker-compose.yaml
```
ensure you are in the top level folder of the stack 

now paste the following 

```bash
services:

# Ollama

  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    environment:
      - PUID=${PUID:-1000}
      - PGID=${PGID:-1000}
      - OLLAMA_KEEP_ALIVE=1h
    networks:
      - proxy
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./ollama:/root/.ollama
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.ollama.rule=Host(`ollama.${DOMAIN_NAME}`)"
      - "traefik.http.routers.ollama.entrypoints=https"
      - "traefik.http.routers.ollama.tls=true"
      - "traefik.http.routers.ollama.tls.certresolver=${RESOLVER}"
      - "traefik.http.routers.ollama.middlewares=default-headers@file"
      - "traefik.http.routers.ollama.middlewares=ollama-auth"
      - "traefik.http.services.ollama.loadbalancer.server.port=11434"
      - "traefik.http.routers.ollama.middlewares=auth"
      - "traefik.http.middlewares.auth.basicauth.users=${OLLAMA_API_CREDENTIALS}"
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

# open web ui
  open-webui:
    image: ghcr.io/open-webui/open-webui:latest
    container_name: open-webui
    restart: unless-stopped
    networks:
      - proxy
    environment:
      - PUID=${PUID:-1000}
      - PGID=${PGID:-1000}
      - 'OLLAMA_BASE_URL=http://ollama:11434'
      - ENABLE_RAG_WEB_SEARCH=True
      - RAG_WEB_SEARCH_ENGINE=searxng
      - RAG_WEB_SEARCH_RESULT_COUNT=3
      - RAG_WEB_SEARCH_CONCURRENT_REQUESTS=10
      - SEARXNG_QUERY_URL=http://searxng:8080/search?q=<query>
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./open-webui:/app/backend/data
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.open-webui.rule=Host(`chat.${DOMAIN_NAME}`)"
      - "traefik.http.routers.open-webui.entrypoints=https"
      - "traefik.http.routers.open-webui.tls=true"
      - "traefik.http.routers.open-webui.tls.certresolver=${RESOLVER}"
      - "traefik.http.routers.open-webui.middlewares=default-headers@file"
      - "traefik.http.services.open-webui.loadbalancer.server.port=8080"
    depends_on:
      - ollama
    extra_hosts:
      - host.docker.internal:host-gateway

  searxng:
    image: searxng/searxng:latest
    container_name: searxng
    networks:
      - proxy
    environment:
      - PUID=${PUID:-1000}
      - PGID=${PGID:-1000}
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./searxng:/etc/searxng
    depends_on:
      - ollama
      - open-webui
    restart: unless-stopped


  
# whisper
  mongo:
    image: mongo
    env_file:
      - .env
    networks:
      - proxy
    restart: unless-stopped
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./whisper/db_data:/data/db
      - ./whisper/db_data/logs/:/var/log/mongodb/
    environment:
      - PUID=${PUID:-1000}
      - PGID=${PGID:-1000}
      - MONGO_INITDB_ROOT_USERNAME=${DB_USER:-whishper}
      - MONGO_INITDB_ROOT_PASSWORD=${DB_PASS:-whishper}
    command: ['--logpath', '/var/log/mongodb/mongod.log']

  translate:
    container_name: whisper-libretranslate
    image: libretranslate/libretranslate:latest-cuda
    env_file:
      - .env
    networks:
      - proxy
    restart: unless-stopped
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./whisper/libretranslate/data:/home/libretranslate/.local/share
      - ./whisper/libretranslate/cache:/home/libretranslate/.local/cache
    user: root
    tty: true
    environment:
      - PUID=${PUID:-1000}
      - PGID=${PGID:-1000}
      - LT_DISABLE_WEB_UI=True
      - LT_LOAD_ONLY=${LT_LOAD_ONLY:-en,fr,es}
      - LT_UPDATE_MODELS=True
    deploy:
      resources:
        reservations:
          devices:
          - driver: nvidia
            count: all
            capabilities: [gpu]

  whisper:
    container_name: whisper
    pull_policy: always
    image: pluja/whishper:latest-gpu
    env_file:
      - .env
    networks:
      - proxy
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./whisper/uploads:/app/uploads
      - ./whisper/logs:/var/log/whishper
      - ./whisper/models:/app/models
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whisper.rule=Host(`whisper.${DOMAIN_NAME}`)"
      - "traefik.http.routers.whisper.entrypoints=https"
      - "traefik.http.routers.whisper.tls=true"
      - "traefik.http.routers.whisper.tls.certresolver=${RESOLVER}"
      - "traefik.http.services.whisper.loadbalancer.server.port=80"
      - "traefik.http.routers.whisper.middlewares=default-headers@file"
    depends_on:
      - mongo
      - translate
    environment:
      - PUID=${PUID:-1000}
      - PGID=${PGID:-1000}
      - PUBLIC_INTERNAL_API_HOST=${WHISHPER_HOST}
      - PUBLIC_TRANSLATION_API_HOST=${WHISHPER_HOST}
      - PUBLIC_API_HOST=${WHISHPER_HOST:-}
      - PUBLIC_WHISHPER_PROFILE=gpu
      - WHISPER_MODELS_DIR=/app/models
      - UPLOAD_DIR=/app/uploads
    deploy:
      resources:
        reservations:
          devices:
          - driver: nvidia
            count: all
            capabilities: [gpu]
networks:
  proxy:
    external: true
```

now you need to ensure you change the networks prompted here as the one i am using is called proxy (yours may not be)

you will now need to amend the env file to use your data see below for generating the Ollama credential

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 7 Generating the Ollama Credential 	

echo $(htpasswd -nB ollamauser) | sed -e s/\\$/\\$\\$/g
then place this value in the .env file under the OLLAMA_API_CREDENTIALS section

	

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 8 DNS Configuration
you need to ensure you have Cnames marked and pointed correctly for the URL (Traefik) to work

I used Pi-hole for my dns so all it requires is for you to go into the Pihole dashboard -> Settings -> Local DNS Records -> put the URL you specified in the compose file for example chat.Domain and assign it to the IP of the docker host to find this on the host run the following
```bash
ip a
```


cd to the docker directory where the AI compose file is and run 
```bash
docker compose up -d --build --force-recreate --remove-orphans
```

Congratulations you now have deployed it you should now be able to go to chat.domain (replace with your domain) and create a new user 


you can get models from ollama and download them directly to the AI 