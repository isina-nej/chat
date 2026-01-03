# سامانه چت بلادرنگ - راهنمای نصب و تنظیم

## 📋 متطلبات سیستم

- Node.js 18 یا بالاتر
- PostgreSQL 12 یا بالاتر
- npm یا yarn

## 🔧 مراحل نصب

### 1. نصب Packages

```bash
cd nextjs-chat
npm install
```

### 2. تنظیم متغیرهای محیطی

فایل `.env` را ایجاد کنید:

```env
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/chat_db"

# JWT Secret (تغییر دهید!)
JWT_SECRET="your-super-secret-jwt-key-change-in-production"

# Socket.io
NEXT_PUBLIC_SOCKET_URL="http://localhost:3000"

# App
NEXT_PUBLIC_API_URL="http://localhost:3000"
NODE_ENV="development"
```

### 3. ایجاد دیتابیس

```bash
# PostgreSQL را شروع کنید:
# Windows: استارت PostgreSQL از Services
# Linux: sudo systemctl start postgresql
# macOS: brew services start postgresql

# اجرای migrations:
npx prisma migrate dev --name init

# (اختیاری) پایگاه داده Seed:
npx prisma db seed
```

### 4. اجرای برنامه

```bash
npm run dev
```

برنامه در `http://localhost:3000` باز می‌شود.

## 📱 استفاده

### ثبت‌نام و ورود

1. به `http://localhost:3000` بروید
2. **ثبت‌نام**: نام، ایمیل و رمز عبور وارد کنید
3. **ورود**: دوباره با ایمیل و رمز عبور وارد شوید

### چت

- `http://localhost:3000/chat` برای دسترسی به صفحه چت
- پیام‌های شما به سایر کاربران آنلاین ارسال می‌شود
- تصاویر را می‌توانید آپلود کنید

### پنل مدیریت

- `http://localhost:3000/admin` برای مدیریت کاربران (ادمین فقط)

## 🌐 API Testing

Postman یا cURL استفاده کنید:

```bash
# ثبت‌نام
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123",
    "name": "نام کاربر"
  }'

# ورود
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123"
  }'

# دریافت پیام‌ها
curl -X GET http://localhost:3000/api/messages?page=1 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## 📦 ساختار پوشه‌ها

```
nextjs-chat/
├── app/
│   ├── api/
│   │   ├── auth/
│   │   ├── messages/
│   │   ├── upload/
│   │   ├── admin/
│   │   └── widget/
│   ├── auth/
│   ├── chat/
│   ├── admin/
│   └── page.tsx
├── components/
│   └── ChatContainer.tsx
├── lib/
│   ├── auth.ts
│   ├── db.ts
│   ├── password.ts
│   ├── socket.ts
│   └── utils.ts
├── prisma/
│   └── schema.prisma
├── public/
│   └── uploads/
├── .env
├── server.ts
├── next.config.js
└── package.json
```

## 🚀 استقرار بر روی سرور

### استفاده از PM2

```bash
# نصب PM2
npm install -g pm2

# Build پروژه
npm run build

# شروع برنامه
pm2 start npm --name "chat-app" -- run start

# ذخیره configuration
pm2 save
pm2 startup
```

### استفاده از Nginx

```nginx
upstream chat_app {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://chat_app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /uploads {
        alias /path/to/nextjs-chat/public/uploads;
    }
}
```

## 🔐 امنیت

- تغییر `JWT_SECRET` در production
- استفاده از HTTPS برای سرور
- تنظیم firewall برای پورت‌ها
- منظم backup دیتابیس

## 📞 مشکل‌گیری

### دیتابیس متصل نشد

```bash
# بررسی PostgreSQL
psql -U postgres -d chat_db

# مجدد migration
npx prisma migrate reset
```

### Socket.io کار نمی‌کند

- بررسی `NEXT_PUBLIC_SOCKET_URL` در .env
- CORS تنظیم شده است

### خطای آپلود

- دایرکتوری `public/uploads` وجود دارد
- دسترسی write برای دایرکتوری

---

**نسخه**: 1.0.0
**تاریخ**: 2026-01-03
