# Traefik 3.6 on Raspberry Pi
> Production Setup Guide

---

## Overview

Production deployment of Traefik v3.6 as a reverse proxy on a Raspberry Pi, with automatic TLS via Let's Encrypt, a secured dashboard, and Docker-based service discovery.

---

## Prerequisites

- Raspberry Pi running Raspberry Pi OS (Bookworm recommended)
- Docker and Docker Compose installed
- A public domain name pointing to your Pi's IP
- Ports `80` and `443` open on your router (port forwarding)
- SSH access to your Pi

---

## Project Structure

```
traefik/
├── docker-compose.yaml
├── .env
├── letsencrypt/
│   └── acme.json          # TLS certificates (auto-generated)
└── logs/
    ├── traefik.log
    └── access.log
```

---

## Initial Setup

### 1. Create the Docker network

All services that Traefik will route to must share this network:

```bash
docker network create traefik_network
```

### 2. Create directories and fix permissions

The `acme.json` file must have strict permissions or Let's Encrypt will refuse to use it:

```bash
mkdir -p letsencrypt logs
touch letsencrypt/acme.json
chmod 600 letsencrypt/acme.json
```

### 3. Configure the .env file

Create a `.env` file in the same directory as `docker-compose.yml`:

```env
DOMAIN=mydomain.com
ACME_EMAIL=you@email.com

# Generate with the command below
BASIC_AUTH_USERS=admin:$$2y$$10$$...
```

### 4. Generate Basic Auth credentials

```bash
sudo apt install apache2-utils -y
echo $(htpasswd -nb admin yourpassword) | sed -e s/\\$/\\$\\$/g
```

Copy the output into `BASIC_AUTH_USERS` in your `.env` file.

### 5. Start Traefik

```bash
docker compose up -d
docker compose logs -f   # watch startup logs
```