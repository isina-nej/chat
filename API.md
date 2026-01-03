# API Documentation

## مقدمه

سامانه چت دارای یک RESTful API است که برای تمام عملیات استفاده می‌شود.

## 🔐 احراز هویت

تمام درخواست‌های API (به جز ورود و ثبت‌نام) به یک JWT token نیاز دارند.

```bash
Authorization: Bearer <token>
```

## 📚 Endpoints

### Authentication API

#### ثبت‌نام

```http
POST /api/auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123",
  "name": "نام کاربر"
}
```

**پاسخ موفق (201):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user-id",
      "email": "user@example.com",
      "name": "نام کاربر",
      "role": "USER"
    },
    "token": "eyJhbGciOiJIUzI1NiIs..."
  },
  "message": "ثبت‌نام موفق"
}
```

#### ورود

```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}
```

**پاسخ موفق (200):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user-id",
      "email": "user@example.com",
      "name": "نام کاربر",
      "role": "USER"
    },
    "token": "eyJhbGciOiJIUzI1NiIs..."
  },
  "message": "ورود موفق"
}
```

### Messages API

#### دریافت پیام‌ها

```http
GET /api/messages?page=1
Authorization: Bearer <token>
```

**پاسخ موفق (200):**
```json
{
  "success": true,
  "data": {
    "messages": [
      {
        "id": "msg-id",
        "content": "سلام!",
        "imageUrl": "/uploads/image.webp",
        "createdAt": "2026-01-03T10:00:00Z",
        "user": {
          "id": "user-id",
          "email": "user@example.com",
          "name": "نام کاربر"
        }
      }
    ],
    "pagination": {
      "page": 1,
      "perPage": 50,
      "total": 100,
      "pages": 2
    }
  }
}
```

#### ارسال پیام

```http
POST /api/messages
Authorization: Bearer <token>
Content-Type: application/json

{
  "content": "سلام دنیا",
  "imageUrl": "/uploads/image.webp"
}
```

**پاسخ موفق (201):**
```json
{
  "success": true,
  "data": {
    "id": "msg-id",
    "content": "سلام دنیا",
    "imageUrl": "/uploads/image.webp",
    "createdAt": "2026-01-03T10:00:00Z",
    "user": {
      "id": "user-id",
      "email": "user@example.com",
      "name": "نام کاربر"
    }
  },
  "message": "پیام با موفقیت ارسال شد"
}
```

#### حذف پیام

```http
DELETE /api/messages/{messageId}
Authorization: Bearer <token>
```

### Upload API

#### آپلود تصویر

```http
POST /api/upload
Authorization: Bearer <token>
Content-Type: multipart/form-data

file: <binary image file>
```

**پاسخ موفق (201):**
```json
{
  "success": true,
  "data": {
    "imageUrl": "/uploads/1234567890-user-id.webp"
  },
  "message": "تصویر با موفقیت آپلود شد"
}
```

### Admin API

#### لیست کاربران

```http
GET /api/admin/users
Authorization: Bearer <token>
```

**نیاز:** ادمین بودن

**پاسخ موفق (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "user-id",
      "email": "user@example.com",
      "name": "نام کاربر",
      "role": "USER",
      "isActive": true,
      "createdAt": "2026-01-03T10:00:00Z",
      "_count": {
        "messages": 5
      }
    }
  ]
}
```

#### تغییر وضعیت کاربر

```http
POST /api/admin/users/{userId}
Authorization: Bearer <token>
Content-Type: application/json

{
  "isActive": false
}
```

#### آمار

```http
GET /api/admin/stats
Authorization: Bearer <token>
```

**پاسخ موفق (200):**
```json
{
  "success": true,
  "data": {
    "totalUsers": 100,
    "activeUsers": 85,
    "totalMessages": 1000,
    "messagesLastDay": 50,
    "stats": [
      {"label": "کل کاربران", "value": 100},
      {"label": "کاربران فعال", "value": 85},
      {"label": "کل پیام‌ها", "value": 1000},
      {"label": "پیام‌های امروز", "value": 50}
    ]
  }
}
```

### Widget API

#### دریافت پیام‌های Widget

```http
GET /api/widget/messages?limit=20
x-api-key: your-api-key
```

**پاسخ موفق (200):**
```json
{
  "success": true,
  "data": {
    "messages": [...],
    "count": 20
  }
}
```

#### ارسال پیام از Widget

```http
POST /api/widget/messages
x-api-key: your-api-key
Content-Type: application/json

{
  "content": "پیام از سایت خارجی",
  "guestEmail": "guest@example.com"
}
```

#### تنظیمات Widget

```http
GET /api/widget/config
```

## 🔔 WebSocket Events

### Client → Server

```javascript
// کاربر وارد می‌شود
socket.emit('user:join', token);

// ارسال پیام
socket.emit('message:send', {
  content: 'سلام',
  imageUrl: '/uploads/image.webp'
});
```

### Server → Client

```javascript
// پیام جدید
socket.on('message:new', (message) => {
  // message = { id, content, imageUrl, user, createdAt }
});

// کاربر آنلاین شد
socket.on('user:online', (data) => {
  // data = { userId, email, name, onlineUsers }
});

// کاربر آفلاین شد
socket.on('user:offline', (data) => {
  // data = { userId, email, onlineUsers }
});

// خطا
socket.on('error', (error) => {
  // error = string message
});
```

## ⚠️ Error Responses

تمام پیام‌های خطا این فرمت را دارند:

```json
{
  "success": false,
  "error": "توضیح خطا"
}
```

### Status Codes

- `200` - OK
- `201` - Created
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `409` - Conflict (مثل ایمیل تکراری)
- `500` - Server Error

## 🧪 مثال‌های cURL

```bash
# ثبت‌نام
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"123456","name":"Test"}'

# ورود
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"123456"}'

# دریافت پیام‌ها
curl -X GET http://localhost:3000/api/messages?page=1 \
  -H "Authorization: Bearer TOKEN"

# ارسال پیام
curl -X POST http://localhost:3000/api/messages \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"سلام"}'

# آپلود تصویر
curl -X POST http://localhost:3000/api/upload \
  -H "Authorization: Bearer TOKEN" \
  -F "file=@/path/to/image.jpg"
```

---

**آخرین بروزرسانی**: 2026-01-03
