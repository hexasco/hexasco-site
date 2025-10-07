# Hexasco Setup Guide

Visit [my site](https://hexasco.org)

Complete setup guide for Hugo + Nginx + Cloudflare Tunnel deployment.

## Prerequisites

- Ubuntu/Debian server
- Domain name (hexasco.org) added to Cloudflare

## 1. Install Hugo

```bash
# use snaps for latest release
sudo snap isntall hugo
hugo version
```

## 2. Create Hugo Site

```bash
hugo new site [your-site-name]
cd [your-site-name]
git init
```

### Install Nightfall Theme

```bash
hugo mod init github.com/yourusername/your-site-name
hugo mod get github.com/lordmathis/hugo-theme-nightfall
```

## 3. Setup Nginx

### Install Nginx

```bash
sudo apt install nginx
```

### Create Site Config

```bash
sudo nano /etc/nginx/sites-available/[your-site]
```

Add configuration:

```nginx
server {
	listen 80;
	listen [::]:80;
	
	server_name [your-domain];
	
	root /var/www/[your-site]/public;
	index index.html;
	
	error_page 404 /404.html;
	
	location / {
		try_files $uri $uri/ =404;
	}
	
	location = /404.html {
		internal;
	}
}
```

### Enable Site

```bash
sudo ln -s /etc/nginx/sites-available/[your-site] /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
# check the nginx config for syntax errors before restarting the service
sudo nginx -t
sudo systemctl restart nginx
```

### Setup Web Directory

```bash
sudo mkdir -p /var/www/hexasco
sudo chown -R www-data:www-data /var/www/hexasco
```

## 4. Setup Cloudflare Tunnel

### Install Cloudflared

[Go to download and install](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/)

### Login to Cloudflare

```bash
cloudflared tunnel login
```

Select your domain when prompted.

### Create Tunnel

```bash
cloudflared tunnel create [tunnel-name]
```

Note the tunnel ID from the output.

### Configure Tunnel

Create `~/.cloudflared/config.yml`:

```yaml
tunnel: YOUR_TUNNEL_ID
credentials-file: ~/.cloudflared/YOUR_TUNNEL_ID.json

ingress:
  - hostname: [your-domain]
    service: http://localhost:80
  - service: http_status:404
```

Replace `YOUR_TUNNEL_ID` with your actual tunnel ID.

### Route DNS

```bash
cloudflared tunnel route dns [tunnel-name] [your-domain]
```

### Run Tunnel

```bash
cloudflare tunner run [runnel-name]
```


Done! Now you can visit your site at https://[your-domain]