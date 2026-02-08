
## Day 16: Install and Configure Nginx as LBR

### Task
Day by day traffic is increasing on one of the websites managed by the Nautilus production support team. The team has decided to deploy this application on a high availability stack in Stratos DC. The migration is almost complete, and only the LBR server configuration is pending.

### Requirements
- Install **nginx** on the LBR server (`stlb01`).
- Configure load balancing in the main Nginx configuration file `/etc/nginx/nginx.conf`.
- Use all App Servers (`stapp01`, `stapp02`, `stapp03`) in the upstream pool.
- Do **not** change Apache ports on the app servers.
- Ensure Apache (`httpd`) service is running on all app servers.
- Verify access via the StaticApp button or by curling the LBR IP.

---

## Steps Taken

### 1. Install Nginx on LBR
```bash
ssh loki@stlb01.stratos.xfusioncorp.com
sudo dnf update -y
sudo dnf install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```

### 2. Verify Apache Ports on App Servers
On each app server:
```bash
ssh <user>@<hostname>
sudo grep -i listen /etc/httpd/conf/httpd.conf
ss -tulnp | grep httpd
```

- `stapp01` → Listen 8086  
- `stapp02` → Listen 8086  
- `stapp03` → Listen 8086  

### 3. Update Nginx Configuration
Edited `/etc/nginx/nginx.conf`:

```nginx
http {
    upstream stapp {
        server 172.16.238.10:8086;
        server 172.16.238.11:8086;
        server 172.16.238.12:8086;
    }

    server {
        listen 80;
        server_name _;

        location / {
            proxy_pass http://stapp;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";

            proxy_connect_timeout 5s;
            proxy_read_timeout 60s;
        }
    }
}
```

### 4. Test and Reload Nginx
```bash
sudo nginx -t
sudo systemctl reload nginx
```

### 5. Verification
- Curl each app server directly:
```bash
curl http://172.16.238.10:8086
curl http://172.16.238.11:8086
curl http://172.16.238.12:8086
```
→ All returned Apache welcome page.

- Curl the LBR:
```bash
curl http://172.16.238.14
```
→ Returns content served by upstream pool.

- Tail logs to confirm rotation:
```bash
sudo tail -f /var/log/nginx/access.log
```
→ Shows requests distributed across all three upstream servers.

---

## Learnings
- Always confirm Apache ports before configuring Nginx upstream.
- Do not modify app server configs; match Nginx to existing ports.
- Use `$upstream_addr` in log format to prove load balancing.
- Identical content across app servers means curl output looks the same, but logs confirm distribution.

---

## Progress Tracker
- [x] Day 16: Nginx LBR configured and verified
```

---
