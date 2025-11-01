# Raspberry-Pi-5-Artemis-Setup
How to setup Artemis On a raspberry Pi 5, includes basic how to setup a raspberry pi 5 from absolute beginner level


## Step 1
Get a raspberry Pi 5
Get a raspberry Pi 5 case
Get a micro sd card
Get a ssd harddrive
Get an ssd Harddrive Case
Get a raspberry Pi 5 USB-C charger
(Get a micro sd card reader if you dont already have a way to read the cart on the pc)

## Step 2
Place the micro SD card into the card reader and then into your pc.
Download Raspberry Pi Imager onto your PC
Run the imager and select Raspberry Pi 5 - In the OS section select Raspberry Pi OS (other) and then Raspberry Pi OS Lite (64-Bit) - in the last section select your micro sd cart location.

In the settings select custom, leave the hostname as is. Select the username as pi and choose a password thats strong but easy to remember. Leave the wireless LAN unchecked if ethernet is picked. In services enable SSH and leave the default password authentication. in options leave it default.

Save then remove the micro sd and place it into the raspberry Pi 5 (Its on the left below the actual Pi there should be a slot that perfectly fits a micro SD)

## Step 3
Open the raspberry Pi 5 case, then remove the hat on the fan port (the white one near the top right of the raspberry pi)

Place the fan charger into the port on the raspberry pi, then place the heat sink (it should be in one of the boxes) on top of the raspberry pi middle section it should align perfectly with it.

Next place the raspberry Pi 5 into the case and close the case. 


## Step 4
Next find a spot where you can have a ethernet cable to connect to the Pi, aswell as a socket for the charger. Connect the charger and the ethernet and the ssd case into the Raspbery Pi. If done correctly the fan should spin a few times and then stop. (The pi has a temperature system to determine when to spin the fan so its normal if it stops spinning.)



### Software part

Got it 👍 — here’s a **plain text version** (no Markdown, no formatting), just a straightforward step-by-step instruction sheet that you can copy into a `.txt` file for your Raspberry Pi 5 setup.

---

**Raspberry Pi 5 – Software and Cloudflare Setup**

This guide explains how to set up Docker, PostgreSQL, and n8n on a Raspberry Pi 5, and connect it securely through a Cloudflare Tunnel.

---

**1. Connect to the Raspberry Pi**

* Open a terminal or PowerShell.
* Type:
  ssh pi@<your-pi-ip>
* Default username: pi
  Default password: raspberry

If you don’t know the Pi’s IP, try:
ping raspberrypi.local

---

**2. Update the system**

Run:

```
sudo apt update && sudo apt upgrade -y
sudo reboot
```

Reconnect after reboot:

```
ssh pi@<your-pi-ip>
```

---

**3. Install Docker and Docker Compose**

Install Docker:

```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

Allow the pi user to run Docker without sudo:

```
sudo usermod -aG docker $USER
newgrp docker
```

Install Docker Compose:

```
sudo apt install docker-compose-plugin -y
```

Check installation:

```
docker --version
docker compose version
```

---

**4. Create the n8n folder**

```
mkdir ~/n8n
cd ~/n8n
```

---

**5. Create the .env file**

```
nano .env
```

Paste the following:

```
POSTGRES_USER=n8n
POSTGRES_PASSWORD=n8n
POSTGRES_DB=n8n
POSTGRES_NON_ROOT_USER=n8n
POSTGRES_NON_ROOT_PASSWORD=n8n

DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=postgres
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=n8n

GENERIC_TIMEZONE=Europe/Copenhagen
N8N_ENCRYPTION_KEY=replace_this_with_a_random_string
WEBHOOK_TUNNEL_URL=https://your-tunnel-name.trycloudflare.com
```

Generate a random encryption key:

```
openssl rand -hex 32
```

Copy the key and replace the placeholder value.

Save and exit (Ctrl + O, Enter, Ctrl + X).

---

**6. Create the docker-compose.yml file**

```
nano docker-compose.yml
```

Paste this:

```
version: '3.8'

services:
  postgres:
    image: postgres:16
    restart: always
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - ./postgres_data:/var/lib/postgresql/data

  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - DB_TYPE=${DB_TYPE}
      - DB_POSTGRESDB_HOST=${DB_POSTGRESDB_HOST}
      - DB_POSTGRESDB_PORT=${DB_POSTGRESDB_PORT}
      - DB_POSTGRESDB_DATABASE=${DB_POSTGRESDB_DATABASE}
      - DB_POSTGRESDB_USER=${DB_POSTGRESDB_USER}
      - DB_POSTGRESDB_PASSWORD=${DB_POSTGRESDB_PASSWORD}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - GENERIC_TIMEZONE=${GENERIC_TIMEZONE}
      - WEBHOOK_TUNNEL_URL=${WEBHOOK_TUNNEL_URL}
    depends_on:
      - postgres
    volumes:
      - ./n8n_data:/home/node/.n8n

  cloudflared:
    image: cloudflare/cloudflared:latest
    command: tunnel --no-autoupdate run --token YOUR_CLOUDFLARE_TUNNEL_TOKEN
    restart: always
```

Replace YOUR_CLOUDFLARE_TUNNEL_TOKEN with the actual token from your Cloudflare dashboard.

Save and exit (Ctrl + O, Enter, Ctrl + X).

---

**7. Start everything**

Run:

```
docker compose pull
docker compose up -d
```

Check containers:

```
docker ps
```

You should see postgres, n8n, and cloudflared running.

---

**8. Access n8n**

Local:

```
http://<your-pi-ip>:5678
```

Through Cloudflare:

```
https://your-tunnel-name.trycloudflare.com
```

---

**9. Useful Docker commands**

Stop:

```
docker compose down
```

Start:

```
docker compose up -d
```

Logs:

```
docker compose logs -f n8n
```

Update:

```
docker compose pull n8n
docker compose up -d
```

---

**10. Notes**

* Default n8n credentials are created the first time you access it.
* Data is stored in:
  ~/n8n/postgres_data
  ~/n8n/n8n_data
* Cloudflare automatically removes a leading slash in access paths.
  Example: use `webhook/*` instead of `/webhook/*`.

---

**Done!**

Your Raspberry Pi 5 is now running n8n with PostgreSQL and Cloudflare.
You can start building automations and connecting your services right away.
