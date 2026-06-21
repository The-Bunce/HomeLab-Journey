#  Homelab Issue
# <img src="/Images/Docker-Images/Netbox-Image.png" width="800" height="500"/>
The issue i have identified in my homelab is the ability to document the layout, connections and virtual machines running within it. I took some time to research tooling that would fit my use case, i decided on Netbox as it seems to be very versatile and could come into use in an enterprise environment. 

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step By Step Guide

Here is a step by step breakdown on how to get netbox up and running and how to change the default passwords

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 1

First step is to make the project directory this will be stored in (there will be a lot folders and files copied over)

```bash
mkdir -p ~/projects && cd projects
```
# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 2

Now we need to clone the release branch variant of netbox community edition 

```bash
git clone -b release https://github.com/netbox-community/netbox-docker.git
```

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 3

Now we nee to move into the netbox-docker folder

```bash
cd netbox-docker
```
Now we need to copy the override example and make our own (you can leave it default aswell)

```bash
cp docker-compose.override.yml.example docker-compose.override.yml
```
```bash
sudo docker compose pull
```
We will not be running the below command yet as the configuration will be reset when resetting the passwords
```bash
docker compose exec netbox /opt/netbox/netbox/manage.py createsuperuser
```

Make sure you are in the netbox folder (you should be)
and now run 

```bash
docker compose up
```
# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 4

Now netbox has been deployed we will now need to reset the passwords to do this we will be doing the following
make a copy of the standard env files (just incase you make a mistake)
do the below to all env files 

```bash
cd env/
```
```bash
cp redis.env redisbackup.env
```

We will be changing the following variables within the .env files 
DB_Password
API_TOKEN_PEPPER_1
REDIS_CACHE_PASSWORD
REDIS_PASSWORD
SECRET_KEY

ensure you mirror changes done in netbox.env in the other .env files they need to match

for your information the postgres.env uses the db username & password

to generate new random passwords i used the following commands, but feel free to use which ever tool you like 
```bash
openssl rand -base64 32
```
```bash
openssl rand -base64 50
```

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 5

Once you have change the passwords to your desired ones 
please run the following command (it will remove the database)

```bash
docker compose down -v
```

```bash
docker compose up
```

Netbox will fail to start on the first boot due to the following: 
PostgreSQL had to:
initialise a fresh database,
create roles,
run NetBox migrations,
initialise NetBox,
create caches/queues.
basically Netbox started before Postgres was fully ready


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 6

Now you need to down the docker container and restart it 

```bash
docker compose down
```

wait for it to finish 

Now run the below command it will take some time but be patient

```bash
docker compose up -d
```
# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 7

Now that it is all up and running please run the following command to create your super user, this will be used to login to the webpage 

```bash
docker compose exec netbox /opt/netbox/netbox/manage.py createsuperuser
```

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Optional steps

you can check the passwords to ensure they have been updated by using the following commands
ensure you are in the env folder of the project directory and change the .env to the env file you want to check for instance netbox.env

```bash
grep -E 'SECRET_KEY|POSTGRES_PASSWORD|DB_PASSWORD|REDIS_PASSWORD' .env
```

ensure the ports are not exposed to other machines and only internally to the docker containers it should look like this 
5432/tcp and not like 
0.0.0.0:5432->5432/tcp 
to see this please run 

```bash
docker ps
```
