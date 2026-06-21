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

