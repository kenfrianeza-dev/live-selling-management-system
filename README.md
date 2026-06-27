# Live Selling Management System (LSMS)

An internal operations platform for managing live commerce — the business of selling products in real time through live streams on social platforms, marketplaces, and owned channels.

LSMS gives sellers and operations teams a single place to run the back office: staff access, organization settings, billing configuration, and performance visibility. The current release ships the core admin foundation; live-selling modules (streams, orders, inventory, customers) are built on top of this base.

## What It Does

Live selling combines real-time audience engagement with order fulfillment. This system is designed to support that workflow end to end:

| Area | Purpose |
|------|---------|
| **Dashboard** | At-a-glance metrics for revenue, sales volume, and live-session activity |
| **User Management** | Onboard staff, assign roles, and control who can access each module |
| **System Settings** | Configure organization details, localization, auth policies, UI branding, tax/billing rules, and integrations |
| **Access Control** | Granular RBAC so hosts, admins, and support staff see only what they need |

## Current Modules

### Dashboard (`/dashboard`)
Overview page with stat cards and a revenue chart. Metrics are placeholder data today; this will connect to live-session and order data as those modules are added.

### User Management (`/user-management`)
- **Users** — Create, edit, activate/deactivate, and assign roles to staff accounts
- **Roles & Permissions** — Define roles and map fine-grained permissions per module

### System Settings (`/system-settings`)
Grouped, editable configuration across:
- Organization (name, address, timezone, currency)
- Localization (language, country, number formats)
- User & Auth (registration, password rules, session timeout, 2FA toggle)
- UI Branding (app name, colors, favicon)
- System Behavior (maintenance mode)
- Module Toggles (enable/disable feature modules)
- Financial & Billing (tax rate, VAT, invoice/receipt prefixes)
- Developer Integration (API keys, webhooks, payment provider credentials)

## Tech Stack

### Frontend
- **Next.js 16** — App Router, Server Components, Server Actions
- **React 19** — UI library
- **TypeScript** — End-to-end type safety
- **Tailwind CSS 4** — Utility-first styling
- **shadcn/ui + Radix UI** — Accessible component primitives
- **Recharts** — Dashboard charts
- **Zustand** — Client-side state
- **TanStack Table** — Data tables for user/role management

### Backend & Database
- **Prisma ORM 7** — Database access and migrations
- **PostgreSQL 18** — Primary datastore
- **JOSE** — JWT-based session handling

### Development & DevOps
- **pnpm** — Package management
- **Docker & Docker Compose** — Local development environment
- **Vitest** — Unit testing
- **ESLint** — Linting

## Architecture

The codebase follows a layered structure:

```
app/          → Routes, UI, and Server Actions (presentation)
domain/       → Business logic, policies, and repositories
lib/          → Auth, Prisma client, shared server utilities
prisma/       → Schema, migrations, and seeders
```

Permission checks flow through a central `PermissionEngine` in `domain/shared/`, supporting both role-based and direct user permissions.

## Prerequisites

- **Node.js** 18+
- **pnpm** 9+ (`npm install -g pnpm`)
- **Docker & Docker Compose** (recommended)
- **PostgreSQL** 18+ (if running without Docker)
- **Git**

## Installation & Setup

### Option 1: Docker (Recommended)

#### 1. Clone the repository
```bash
git clone <repository-url>
cd live-selling-management-system
```

#### 2. Create environment variables
Copy `.env.example` to `.env` and adjust values as needed:

```env
NEXT_PUBLIC_APP_NAME="Live Selling Management System"
NEXT_PUBLIC_SHORT_NAME="LSMS"
DATABASE_URL="postgresql://root:secret@db:5432/homestead?schema=public"
PASSWORD_SALT="your-secure-password-salt-here-change-this"
NODE_ENV="development"
JWT_SECRET=<generate-a-secure-secret>
JWT_REFRESH_SECRET=<generate-a-secure-secret>
SUPER_ADMIN_PASSWORD=ChangeMeImmediately!
```

#### 3. Build and start
```bash
docker compose build
docker compose up
```

This will:
- Start PostgreSQL on host port **5433**
- Start the Next.js app on host port **3001**
- Run `pnpm install`, generate the Prisma client, apply migrations, and start the dev server

#### 4. Seed the database (first run)
```bash
docker compose exec app pnpm prisma db seed
```

