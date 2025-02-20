# Self‑Host n8n for Free (Free SSL on AWS/GCP)  
*Your Step‑by‑Step Guide to Unlimited Automation*  


**By Kunal Barve | [CodeNAutomate](https://youtube.com/@CodeNAutomate) 🔥**  

---

## Why Follow This Guide?  
I used this exact method to deploy n8n for several client projects. It works and it’s completely free when using the cloud providers’ free tiers.  
- Zero monthly costs (free tier friendly)  
- Full control over your automation stack  
- SSL included (critical for secure workflows)  

**_For custom automation solutions, email: KunalBarve@articflow.io_**

---

## YouTube Video Explanation  
Watch the video explanation: [Self‑Host n8n 100% Free](https://www.youtube.com/watch?v=Temh_Ddxp24)  
![Thumbnail_v1](https://github.com/user-attachments/assets/d7874457-c2a6-4143-be70-818c8829c6b4)

---

## Step 1: Installing Docker

```bash
# Update and install Docker (works on Ubuntu/Debian)
sudo apt update && sudo apt install docker.io -y

# Start Docker and enable it to start at boot
sudo systemctl start docker && sudo systemctl enable docker
```

---

## Step 2: Starting n8n in Docker

Replace **`your-domain.com`** with your actual domain name (or **instance public IP address**) and run the following command to start n8n in a detached Docker container:

```bash
sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="your-domain.com" \
-e WEBHOOK_TUNNEL_URL="https://your-domain.com/" \
-e WEBHOOK_URL="https://your-domain.com/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n
```

If you’re using a subdomain, use:

```bash
sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="subdomain.your-domain.com" \
-e WEBHOOK_TUNNEL_URL="https://subdomain.your-domain.com/" \
-e WEBHOOK_URL="https://subdomain.your-domain.com/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n
```

*This command downloads and runs the n8n Docker image, exposes it on port 5678, sets the necessary environment variables, and mounts the local data directory for persistent storage.*

---

## Step 3: Installing Nginx

Nginx is used as a reverse proxy to forward requests to n8n and handle SSL termination.

```bash
# Install Nginx
sudo apt install nginx -y
```

---

## Step 4: Configuring Nginx

1. **Create a New Nginx Configuration File:**

   ```bash
   sudo nano /etc/nginx/sites-available/n8n
   ```

2. **Paste the Following Configuration:**

   ```bash
   server {
       listen 80;
       server_name your-domain.com;  # Use subdomain.your-domain.com if using a subdomain

       location / {
           proxy_pass http://localhost:5678;
           proxy_http_version 1.1;
           chunked_transfer_encoding off;
           proxy_buffering off;
           proxy_cache off;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
       }
   }
   ```

   Replace `your-domain.com` with your actual domain.

3. **Enable the Configuration:**

   ```bash
   sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
   ```

   *(If `/etc/nginx/sites-enabled/` doesn’t exist, create it with: `sudo mkdir /etc/nginx/sites-enabled/`)*

4. **Test and Restart Nginx:**

   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

---

## Step 5: Setting up SSL with Certbot

Certbot obtains and installs an SSL certificate from Let's Encrypt.

1. **Install Certbot and the Nginx Plugin:**

   ```bash
   sudo apt install certbot python3-certbot-nginx
   ```

2. **Obtain an SSL Certificate:**

   ```bash
   sudo certbot --nginx -d your-domain.com
   # For a subdomain, use: sudo certbot --nginx -d subdomain.your-domain.com
   ```



---

## Updating n8n

Before updating, always back up your n8n data.

### Using the Docker CLI

1. **Backup your data:**

   ```bash
   cp -r ~/.n8n ~/.n8n_backup
   ```

2. **Pull the desired n8n image:**
   - Latest (stable) version:
     ```bash
     docker pull docker.n8n.io/n8nio/n8n
     ```
   - Specific version (e.g., 0.220.1):
     ```bash
     docker pull docker.n8n.io/n8nio/n8n:0.220.1
     ```
   - Next (unstable) version:
     ```bash
     docker pull docker.n8n.io/n8nio/n8n:next
     ```

3. **Stop and remove the old container:**

   ```bash
   docker stop n8n && docker rm n8n
   ```

4. **Recreate and start the container using your original run command:**

   ```bash
   docker run -d --name n8n -p 5678:5678 -v ~/.n8n:/root/.n8n docker.n8n.io/n8nio/n8n
   ```

### Using Docker Compose

If you’re using Docker Compose, follow these steps:

1. **Backup your data:**

   ```bash
   cp -r ~/.n8n ~/.n8n_backup
   ```

2. **Pull the latest version:**

   ```bash
   docker compose pull
   ```

3. **Stop and remove the older version:**

   ```bash
   docker compose down
   ```

4. **Start the updated container:**

   ```bash
   docker compose up -d
   ```

---

## Important Notes

- Ensure your domain's DNS **A record** points to your **server's IP address.**
- **Open ports 80 (HTTP), 443 (HTTPS), and 5678 (n8n) in your firewall.**
- Nginx handles SSL termination, forwarding requests to n8n over HTTP internally.

---

## Troubleshooting

- **Nginx Issues:** Check logs at `/var/log/nginx/error.log` for details.
- **Docker Issues:** Verify the Docker service is running with `sudo systemctl status docker`.

---

**Need Help?**  
**Email:** KunalBarve@articflow.io  
**Subscribe:** [CodeNAutomate](https://youtube.com/@CodeNAutomate) for more no-fluff guides.

---




