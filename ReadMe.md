# 🧩 Mediastack — The Ultimate Self-Hosted ARR Stack

![Docker](https://img.shields.io/badge/Docker-Ready-blue?logo=docker)
![License](https://img.shields.io/github/license/InsafNilam/mediastack)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%2B-orange?logo=ubuntu)
![Made with ❤️](https://img.shields.io/badge/Made%20with-%E2%9D%A4-red)

> Self-hosted **media automation stack** featuring Radarr, Sonarr, Prowlarr, Jellyfin, qBittorrent, SABnzbd, and more  
> secured by **Gluetun VPN**, **WireGuard**, and **Nginx reverse proxy** with **mkcert SSL**.  
> Privacy-first. Modular. Fully automated.

---

## 🎬 Overview

**Mediastack** is a complete self-hosted **ARR ecosystem** built with Docker.  
It automates everything — from media downloading to organization and playback — in a secure, VPN-isolated environment.

---

## 🧠 Why Mediastack?

Unlike most ARR stacks, Mediastack focuses on:

- 🔒 **VPN-first architecture** — all torrent traffic goes through Gluetun.
- 🧩 **Modular micro-stack** — each service has isolated data/config.
- 🌐 **Local SSL** with mkcert for trusted HTTPS on `.local` domains.
- 🧱 **Reverse proxy** with domain-based routing.
- 🧰 **WireGuard + Pi-hole** integration for secure remote access.

---

## 🧭 Architecture Overview

```
 ┌────────────┐     ┌─────────────┐
 │   Client   │───▶│ Nginx Proxy  │──┐
 └────────────┘     └─────┬───────┘  │
                          ▼          │
                  ┌─────────────┐    │
                  │   ARR Apps  │    │
                  │(Radarr etc.)│◀───┤
                  └─────────────┘    │
                          │          │
                          ▼          │
                  ┌─────────────┐    │
                  │  Gluetun VPN│────┘
                  └─────────────┘
```

---

## 🧩 Included Services

| Service         | Role / Purpose           | Access URL                  |
| --------------- | ------------------------ | --------------------------- |
| **Sonarr**      | TV show automation       | `https://sonarr.local`      |
| **Radarr**      | Movie automation         | `https://radarr.local`      |
| **Lidarr**      | Music automation         | `https://lidarr.local`      |
| **Readarr**     | Book automation          | `https://readarr.local`     |
| **Mylar**       | Comic automation         | `https://mylar.local`       |
| **Bazarr**      | Subtitle manager         | `https://bazarr.local`      |
| **Prowlarr**    | Indexer manager          | `https://prowlarr.local`    |
| **qBittorrent** | Torrent client           | `https://qbittorrent.local` |
| **SABnzbd**     | Usenet downloader        | `https://sabnzbd.local`     |
| **Huntarr**     | Missing Content Manager  | `https://huntarr.local`     |
| **Jellyfin**    | Media server             | `https://jellyfin.local`    |
| **Jellyseerr**  | Media requests interface | `https://jellyseerr.local`  |
| **Heimdall**    | Dashboard / landing page | `https://media.local`       |
| **Gluetun**     | VPN gateway (WireGuard)  | Internal only               |

---

## ⚙️ Prerequisites

- Ubuntu 22.04+ VPS or machine
- Sudo access
- Internet connection
- Basic understanding of Docker & terminal commands

---

## 🐳 Step 1: Install Docker (Official Method)

> Follow the official Docker installation guide:
> 🔗 [Install Docker on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

### Uninstall Old Versions

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt-get remove -y $pkg
done
```

### Remove Docker Engine (if previously installed)

```bash
sudo apt-get purge -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
sudo rm -rf /var/lib/docker /var/lib/containerd
sudo rm /etc/apt/sources.list.d/docker.list
sudo rm /etc/apt/keyrings/docker.asc
```

### Add Docker GPG Key & Repo

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install Docker Engine

```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

## 🧹 Step 2: Clean Up Resources (Optional)

```bash
sudo docker container prune -f
sudo docker image prune -a -f
sudo docker volume prune -f
sudo docker network prune -f
```

---

## 📁 Step 3: Setup Folders

We’ll organize everything under `/opt/docker/mediastack`:

```bash
sudo mkdir -p /opt/docker/mediastack/{media,data}
```

---

## 👥 Step 4: Configure Users & Permissions

```bash
sudo useradd -MU docker
sudo useradd -Mg docker docker
# NOTE: Its OK if some of the above steps failed, you may already have a "docker" user or group

sudo groupadd mediastack
sudo usermod -aG mediastack docker
sudo usermod -aG mediastack <your_username>
```

---

## 🧰 Step 5: Environment Setup

```bash
export FOLDER_FOR_MEDIA=/opt/docker/mediastack/media
export FOLDER_FOR_DATA=/opt/docker/mediastack/data
export PUID=1001  # id docker
export PGID=1001  # grep mediastack /etc/group
```

### Create Application Folders

```bash
sudo -E mkdir -p $FOLDER_FOR_DATA/{bazarr,gluetun,heimdall,jellyfin,jellyseerr,lidarr,mylar,prowlarr,qbittorrent,radarr,readarr,sabnzbd,sonarr,unpackerr,whisparr,filebot,huntarr}
sudo -E mkdir -p $FOLDER_FOR_MEDIA/{media/{anime,audio,books,comics,movies,music,photos,tv,xxx},usenet/{anime,audio,books,comics,complete,incomplete,movies,music,tv},torrents/{anime,audio,books,comics,complete,incomplete,movies,music,tv},watch,filebot/{input,output}}
sudo chmod -R 775 $FOLDER_FOR_MEDIA $FOLDER_FOR_DATA
sudo chown -R $PUID:$PGID $FOLDER_FOR_MEDIA $FOLDER_FOR_DATA
```

---

## 🧱 Step 6: Pull and Deploy Containers

Download the Docker Compose and `.env` files from the repo:

🔗 [GitHub – InsafNilam/mediastack](https://github.com/InsafNilam/mediastack)

Then, in `/opt/docker/mediastack`:

```bash
git clone https://github.com/InsafNilam/mediastack.git

for file in *.yml; do
  echo "Pulling images from $file..."
  sudo docker compose --file "$file" --env-file .env pull
done

cp .env.example .env
```

> Then, configure the environment variables

### Start Containers in Order

> Gluetun must start first — it manages the VPN network.

```bash
sudo docker compose --file docker-compose.yml up -d gluetun
sudo docker compose --file docker-compose.yml up -d
```

If you update any configuration (e.g., API keys) or encounter issues with a service, simply recreate that service:

```bash
docker compose up -d --no-deps --build --force-recreate <service_name>
```

---

## 🔐 Step 7: Fix File Permissions

```bash
sudo chmod -R 2775 /opt/docker/mediastack
sudo chown -R docker:mediastack /opt/docker/mediastack
sudo find /opt/docker/mediastack -type d -exec chmod 2775 {} \;
sudo find /opt/docker/mediastack -type f -exec chmod 0664 {} \;
```

---

## 🌐 Step 8: Network & Access Configuration

By default, ports are bound to localhost for security:

```yaml
ports:
  - 127.0.0.1:8888:8888
```

To expose publicly, remove the `127.0.0.1:` prefix:

```yaml
ports:
  - 8888:8888
```

---

## 🧱 Step 9: Setup Nginx Reverse Proxy + SSL (mkcert)

> _(Not required if you’re exposing services via public IP)_

If you prefer to access your services securely through local domains (e.g. `sonarr.local`, `jellyfin.local`), you can set up **Nginx** as a reverse proxy and use **mkcert** to generate trusted local SSL certificates.

---

### 1️⃣ Copy Nginx Configuration

Copy your `mediastack.conf` file into the Nginx configuration directory:

```bash
sudo cp mediastack.conf /etc/nginx/sites-available/
```

---

### 2️⃣ Enable the Site

Create a symbolic link to enable the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/mediastack.conf /etc/nginx/sites-enabled/
```

---

### 3️⃣ Install mkcert for Local SSL

Install `mkcert` to generate locally trusted SSL certificates:

```bash
sudo apt update -y
sudo apt install -y libnss3-tools
wget https://dl.filippo.io/mkcert/latest?for=linux/amd64 -O mkcert
chmod +x mkcert
sudo mv mkcert /usr/bin/
mkcert -install
```

---

### 4️⃣ Create SSL Certificates

Generate certificates for all your local domains and store them under `/etc/nginx/ssl`:

```bash
sudo mkdir -p /etc/nginx/ssl
cd /etc/nginx/ssl

mkcert huntarr.local filebot.local unpackerr.local whisparr.local mylar.local sabnzbd.local qbittorrent.local bazarr.local prowlarr.local readarr.local lidarr.local sonarr.local radarr.local jellyseerr.local jellyfin.local media.local

mv media.local+15.pem mediastack.crt
mv media.local+15-key.pem mediastack.key
```

---

### 5️⃣ Update Your Nginx Configuration

SSL is already configured in your `.conf`, no changes are needed.
If you’re not using SSL and prefer plain HTTP, use the non-SSL version of the config instead.
However, be aware of the **drawbacks** below:

#### ⚠️ Why create local SSL certificates?

- When accessing your services by typing only the domain name (e.g. `https://media.local`), browsers often **force HTTPS**.
- Without valid SSL, browsers may redirect you to **Nginx’s default page** or another active site.
- Although `http://` works, modern browsers like Chrome and Firefox enforce HTTPS by default for `.local` or known hostnames.
- Generating **local certificates with mkcert** ensures your local domains load securely without warnings or unwanted redirects.

Add or confirm these lines in your Nginx config if you’re using SSL:

```nginx
    ssl_certificate /etc/nginx/ssl/mediastack.crt;
    ssl_certificate_key /etc/nginx/ssl/mediastack.key;
```

Finally, test and reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🧭 Step 10: DNS & VPN Integration

If using **Pi-hole + WireGuard**, add local DNS records like:

```
huntarr.local → 10.8.0.1
qbittorrent.local → 10.8.0.1
jellyfin.local → 10.8.0.1
...

```

The IP `10.8.0.1` corresponds to the WireGuard interface (`wg0`).
These services will be accessible only when connected via VPN — ensuring full privacy.

---

## 🧰 Optional: Troubleshooting

If a service is misconfigured:

```bash
docker compose stop sabnzbd
docker compose rm -f sabnzbd
docker compose up -d sabnzbd
```

To check outbound IP from within a container:

```bash
docker exec -it qbittorrent curl ifconfig.me
```

---

Here are a few improved, professional alternatives for that line — depending on the tone you want:

---

## 🧠 Pro Tips

- Prefer **WireGuard** for secure access → [Privacy & Security Series](https://insafnilam.hashnode.dev/series/self-hosted-privacy-security).
- For remote access via domain + HTTPS, use **Let’s Encrypt**:
  🔗 [Fortify Your Droplet with Free HTTPS (Let’s Encrypt)](https://insafnilam.hashnode.dev/fortify-your-droplet-unlock-free-https-with-lets-encrypt-ssl)
- Consider **Portainer** for managing Docker containers.

---

## 🏁 The End

Your ARR stack is now fully set up — VPN-protected, SSL-secured, and automation-ready.
You can access your apps at:

```
https://sonarr.local
https://radarr.local
https://jellyfin.local
...

```

---

### 📚 Credits

- [Docker Documentation](https://docs.docker.com/)
- [mkcert](https://github.com/FiloSottile/mkcert)
- [WireGuard](https://www.wireguard.com/)
- [Pi-hole](https://pi-hole.net/)
- [Mediastack by Insaf Nilam](https://github.com/InsafNilam/mediastack)
