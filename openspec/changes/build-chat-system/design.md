# Technical Design

**Change ID:** `build-chat-system`

## Architecture Overview

```
┌─────────────────────────────────────┐
│      Next.js Full-Stack App         │
├─────────────────────────────────────┤
│  Frontend (React)                   │
│  - Pages: /chat, /admin, /auth      │
│  - Components: ChatWidget, etc      │
├─────────────────────────────────────┤
│  Backend (Next.js API Routes)       │
│  - /api/auth (login/register)       │
│  - /api/messages (CRUD)             │
│  - /api/upload (image upload)       │
│  - /api/widget (external API)       │
├─────────────────────────────────────┤
│  WebSocket Server (Socket.io)       │
│  - Real-time message broadcasting   │
│  - Online/Offline status            │
├─────────────────────────────────────┤
│  Database (PostgreSQL)              │
│  - Users, Messages, Uploads         │
└─────────────────────────────────────┘
```

## Database Schema

```prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  password  String   // bcryptjs hashed
  name      String?
  role      Role     @default(USER) // ADMIN or USER
  isActive  Boolean  @default(true)
  messages  Message[]
  createdAt DateTime @default(now())
}

enum Role {
  USER
  ADMIN
}

model Message {
  id        String   @id @default(cuid())
  content   String?  // optional if image-only
  imageUrl  String?  // path to uploaded image
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())
  @@index([userId])
  @@index([createdAt])
}
```

## Folder Structure

```
project/
├── app/
│   ├── api/
│   │   ├── auth/
│   │   │   ├── register/route.ts
│   │   │   └── login/route.ts
│   │   ├── messages/
│   │   │   ├── route.ts
│   │   │   └── [id]/route.ts
│   │   ├── upload/
│   │   │   └── route.ts
│   │   └── widget/
│   │       └── route.ts
│   ├── chat/
│   │   └── page.tsx
│   ├── admin/
│   │   ├── page.tsx
│   │   └── users/page.tsx
│   ├── auth/
│   │   ├── register/page.tsx
│   │   └── login/page.tsx
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── ChatWidget.tsx
│   ├── MessageList.tsx
│   ├── MessageInput.tsx
│   └── AdminPanel.tsx
├── lib/
│   ├── auth.ts
│   ├── socket.ts
│   └── db.ts
├── prisma/
│   └── schema.prisma
├── public/
│   └── uploads/
├── middleware.ts
└── env.local
```

## API Endpoints

### Authentication
- `POST /api/auth/register` - ثبت‌نام کاربر
- `POST /api/auth/login` - ورود کاربر

### Messages
- `GET /api/messages` - دریافت لیست پیام‌ها (pagination)
- `POST /api/messages` - ارسال پیام
- `GET /api/messages/[id]` - دریافت یک پیام
- `DELETE /api/messages/[id]` - حذف پیام

### File Upload
- `POST /api/upload` - آپلود تصویر

### Admin
- `GET /api/admin/users` - لیست کاربران
- `POST /api/admin/users/[id]/block` - بلاک کردن کاربر
- `GET /api/admin/stats` - آمار سیستم

### Widget (External)
- `GET /api/widget/config` - تنظیمات widget
- `GET /api/widget/messages` - دریافت پیام‌ها برای widget

## WebSocket Events

**From Client:**
- `connect` - کاربر متصل می‌شود
- `disconnect` - کاربر قطع می‌شود
- `message:send` - ارسال پیام

**To Client:**
- `message:new` - پیام جدید
- `user:online` - کاربر آنلاین شد
- `user:offline` - کاربر آفلاین شد

## Security Considerations

1. **Password:** bcryptjs hashing
2. **JWT:** Secret key in .env
3. **CORS:** Configure for external widgets
4. **File Upload:** 
   - Whitelist extensions (jpg, png, webp)
   - Max size: 5MB
   - Scan for viruses
5. **SQL Injection:** Prisma protection
6. **XSS:** React escaping + sanitization
7. **CSRF:** Token validation

## Performance Optimizations

1. **Database Indexing:** userId, createdAt
2. **Message Pagination:** 50 messages per page
3. **Image Compression:** automatic via sharp
4. **Caching:** Redis for online users (optional)
5. **Socket.io Rooms:** per-channel isolation

