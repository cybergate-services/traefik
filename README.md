# Traefik with Docker and Let's Encrypt

As you are aware, we run most of our services in Docker and currently we are using nginx as a reverse and TLS termination proxy together with Let's Encrypt. That worked great but everytime when we wanted to try something new we had to copy-paste another conf and change a few values. This probably could have been automated that to some extent and there are others who have but I recently figured I'd give Traefik a try.

Here I'll show you how to set up Traefik with GUI, http redirection and automatic Let's Encrypt certificates. We'll also add basic auth to the Traefik GUI.

# Prerequisites

This guide assumes some general knowledge of Linux and that you have a server available with these services installed:

_ docker
_ docker-compose
_ A domain to host your apps on.For cheap and good servers, checkout DigitalOcean or Hetzner

# Setup

This is based on the official guide but with a few additions. I will show you how to add the web dashboard and API - protected by Basic Auth - mostly because it's fun. If you have no use for it or believe it to be unsafe, you can skip that part.

First start with creating a network for your web-facing containers to connect to.

docker network create web
Then we create a directory and the necessary files, as sudo if needed.

sudo su
mkdir -p /opt/traefik
touch /opt/traefik/docker-compose.yml
touch /opt/traefik/acme.json && chmod 600 /opt/traefik/acme.json
touch /opt/traefik/traefik.toml
