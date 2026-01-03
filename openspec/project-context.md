# Project Context

## Purpose

سیستم چت بلادرنگ (Real-time Chat) برای ارتباطات آنی میان کاربران با قابلیت ادغام در سایت‌های دیگر. این پروژه بر اساس یک PRD فارسی ساخته می‌شود و باید روی سرور شخصی (Self-hosted) اجرا شود.

## Tech Stack

- **Frontend:** React 18+, Next.js 14+ (App Router), TypeScript, Tailwind CSS
- **Backend:** Next.js API Routes, Node.js 18+
- **Database:** PostgreSQL 12+, Prisma ORM
- **Real-time:** Socket.io
- **Authentication:** JWT, bcryptjs
- **File Upload:** formidable, sharp (image compression)
- **Deployment:** PM2, Nginx
- **Testing:** Jest, React Testing Library

## Project Conventions

### Code Style

- **Language:** TypeScript (strict mode)
- **Formatting:** Prettier with 2-space indentation
- **Naming:** 
  - Components: PascalCase (e.g., `ChatWidget.tsx`)
  - Files: kebab-case (e.g., `message-service.ts`)
  - Functions: camelCase (e.g., `sendMessage()`)
  - Constants: UPPER_SNAKE_CASE
- **Imports:** Absolute paths using `@/` alias

### Architecture Patterns

- **API Routes:** One concern per file
- **Components:** Functional components with hooks
- **Services:** Separated business logic in `lib/`
- **Database:** Prisma models with relations
- **Socket.io:** Event-driven architecture
- **State Management:** React Context + hooks

### Testing Strategy

- **Unit Tests:** API routes, utilities, services
- **Integration Tests:** Socket.io events, database operations
- **E2E Tests:** User flows (register → chat → upload)
- **Coverage Target:** 70%+

### Git Workflow

- **Branches:** feature/*, bugfix/*, refactor/*
- **Commits:** Conventional commits (feat:, fix:, docs:, etc.)
- **PRs:** Required for all changes

## Domain Context

- **Real-time Chat:** WebSocket-based messaging system
- **User Roles:** ADMIN (مدیریت), USER (کاربر عادی)
- **File Management:** Image uploads in local filesystem
- **External Integration:** API + Widget for third-party sites

## Important Constraints

1. **Self-hosted:** باید روی سرور شخصی اجرا شود (نه cloud)
2. **Bandwidth:** محدودیت bandwidth برای uploads
3. **Storage:** محدودیت storage برای فایل‌های آپلود شده
4. **Performance:** پیام‌ها باید در < 200ms منتقل شوند
5. **Privacy:** داده‌ها محلی و تحت کنترل مالک

## External Dependencies

- PostgreSQL: نیاز به نصب و پیکربندی
- PM2: برای process management
- Nginx: برای reverse proxy و SSL
- Node.js: Runtime environment
