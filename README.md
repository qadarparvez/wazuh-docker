# Wazuh containers for Docker

[![Slack](https://img.shields.io/badge/slack-join-blue.svg)](https://wazuh.com/community/join-us-on-slack/)
[![Email](https://img.shields.io/badge/email-join-blue.svg)](https://groups.google.com/forum/#!forum/wazuh)

## Description

The `wazuh/wazuh-docker` repository provides resources to deploy the Wazuh cybersecurity platform using Docker containers. This setup enables easy installation and orchestration of the full Wazuh stack, including the Wazuh server, dashboard (based on OpenSearch Dashboards), and OpenSearch for indexing and search.

## Capabilities

- Full deployment of the Wazuh stack using Docker.
- `docker compose` support for orchestration.
- Scalable architecture with multi-node support.
- Data persistence through configurable volumes.
- Ready-to-use configurations for production or testing environments.

## Branch Convention

- `main`: Developing and testing of new features.
- `X.Y.Z`: Version-specific branches (e.g., `5.0.0`, `4.14.0`, etc.).

## Documentation

Official documentation is available at:

[https://documentation.wazuh.com/current/deployment-options/docker/index.html](https://documentation.wazuh.com/current/deployment-options/docker/index.html)

You can also explore internal documentation in the [`docs`](https://github.com/wazuh/wazuh-docker/tree/main/docs) folder of this repository.

## Get Involved

- **Fork the repository** and create your own branches to add features or fix bugs.
- **Open issues** to report bugs or request features.
- **Submit pull requests** following the contributing guidelines.
- Participate in [discussions](https://github.com/wazuh/wazuh-docker/discussions) if available.

## Authors / Maintainers

These Docker containers are based on:

*  "deviantony" dockerfiles which can be found at [https://github.com/deviantony/docker-elk](https://github.com/deviantony/docker-elk)
*  "xetus-oss" dockerfiles, which can be found at [https://github.com/xetus-oss/docker-ossec-server](https://github.com/xetus-oss/docker-ossec-server)

This project is maintained by the [Wazuh](https://wazuh.com) team, with active contributions from the community.

See the full list of contributors at:
[https://github.com/wazuh/wazuh-docker/graphs/contributors](https://github.com/wazuh/wazuh-docker/graphs/contributors)

We thank them and everyone else who has contributed to this project.

## License and copyright

Wazuh Docker Copyright (C) 2017, Wazuh Inc. (License GPLv2)

## Web references

[Wazuh website](http://wazuh.com)



################################################################## Note from Qadar ###########################################


I have to deploy Wazuh-docker, single node on my sub domain

https://wazuh.qadarparvez.space


## Generate SSL certificaate
sudo docker run -it --rm \
  -p 80:80 \
  --name certbot \
  -v "/etc/letsencrypt:/etc/letsencrypt" \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  certbot/certbot certonly --standalone \
  --agree-tos \
  --no-eff-email \
  --email qadarparvez@gmail.com \
  -d qadarparvez.space


### Check SSL Cert Location
After a successful run, your certs will be stored in:

/etc/letsencrypt/live/yourdomain.com/

Files:
	•	fullchain.pem — your certificate
	•	privkey.pem — private key

## Run Nginx as Reverse Proxy in Docker with SSL

Let’s set up a reverse proxy container to forward traffic from HTTPS (443) to your Wazuh dashboard container (usually port 5601 internally).

First, create a folder and Nginx config file:


mkdir -p ~/nginx-wazuh/conf.d
nano ~/nginx-wazuh/conf.d/default.conf

Paste the following (edit with your domain and Wazuh container IP):

server {
    listen 443 ssl;
    server_name qadarparvez.space;

    ssl_certificate /etc/letsencrypt/live/qadarparvez.space/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/qadarparvez.space/privkey.pem;

    location / {
        proxy_pass http://wazuh-dashboard>:5601;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

server {
    listen 80;
    server_name qqadarparvez.space;
    return 301 https://$host$request_uri;
}


## Now run the Nginx container:

sudo docker run -d \
  --name nginx-ssl \
   -p 443:443 \
  -v "/etc/letsencrypt:/etc/letsencrypt:ro" \
  -v "$HOME/nginx-wazuh/conf.d:/etc/nginx/conf.d:ro" \
  nginx

## Renewal (Every 90 Days)

You’ll need to stop your nginx container and renew manually like so:


sudo docker stop nginx-ssl
sudo docker run -it --rm \
  -p 80:80 \
  -v "/etc/letsencrypt:/etc/letsencrypt" \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  certbot/certbot renew
sudo docker start nginx-ssl

