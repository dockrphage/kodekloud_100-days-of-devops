## Day 19: Install and Configure Web Application

### **Requirement**
xFusionCorp Industries is planning to host two static websites on their infra in Stratos Datacenter. The development of these websites is still in-progress, but we want to get the servers ready. Please perform the following steps to accomplish the task:



a. Install httpd package and dependencies on app server 1.


b. Apache should serve on port 3000.


c. There are two website's backups /home/thor/official and /home/thor/demo on jump_host. Set them up on Apache in a way that official should work on the link http://localhost:3000/official/ and demo should work on link http://localhost:3000/demo/ on the mentioned app server.


d. Once configured you should be able to access the website using curl command on the respective app server, i.e curl http://localhost:3000/official/ and curl http://localhost:3000/demo/


Here’s a clean, reliable walkthrough you can run directly on **stapp01** (App Server 1) to meet all requirements. I’ll give you the exact commands and explain the logic so you can adapt it for future tasks too.

---

# ✅ **Step‑by‑step solution for hosting two static sites on Apache (port 3000)**

### **1. Log in to App Server 1**
From the jump host:

```bash
ssh tony@stapp01
```
Password: **Ir0nM@n**

---

### **2. Install Apache (httpd)**

```bash
sudo yum install httpd -y
```

---

### **3. Change Apache port to 3000**

Edit the main config:

```bash
sudo sed -i 's/^Listen 80/Listen 3000/' /etc/httpd/conf/httpd.conf
```

Or manually:

```bash
sudo vi /etc/httpd/conf/httpd.conf
```

Change:

```
Listen 80
```

to:

```
Listen 3000
```

---

### **4. Create document roots for both websites**

```bash
sudo mkdir -p /var/www/official
sudo mkdir -p /var/www/demo
```

---

### **5. Copy website backups from jump host**

On **jump_host**, run:

```bash
scp -r /home/thor/official tony@stapp01:/var/www/
scp -r /home/thor/demo tony@stapp01:/var/www/
```

If needed, adjust permissions:

```bash
sudo chown -R apache:apache /var/www/official /var/www/demo
```

---

### **6. Create Apache virtual host configuration**

Create a new config file:

```bash
sudo vi /etc/httpd/conf.d/websites.conf
```

Paste this:

```apache
<VirtualHost *:3000>
    DocumentRoot /var/www
    Alias /official /var/www/official
    <Directory /var/www/official>
        AllowOverride All
        Require all granted
    </Directory>

    Alias /demo /var/www/demo
    <Directory /var/www/demo>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Save and exit.

---

### **7. Start & enable Apache**

```bash
sudo systemctl restart httpd
sudo systemctl enable httpd
```

---

### **8. Test using curl on stapp01**

```bash
curl http://localhost:3000/official/
curl http://localhost:3000/demo/
```

You should see the HTML content from each site.

---

# 🎉 **Done!**
You now have:

- Apache installed  
- Running on port **3000**  
- Two static sites served at:  
  - `http://localhost:3000/official/`  
  - `http://localhost:3000/demo/`  
