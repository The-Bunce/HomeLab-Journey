#  Docker security audit
| Number | Docker Best Practice | How to  Comply | Tools Required | How I implemented | Commands to check
|----------|-------------|-------------|-------------|-------------|-------------|
| 1 | Keep Host and Docker Up to Date |Need to ensure the hosting system is up to date & is updated on a regularly schedule | built in tools such as: unattended-upgrades , n8n etc | I will be using n8n to run certain scripts such as updating (this is so it can message me when its finished)| sudo apt update (this will show you any packages that can be installed), you can get version information using the following information hostnamectl to get the operating system, apt list --upgradable will show you all packages that can be upgraded.
