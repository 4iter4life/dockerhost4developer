Setup prepared of single server and providing all necasserry tools for small development teams.
This setup was initialy prepared for own needs and containing following tooling:

- nginx-proxy + acme-companion (Reverse proxy with LetsEncrypt SSl certificate generation)
- portainer (Management interface for your Dockerhost and API for continous deployment from CI)
- nextcloud (File Exchange and Colobaration)
- bitwarden (Password Store)
- GitLab (CI/CD)
- ELK-Stack (for centralized Logging)

All this services will be presented in this repository as a example configuration, that allow to everyone create thair own simple Development Environment.

# Precondition
* public server (i prefere dedicated one, but u can use virtual too)
* installed docker engine
* clone this repository to a server and adapt all configurations to your needs
* Write Access to a DNS entries of your domain. 
* create /store directory on route for storing of all (make sense to use it as a mount to a dedicated disc to prevent the OS disc overflow) 

## DNS Entries
Prepare following DNS entries on your DNS Server (DNS Registrar)

* A     server1.exmple.com      <your-server-ip-address>
* CNAME portainer.example.com   server1.example.com
* CNAME nextcloud.example.com   server1.example.com
* CNAME vault.example.com       server1.example.com
* CNAME gitlab.example.com      server1.example.com
* CNAME registry.example.com    server1.example.com
* CNAME kibana.example.com      server1.example.com

# Installation
I will use example.com in my example configurations. You need to adapt it to your FQDN (Full Qualified Domain Name).

## Docker Network
First of all i prefere to use a dedicated docker network for all our deployments. This network will be used by acme-companion and nginx proxy to generate a certificates and create a nginx configurations.

```bash
docker network create server1 
```

## nginx + acme + portainer
On front of all our services we will use nginx with SSL Termination and to make a management easier we will use portainer. 
Lets install it.

```bash
cd .management
# don't forgett to adapt the configuration in initial.yaml
docker-compose -f initial.yaml up -d
```

Check the result under https://portainer.example.com

## nextcloud
Create new stack in portainer based on following docker compose file
```bash
# Dont forget to chande domain names and passwords
nextcloud.yaml
```

## bitwarden

## Gitlab

## ELK
