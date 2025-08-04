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


To host Wazuh Docker single-node at https://qadar.space with a custom SSL certificate via Let’s Encrypt and use NGINX as a reverse proxy, based on Wazuh’s official documentation, follow the complete step-by-step guide below.

⸻

🛠️ Prerequisites

✅ A domain: qadar.space
✅ A record pointing to your EC2 public IP: 63.35.217.170
✅ Ports 80 and 443 open in your EC2 Security Group
✅ Docker and Docker Compose installed

⸻

📦 STEP 1: Clone Wazuh Docker Repository

git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.3
cd wazuh-docker/single-node

🌐 STEP 2: Set Up SSL Certificate with Let’s Encrypt (Using Certbot)

Use standalone Certbot with Docker:

sudo docker run -it --rm \
  -p 80:80 \
  -v "/etc/letsencrypt:/etc/letsencrypt" \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  certbot/certbot certonly \
  --standalone \
  --agree-tos \
  --no-eff-email \
  --email your-email@example.com \
  -d qadar.space


You will now have your certs at:


📝 STEP 3: Configure NGINX Proxy for SSL

🔸 3.1 Install NGINX (On EC2 Host):

sudo apt update
sudo apt install nginx -y

🔸 3.2 Create NGINX Config File

sudo nano /etc/nginx/sites-available/wazuh




Paste this config:

server {
    listen 80;
    server_name qadar.space;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name qadar.space;

    ssl_certificate /etc/letsencrypt/live/qadar.space/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/qadar.space/privkey.pem;

    location / {
        proxy_pass https://127.0.0.1:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}



🔸 3.3 Enable Config


sudo ln -s /etc/nginx/sites-available/wazuh /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx


🐳 STEP 4: Configure Docker Compose to Bind Dashboard on Localhost

🔸 Edit docker-compose.yml in wazuh-docker/single-node/:

Ensure Wazuh Dashboard exposes port 5601 only to localhost:

dashboard:
  ports:
    - "127.0.0.1:5601:5601"



▶️ STEP 5: Launch Wazuh Single-Node

From inside wazuh-docker/single-node directory:

docker-compose -f generate-indexer-certs.yml run --rm generator
docker-compose up -d


✅ STEP 6: Access Wazuh Dashboard

Visit:

👉 https://qadar.space


## Renewal (Every 90 Days)

You’ll need to stop your nginx container and renew manually like so:


sudo docker stop nginx-ssl
sudo docker run -it --rm \
  -p 80:80 \
  -v "/etc/letsencrypt:/etc/letsencrypt" \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  certbot/certbot renew
sudo docker start nginx-ssl