#### 5. Open the app
Navigate to [http://localhost:3001](http://localhost:3001)

Default dev credentials are printed in the container logs after seeding (Super Admin, Admin, and User accounts).

#### 6. Stop
```bash
docker compose down
```

---

### Option 2: Local Setup (Without Docker)

#### 1. Clone and install
```bash
git clone <repository-url>
cd live-selling-management-system
pnpm install
```

#### 2. Start PostgreSQL
Either use Docker for the database only:
```bash
docker run --name lsms-db \
  -e POSTGRES_USER=root \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=homestead \
  -p 5433:5432 \
  -d postgres:18
```

Or use a local PostgreSQL installation and create the `homestead` database.

#### 3. Configure environment
Create `.env.local`:

```env
NEXT_PUBLIC_APP_NAME="Live Selling Management System"
NEXT_PUBLIC_SHORT_NAME="LSMS"
DATABASE_URL="postgresql://root:secret@localhost:5433/homestead?schema=public"
PASSWORD_SALT="your-secure-password-salt-here-change-this"
NODE_ENV="development"
JWT_SECRET=<generate-a-secure-secret>
JWT_REFRESH_SECRET=<generate-a-secure-secret>
SUPER_ADMIN_PASSWORD=ChangeMeImmediately!
```

#### 4. Run migrations and seed
```bash
pnpm prisma migrate dev
pnpm prisma db seed
```

#### 5. Start the dev server
```bash
pnpm dev
```

The app will be available at [http://localhost:3000](http://localhost:3000).

## Available Scripts

```bash
# Development
pnpm dev              # Start Next.js dev server
pnpm build            # Production build
pnpm start            # Start production server
pnpm test             # Run Vitest tests
pnpm lint             # Run ESLint

# Database
pnpm prisma migrate dev       # Create and apply migrations
pnpm prisma migrate reset     # Reset database (development only)
pnpm prisma studio            # Open Prisma Studio GUI
pnpm prisma db seed           # Seed all data (permissions → auth → settings)
pnpm prisma generate          # Regenerate Prisma Client

# Modular seeding (run individual seeders)
pnpm prisma db seed -- permissions-seeder
pnpm prisma db seed -- auth-seeder
pnpm prisma db seed -- settings-seeder
pnpm prisma db seed -- user-stress-seeder
```

## Project Structure

```
live-selling-management-system/
├── app/
│   ├── (admin)/                    # Protected admin routes
│   │   ├── dashboard/              # Metrics overview
│   │   ├── system-settings/      # Configurable system settings
│   │   └── user-management/      # Users, roles & permissions
│   ├── (public)/                   # Login and public pages
│   ├── _components/                # Shared UI (nav, layout, shadcn/ui)
│   ├── auth-actions/               # Auth server actions
│   ├── config/navigation/          # Sidebar navigation config
│   ├── signup/                     # Self-registration
│   └── unauthorized/               # Access-denied page
├── domain/
│   ├── shared/permission.engine.ts # RBAC evaluation
│   ├── system/                     # System settings service layer
│   └── user-management/            # User/role service layer
├── lib/
│   ├── auth.ts                     # JWT session management
│   ├── prisma.ts                   # Prisma client singleton
│   └── permissions.ts              # Permission helpers
├── prisma/
│   ├── schema.prisma               # Database schema
│   ├── seed.ts                     # Seeder coordinator
│   ├── migrations/                 # Migration history
│   └── seeders/                    # Modular seed files
├── docker-compose.yml
├── Dockerfile
└── package.json
```

## Database Schema

### Auth & Access Control
| Model | Description |
|-------|-------------|
| **User** | Staff accounts (email, name, phone, password hash) |
| **Role** | Named roles (SuperAdmin, Admin, User) |
| **Permission** | Action + resource pairs (e.g. `read:user-management.users`) |
| **UserRole** | User-to-role assignments with optional scope |
| **RolePermission** | Role-to-permission mappings |
| **UserPermission** | Direct user-level permission overrides |
| **Session** | Active login sessions |
| **AuditLog** | Activity trail for compliance |

### Configuration
| Model | Description |
|-------|-------------|
| **SystemSettingCategory** | Grouped setting sections (Organization, Billing, etc.) |
| **SystemSetting** | Individual key/value settings with type and sensitivity flags |

## Default Roles

| Role | Access |
|------|--------|
| **SuperAdmin** | Full system access (`*:*`) |
| **Admin** | Dashboard, read/manage users & roles, read/update organization and module settings |
| **User** | Dashboard only |

## Environment Variables

| Variable | Description |
|----------|-------------|
| `NEXT_PUBLIC_APP_NAME` | Display name shown in the UI |
| `NEXT_PUBLIC_SHORT_NAME` | Short abbreviation (e.g. LSMS) |
| `DATABASE_URL` | PostgreSQL connection string |
| `PASSWORD_SALT` | Salt for password hashing |
| `JWT_SECRET` | Secret for access tokens |
| `JWT_REFRESH_SECRET` | Secret for refresh tokens |
| `SUPER_ADMIN_PASSWORD` | Initial Super Admin password (dev seeding) |
| `NODE_ENV` | `development` or `production` |

## Development Workflow

### Database changes
1. Edit `prisma/schema.prisma`
2. Run `pnpm prisma migrate dev --name describe_your_change`
3. Use `import { prisma } from '@/lib/prisma'` in server code

### Adding a new module
1. Create domain service/repo/policy under `domain/`
2. Add permissions in `prisma/seeders/permissions-seeder.ts`
3. Add a nav item in `app/config/navigation/items/`
4. Register it in `app/config/navigation/navigation-config.ts`
5. Create the route under `app/(admin)/`

## Deployment

### Vercel
1. Push to GitHub
2. Import the repo on [vercel.com](https://vercel.com)
3. Set environment variables in the project dashboard
4. Deploy

### Docker
```bash
docker build -t lsms:latest .
docker tag lsms:latest <registry>/lsms:latest
docker push <registry>/lsms:latest
```

Deploy the image with your orchestrator of choice (Docker Compose, Kubernetes, etc.) and point `DATABASE_URL` at a managed PostgreSQL instance.

## Troubleshooting

### Port already in use
```bash
# App (local dev)
lsof -ti:3000 | xargs kill -9

# App (Docker)
lsof -ti:3001 | xargs kill -9
```

### Database connection failed
- Confirm PostgreSQL is running
- Check `DATABASE_URL` — Docker uses port **5433** on the host
- Test: `psql $DATABASE_URL`

### Prisma client out of sync
```bash
docker compose exec app pnpm prisma generate
docker restart lsms-app
```

### Tables missing
```bash
docker compose exec app pnpm prisma migrate dev
```

### Reset Docker environment
```bash
docker compose down -v
docker compose up --build
```

## Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Prisma Documentation](https://www.prisma.io/docs/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [shadcn/ui Components](https://ui.shadcn.com/)
