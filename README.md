# Overview

This project is a publicised version of my personal homelab setup. It **will not** work out of the box, and is largely intended as a starting point. The necessary files and/or variables have been redacted. It is up to the user to fill in the necessary information to get the project working.

It uses traefik as a reverse proxy to route traffic to different services running on the server. The traefik container offers a DNS challenge to automatically generate SSL certificates for the services. The services are deployed using docker-compose and are managed using ansible. The services are deployed to a single server running Ubuntu 20.04.

| Service | Description |
| --- | --- |
| Heimdall | Home dashboard for all services. I don't currently use this but the compose file is included for completeness |
| Traefik | Reverse proxy |
| Jellyfin | Media server |
| Pi-hole | Network wide ad blocker and local DNS server |
| Grafana | Dashboard for monitoring server metrics |
| InfluxDB | Time series database for storing server metrics |
| Promtail | Log collector for sending logs to Loki |
| Loki | Log aggregation system |
| Prometheus | Monitoring system |
| *arr* stack | torrent indexers and client for downloading linux isos |

To use this project you will need to have a domain name and a cloudflare account. Ensure the target server has a static IP address and SSH access is enabled. Running `ansible-playbook -i hosts.yml {production|development}.yml` will run the infrastructure as code to deploy the services to the server. *Note: you will need to have configured your own hosts file and SSH keys to run the playbook.*



# Traefik Configuration

## Port Forwarding

In order for all of this to work we need to forward the ports so that cloudflare can communicate with the traefik container. To do this we need to forward the following ports:

- 80
- 443

Go to the router settings and forward the ports to the IP address of the server.

## Docker Network

navigate to the `traefik/data` directory and run the following command:

```bash
sudo docker network create traefik
``` 

This will create a new network called `traefik` which will be used by the traefik container to communicate with other containers.

we can verify that the network has been created by running the following command:

```bash
sudo docker network ls
```

This will list all the networks on the system and we should see the `traefik` network in the list.

## Basic Pass Authentication

- install apache2-utils, if not already installed:

```bash
sudo apt update && sudo apt install apache2-utils
```

- Run the following command, see traefik docs here for more information: https://doc.traefik.io/traefik/middlewares/http/basicauth/

```bash
echo $(htpasswd -nB user) | sed -e s/\\$/\\$\\$/g
```

- Copy the output and replace the `auth` field in the traefik `docker-compose.yml` file with the output.
- When you visit the traefik dashboard you will be prompted to enter the username and password. Use the username and password you created in the previous step (not the output of the command).
