# 🚀 راهنمای استقرار برروی سرور

راهنمای کامل برای استقرار سامانه چت بر روی سرور شخصی.

## 📋 متطلبات

- سرور Linux (Ubuntu 20.04+)
- Node.js 18+
- PostgreSQL 12+
- Nginx (اختیاری)
- SSL Certificate (اختیاری)

## 🔧 مراحل استقرار

### 1️⃣ نصب Node.js و npm

```bash
# بروز رسانی packages
sudo apt update && sudo apt upgrade -y

# نصب Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# تایید نصب
node --version
npm --version
```

### 2️⃣ نصب PostgreSQL

```bash
# نصب PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# شروع service
sudo systemctl start postgresql
sudo systemctl enable postgresql

# ایجاد کاربر و دیتابیس
sudo -u postgres psql << EOF
CREATE USER chatuser WITH PASSWORD 'strong_password_here';
CREATE DATABASE chat_db OWNER chatuser;
GRANT ALL PRIVILEGES ON DATABASE chat_db TO chatuser;
\q
EOF
```

### 3️⃣ نصب و تنظیم برنامه

```bash
# ایجاد پوشه برنامه
mkdir -p /var/www/chat-app
cd /var/www/chat-app

# Clone یا کپی پروژه
# git clone <repo> . 
# یا copy files manually

# نصب dependencies
npm install --production

# ایجاد .env
cat > .env << EOF
DATABASE_URL="postgresql://chatuser:strong_password_here@localhost:5432/chat_db"
JWT_SECRET="your-secure-random-key-generate-this"
NEXT_PUBLIC_SOCKET_URL="https://yourdomain.com"
NEXT_PUBLIC_API_URL="https://yourdomain.com"
NODE_ENV="production"
EOF

# تولید build
npm run build
```

### 4️⃣ نصب PM2 (Process Manager)

```bash
# نصب PM2 globally
sudo npm install -g pm2

# شروع برنامه
cd /var/www/chat-app
pm2 start npm --name "chat-app" -- run start

# تنظیم خودکار شروع در boot
pm2 startup systemd -u $USER --hp /home/$USER
pm2 save

# مشاهده logs
pm2 logs chat-app
```

### 5️⃣ تنظیم Nginx (Reverse Proxy)

```bash
# نصب Nginx
sudo apt install -y nginx

# ایجاد فایل config
sudo nano /etc/nginx/sites-available/chat-app
```

معماری Nginx (`/etc/nginx/sites-available/chat-app`):

```nginx
upstream chat_app {
    server 127.0.0.1:3000;
}

# HTTP to HTTPS redirect
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS Server
server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL Certificates
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Logging
    access_log /var/log/nginx/chat-access.log;
    error_log /var/log/nginx/chat-error.log;

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css text/javascript application/json;

    # Root location
    location / {
        proxy_pass http://chat_app;
        proxy_http_version 1.1;
        
        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Static files
    location /uploads {
        alias /var/www/chat-app/public/uploads;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    location /_next/static {
        alias /var/www/chat-app/.next/static;
        expires 365d;
        add_header Cache-Control "public, immutable";
    }
}
```

Enable و restart Nginx:

```bash
# Link to enabled sites
sudo ln -s /etc/nginx/sites-available/chat-app /etc/nginx/sites-enabled/

# Remove default site
sudo rm /etc/nginx/sites-enabled/default

# Test config
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
sudo systemctl enable nginx
```

### 6️⃣ SSL Certificate (Let's Encrypt)

```bash
# نصب Certbot
sudo apt install -y certbot python3-certbot-nginx

# ایجاد certificate
sudo certbot certonly --nginx -d yourdomain.com -d www.yourdomain.com

# Auto renewal
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer
```

### 7️⃣ Firewall تنظیمات

```bash
# Enable UFW
sudo ufw enable

# Allow SSH
sudo ufw allow 22/tcp

# Allow HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Check status
sudo ufw status
```

## 📊 Monitoring و Maintenance

### Logs مشاهده

```bash
# Application logs
pm2 logs chat-app

# Nginx logs
tail -f /var/log/nginx/chat-access.log
tail -f /var/log/nginx/chat-error.log

# PostgreSQL logs
sudo tail -f /var/log/postgresql/postgresql.log
```

### Backup دیتابیس

```bash
# Manual backup
pg_dump -U chatuser chat_db > /backups/chat_db_$(date +%Y%m%d).sql

# Restore
psql -U chatuser chat_db < /backups/chat_db_backup.sql

# Automated backup (cron)
0 2 * * * pg_dump -U chatuser chat_db | gzip > /backups/chat_db_$(date +\%Y\%m\%d).sql.gz
```

### Update برنامه

```bash
cd /var/www/chat-app

# Pull latest code
git pull origin main

# Install new dependencies
npm install --production

# Run migrations
npx prisma migrate deploy

# Rebuild
npm run build

# Restart app
pm2 restart chat-app
```

## 🔒 Security Best Practices

```bash
# تغییر مالکیت
sudo chown -R www-data:www-data /var/www/chat-app

# تنظیم permissions
sudo chmod 755 /var/www/chat-app
sudo chmod 644 /var/www/chat-app/.env

# Disable root login
sudo sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# Fail2Ban برای protect کردن
sudo apt install -y fail2ban
sudo systemctl enable fail2ban
```

## 📈 Performance Tuning

### PM2 Cluster Mode

```bash
pm2 start npm --name "chat-app" -i max -- run start
```

### Nginx Caching

```nginx
# Add to http block
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=api_cache:10m max_size=1g inactive=60m;

# In server block for specific locations
location ~* ^/api/messages {
    proxy_cache api_cache;
    proxy_cache_valid 200 10m;
    add_header X-Cache-Status $upstream_cache_status;
}
```

## 🆘 Troubleshooting

### برنامه شروع نمی‌شود

```bash
# Check PM2 status
pm2 status

# Check logs
pm2 logs chat-app

# Manually run to see errors
npm run start
```

### Database Connection Error

```bash
# Check PostgreSQL
sudo systemctl status postgresql

# Test connection
psql -U chatuser -d chat_db -h localhost
```

### WebSocket Not Working

- بررسی Nginx config برای Upgrade headers
- بررسی firewall برای port 443
- Check NEXT_PUBLIC_SOCKET_URL در .env

---

**نسخه**: 1.0.0
**آخرین بروزرسانی**: 2026-01-03
