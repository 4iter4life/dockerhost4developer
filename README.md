# Introduction
Setup prepared for single server and providing of usefull tools for small development teams.
This setup was initialy prepared for own needs and contains following tooling:

- nginx-proxy + acme-companion (Reverse proxy with LetsEncrypt SSl certificate generation)
- portainer (Management interface for your Dockerhost and API for continous deployment from CI)
- nextcloud (File Exchange and Colobaration)
- bitwarden (Password Store)
- GitLab (CI/CD)

All this services will be presented in this repository as a example configuration, that allow to everyone create thair own simple Development Environment.

This repository should collect usefull dockerized tools (in form of docker compose files) for self hosted environments.

# Precondition
* public server (i prefere dedicated one, but u can use virtual too)
* installed docker engine
* clone this repository to a server and adapt all configurations to your needs
* Writepermissions to a DNS entries of your domain. 
* create '/store' directory in root for storing of all (make sense to use it as a mount to a dedicated disc to prevent the OS disc overflow) 

## DNS Entries
Prepare following DNS entries on your DNS Server (DNS Registrar)

Type | Name | Target
-----|------|-------
A | server1.exmple.com | {your-server-ip-address}
CNAME | portainer.example.com | server1.example.com
CNAME | nextcloud.example.com | server1.example.com
CNAME | vault.example.com | server1.example.com
CNAME | gitlab.example.com | server1.example.com
CNAME | registry.example.com | server1.example.com

# Installation
I will use example.com in my example configurations. You need to adapt it to your FQDN (Full Qualified Domain Name).

## Docker Network
First of all i prefere to use a dedicated docker network for all our deployments. This network will be used by acme-companion and nginx proxy to generate a certificates and create a nginx configurations.

```bash
docker network create server1 
```

## nginx-proxy + acme 
On front of all our services we will use nginx with SSL termination and LetsEncrypt certificates. 

### Links
https://github.com/nginx-proxy/nginx-proxy  
https://github.com/nginx-proxy/acme-companion  

### Installation

```bash
# don't forgett to adapt the configuration in nginx.yaml
docker-compose -f nginx.yaml up -d
```

## Portainer CE
Portainer CE is a management UI for Docker.  
Sure, u can use all docker-compose files in this repository without portainer too, but sometimes its usefull to give access to your teammembers and restrict it to some deployments and not to the whole server as example. Portainer CE can help with it.
Additionaly we use it do deploy our services via CI/CD (see. Gitlab Pipeline + Portainer deploy (comming soon))

### Links
https://github.com/portainer/portainer  
### Installation
```bash
# don't forgett to adapt the configuration in initial.yaml
docker-compose -f portainer.yaml up -d
```

Check the result under https://portainer.example.com

## NextCloud
NextCloud is more then FileSharing. You can extend it with multiple plugins, that can help you for coloborative work. 

### Links
https://github.com/nextcloud/

### Installation
* Create new stack in portainer
* Customize and use **'nextcloud.yaml'** for your needs

### Additional configuration
Default 'client_max_body_size' ist 250MB. If you need to handle bigger files, you need to create additional vhost file for your domain (as example '/store/proxy/vhost.d/nextcloud.example.com') with following content
```ini
server_tokens off;
client_max_body_size 1000m;
```
After edit u need to restart nginx container
```bash
docker restart dockercompose_nginx-proxy_1
```

## Bitwarden
Bitwarden allows u to safe all project password securely and access it from all your devices

### Links
https://github.com/dani-garcia/vaultwarden

### Installation
* Create new stack in portainer
* Customize and use **'bitwarden.yaml'** for your needs

## Gitlab CE
Gitlab CE is a popular and powerfull CI/CD tool that not need a presentation.

We will deploy gitlab-ce and two buildworker (docker and dind).

### Links
https://hub.docker.com/r/gitlab/gitlab-ce

### Installation
* Create new stack in portainer
* Customize and use **'gitlab.yaml'** for your needs

### Notes
**Gitlab docker registry behind nginx proxy**  
For reason that nginx-proxy cannot handle multiple domains ('VIRTUAL_HOST') with different target ports ('VIRTUAL_PORT'), simple workaround for certificate creation and valid config configuration should be provided (see gitlab.yaml) 
```yaml
  port-forward:
    image: marcnuri/port-forward
    environment:
      LOCAL_PORT: 5005
      REMOTE_HOST: gitlab
      REMOTE_PORT: 5005
      VIRTUAL_HOST: registry.example.com
      VIRTUAL_PORT: 5005
      LETSENCRYPT_HOST: registry.example.com
    stdin_open: true
    tty: true
    networks:
    - server1
    restart: unless-stopped
```
This simple port forward open the port 5005 (docker registry) and register it for hostname registry.example.com and forward it to service 'gitlab'. 

**Gitlab worker registration**  
You need connect to the shell of your gitlab worker container first. Use for it portainer or directly over 'docker exec -it {containername} /bin/bash'.

* docker worker
```bash
gitlab-runner register -n --url https://gitlab.example.com/ --registration-token {your gitlab token} --name dind --tag-list "docker" --executor docker  --docker-image "docker:stable"
```
* dind (docker in docker) - to use docker engine in your builds
```bash
gitlab-runner register -n --url https://gitlab.example.com/ --registration-token {your gitlab token} --name dind --tag-list "dind" --executor docker  --docker-image "docker:stable" --docker-volumes /var/run/docker.sock:/var/run/docker.sock --docker-privileged
```

