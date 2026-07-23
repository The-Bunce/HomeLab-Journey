#  Scanning Of Github Repos
The issue I have identified in my homelab is that i require a way to scan SBOMs and Github repositories to ensure they are secure before i pull from them, to do this i have found a tool called trivy.
# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step By Step Guide

Here is a step by step breakdown on how to get trivy up and running and some useful commands

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Information

First step will be to ensure you have a virtual machine either running linux or windows as this is what we will be installing it on.
please see the guide from their website:
https://trivy.dev/docs/latest/getting-started/

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 1 Installing Docker

```bash
sudo apt update && sudo apt upgrade -y
```

now we install docker 
follow the previous guide to install this or just go to the docker website 
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
to check to see if its running please run the below command
```bash
sudo systemctl status docker
```

# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 2 installing trivy

```bash
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sudo sh -s -- -b /usr/local/bin v0.72.0
```

```bash
sudo apt-get install wget gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```


# <img src="/Images/Docker-Images/Step-by-Step.png" width="25" height="25" /> Step 3 Scanning a Github Repo

to scan a repo simply put 
```bash
trivy repo "put repo here"
```
this will then scan it and is the only thing at this point of time i will be using trivy for, i do have plans to generate SBOMS of my linux systems and try to automate scanning with CI/CD
