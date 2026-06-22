#  Docker security audit
| Number | Docker Best Practice | How to  Comply | Tools Required | How I implemented | Commands to check
|----------|-------------|-------------|-------------|-------------|-------------|
| 1 | Keep Host and Docker Up to Date |Need to ensure the hosting system is up to date & is updated on a regularly schedule | built in tools such as: unattended-upgrades , n8n etc | I will be using n8n to run certain scripts such as updating (this is so it can message me when its finished)| sudo apt update (this will show you any packages that can be installed), you can get version information using the following information hostnamectl to get the operating system, apt list --upgradable will show you all packages that can be upgraded.
| 2 |Do Not Expose the Docker Daemon Socket|-------------|-------------|-------------|-------------|
| 3 |Run Docker in Rootless Mode|-------------|-------------|-------------|-------------|
| 4 |Avoid Privileged Containers|-------------|-------------|-------------|-------------|
| 5 |Limit Container Resources|-------------|-------------|-------------|-------------|
| 6 |Segregate Container Networks|-------------|-------------|-------------|-------------|
| 7 |Improve Container Isolation|-------------|-------------|-------------|-------------|
| 8 |Set Filesystem and Volumes to Read-only|-------------|-------------|-------------|-------------|
| 9 |Complete Lifecycle Management|-------------|-------------|-------------|-------------|
| 10 |Restrict System Calls from Within Containers|-------------|-------------|-------------|-------------|
| 11 |Scan and Verify Container Images|-------------|-------------|-------------|-------------|
| 12 |Use Minimal Base Images|-------------|-------------|-------------|-------------|
| 13 |Don’t Leak Sensitive Info to Docker Images|-------------|-------------|-------------|-------------|
| 14 |Use Multi Stage Builds|-------------|-------------|-------------|-------------|
| 15 |Secure Container Registries|-------------|-------------|-------------|-------------|
| 16 |Use Fixed Tags for Immutability|-------------|-------------|-------------|-------------|
| 17 |Add the HEALTHCHECK Instruction to the Container Image|-------------|-------------|-------------|-------------|
| 18 |Use COPY Instead of ADD When Writing Dockerfiles|-------------|-------------|-------------|-------------|
| 19 |Monitor Container Activity|-------------|-------------|-------------|-------------|
| 20 |Secure Containers at Runtime|-------------|-------------|-------------|-------------|
| 21 |Save Troubleshooting Data Separately from Containers|-------------|-------------|-------------|-------------|
| 22 |Use Metadata Labels for Images|-------------|-------------|-------------|-------------|
| 23 |Host Configuration|-------------|-------------|-------------|-------------|
| 24 |Docker Daemon Configuration|-------------|-------------|-------------|-------------|
| 25 |Container Images and Build File|-------------|-------------|-------------|-------------|
| 26 |Container Runtime|-------------|-------------|-------------|-------------|
| 27 |Docker Security Operations|-------------|-------------|-------------|-------------|
| 28 |Docker Swarm Configuration|-------------|-------------|-------------|-------------|

# <img src="/Images/Docker-Images/Audit.png" width="25" height="25" /> Audit of my own systems against the above standards (before i fixed them to be more secure)

| Number | Docker Best Practice | Compliant | Explaination | How I remediated |
|----------|-------------|-------------|-------------|-------------|
| 1 |Keep Host and Docker Up to Date|<img src="/Images/Docker-Images/Not Compliant.png" width="25" height="25" />|-------------|-------------|
| 2 |Do Not Expose the Docker Daemon Socket|<img src="/Images/Docker-Images/yes.png" width="25" height="25" />|-------------|-------------|
| 3 |Run Docker in Rootless Mode|-------------|-------------|-------------|
| 4 |Avoid Privileged Containers|-------------|-------------|-------------|
| 5 |Limit Container Resources|-------------|-------------|-------------|
| 6 |Segregate Container Networks|-------------|-------------|-------------|
| 7 |Improve Container Isolation|-------------|-------------|-------------|
| 8 |Set Filesystem and Volumes to Read-only|-------------|-------------|-------------|
| 9 |Complete Lifecycle Management|-------------|-------------|-------------|
| 10 |Restrict System Calls from Within Containers|-------------|-------------|-------------|
| 11 |Scan and Verify Container Images|-------------|-------------|-------------|
| 12 |Use Minimal Base Images|-------------|-------------|-------------|
| 13 |Don’t Leak Sensitive Info to Docker Images|-------------|-------------|-------------|
| 14 |Use Multi Stage Builds|-------------|-------------|-------------|
| 15 |Secure Container Registries|-------------|-------------|-------------|
| 16 |Use Fixed Tags for Immutability|-------------|-------------|-------------|
| 17 |Add the HEALTHCHECK Instruction to the Container Image|-------------|-------------|-------------|
| 18 |Use COPY Instead of ADD When Writing Dockerfiles|-------------|-------------|-------------|
| 19 |Monitor Container Activity|-------------|-------------|-------------|
| 20 |Secure Containers at Runtime|-------------|-------------|-------------|
| 21 |Save Troubleshooting Data Separately from Containers|-------------|-------------|-------------|
| 22 |Use Metadata Labels for Images|-------------|-------------|-------------|
| 23 |Host Configuration|-------------|-------------|-------------|
| 24 |Docker Daemon Configuration|-------------|-------------|-------------|
| 25 |Container Images and Build File|-------------|-------------|-------------|
| 26 |Container Runtime|-------------|-------------|-------------|
| 27 |Docker Security Operations|-------------|-------------|-------------|
| 28 |Docker Swarm Configuration|-------------|-------------|-------------|