# Traefik with Docker and Let's Encrypt

As you are aware, we run most of our services in Docker and currently we are using nginx as a reverse and TLS termination proxy together with Let's Encrypt. That worked great but everytime when we wanted to try something new we had to copy-paste another conf and change a few values. This probably could have been automated that to some extent and there are others who have but I recently figured I'd give Traefik a try.

Here I'll show you how to set up Traefik with GUI, http redirection and automatic Let's Encrypt certificates. We'll also add basic auth to the Traefik GUI.

# Prerequisites

This guide assumes some general knowledge of Linux and that you have a server available with these services installed:

* docker
* docker-compose
* A domain to host your apps on.For cheap and good servers, checkout DigitalOcean or Hetzne.

You You can use following guide to install `docker` on your `CentOS/Ubuntu/Fedora` systems

[How To Install Docker on Ubuntu 18.04/19.04/16.04](https://computingforgeeks.com/how-to-install-docker-on-ubuntu/)

You can use following guide to install `docker-comose` on your `CentOS/Ubuntu/Fedora` systems. 

[Install Latest Docker Compose on Ubuntu 18.04 / CentOS 8 / Debian 10 / Fedora 31](https://computingforgeeks.com/how-to-install-latest-docker-compose-on-linux/)

# Setup

This is based on the official guide but with a few additions. I will show you how to add the web dashboard and API - protected by Basic Auth - mostly because it's fun and secure. If you have no use for it or believe it to be unsafe, you can skip that part.

* First start with creating a network for your web-facing containers to connect to.
```
docker network create web
````

* Then we create a directory and the necessary files, as sudo if needed.
```
sudo su -
mkdir -p /opt/traefik
touch /opt/traefik/docker-compose.yml
touch /opt/traefik/acme.json && chmod 600 /opt/traefik/acme.json
touch /opt/traefik/traefik.toml
```
* Now let's create our `docker-compose.yml` file.

```
nano /opt/traefik/docker-compose.yml
```

Add the following contents.

```
version: '2'

services:
  proxy:
    image: traefik:v1.7.12-alpine
    command: --configFile=/traefik.toml
    restart: unless-stopped
    networks:
      - web
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /opt/traefik/traefik.toml:/traefik.toml
      - /opt/traefik/acme.json:/acme.json
    # REMOVE this section if you don't want the dashboard/API
    labels:
      - "traefik.enable=true"
      - "traefik.frontend.rule=Host:teamsproxy.cybergate.lk"
      - "traefik.port=8080"

networks:
  web:
    external: 
```
:pencil: Remember to replace`teamsproxy.cybergate.lk` in `traefik.frontend.rule` if you keep the API.

