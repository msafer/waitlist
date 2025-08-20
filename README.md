# Waitlist App

A minimal waitlist application with SIWE authentication, Farcaster FID linking, and Lens Protocol verification for priority access.

## Architecture

- **Frontend**: Next.js + Tailwind CSS + SIWE (deployed to Vercel)
- **Backend**: Express + Prisma + Redis (deployed to Railway)
- **Database**: SQLite (dev) / PostgreSQL (production)
- **Authentication**: Sign-In with Ethereum (SIWE)
- **Monitoring**: Sentry for both frontend and backend

## Features

### Core Functionality
- **Wallet Authentication**: SIWE-based wallet connection
- **Waitlist Management**: Join waitlist with pending status
- **Priority System**: Lens verification grants priority status and score boost
- **Social Linking**: Connect Farcaster FID and Lens profiles
- **Admin Dashboard**: Batch approval system for waitlist management

### Technical Features
- **Rate Limiting**: Redis-backed rate limiting for API endpoints
- **Audit Logging**: Track all user actions and status changes
- **Priority Scoring**: Points system for queue positioning
- **Error Monitoring**: Sentry integration for production monitoring
- **Responsive UI**: orb.club-inspired design with dark mode

## Database Models

```prisma
model User {
  id              String         @id @default(cuid())
  wallet          String         @unique
  fid             Int?           // Farcaster ID
  lensProfileId   String?        // Lens profileId
  lensOwnerWallet String?        // Lens owner wallet
  status          WaitlistStatus @default(PENDING)
  priorityScore   Int            @default(0)
  createdAt       DateTime       @default(now())
  updatedAt       DateTime       @updatedAt
}

enum WaitlistStatus {
  PENDING       // Default status
  PRIORITY_LENS // After Lens verification
  APPROVED      // Admin approved
  REJECTED      // Admin rejected
}

model LinkAttempt {
  id          String   @id @default(cuid())
  userId      String
  profileId   String
  owner       String
  nonce       String   @unique
  expiresAt   DateTime
  createdAt   DateTime @default(now())
}

model AuditLog {
  id          String   @id @default(cuid())
  userId      String
  action      String   // e.g., "JOINED_WAITLIST", "LINKED_LENS"
  details     String?  // JSON string for additional data
  createdAt   DateTime @default(now())
}
```

## API Endpoints

### Authentication
- `POST /api/auth/siwe` - Sign in with Ethereum
- `POST /api/auth/logout` - Sign out

### Waitlist
- `POST /api/waitlist/join` - Join the waitlist
- `GET /api/waitlist/status` - Get user's waitlist status and position

### Social Linking
- `POST /api/link/farcaster/link` - Link Farcaster FID (+50 priority points)
- `POST /api/link/lens/start` - Start Lens profile verification
- `POST /api/link/lens/verify` - Verify Lens ownership (+100 points, PRIORITY_LENS status)

### Admin
- `POST /api/admin/batch-approve` - Batch approve users (requires admin key)
- `GET /api/admin/stats` - Get waitlist statistics

## Getting Started

### Backend Setup

1. Install dependencies:
```bash
npm install
```

2. Set up environment variables:
```bash
cp .env.example .env
```

Edit `.env`:
```
# Database
DATABASE_URL="file:./dev.db"

# APIs
NEYNAR_API_KEY=your_neynar_api_key
LENS_API_URL=https://api-v2.lens.dev

# Auth
SESSION_SECRET=your_super_secret_key

# Redis (optional for dev)
REDIS_URL=redis://localhost:6379

# Sentry
SENTRY_DSN=your_sentry_dsn

# Admin
ADMIN_KEY=your_admin_secret_key

# Server
PORT=3000
NODE_ENV=development
```

3. Set up database:
```bash
npm run db:migrate
```

4. Start backend:
```bash
npm run dev
```

### Frontend Setup

1. Navigate to frontend:
```bash
cd frontend
```

2. Install dependencies:
```bash
npm install
```

3. Set up environment variables:
```bash
cp .env.example .env.local
```

Edit `.env.local`:
```
NEXT_PUBLIC_API_URL=http://localhost:3000
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_wc_project_id
NEXT_PUBLIC_SENTRY_DSN=your_frontend_sentry_dsn
```

4. Start frontend:
```bash
npm run dev
```

Visit `http://localhost:3001` for the frontend.

## User Flow

### 1. Landing Page
- Clean cards showing "Join Waitlist" and "Priority with Lens"
- Connect wallet with MetaMask/WalletConnect
- View features and benefits

### 2. Join Waitlist
- Sign SIWE message to authenticate
- Automatically join waitlist with PENDING status
- Redirect to dashboard

### 3. Dashboard
- View waitlist status chip (Pending/Priority/Approved)
- See priority score meter
- Link Farcaster FID for +50 points
- Link Lens profile for +100 points and PRIORITY_LENS status
- "Open App" button appears when status = APPROVED

### 4. Lens Verification Flow
- Enter Lens profile handle/ID
- Backend generates verification message
- Sign message with Lens profile owner wallet
- On success: status → PRIORITY_LENS, +100 priority score

### 5. Admin Management
- Batch approve users by count or specific user IDs
- View waitlist statistics
- Audit trail for all actions

## Priority System

Users earn priority points and status upgrades:

- **Base**: 0 points, PENDING status
- **Farcaster**: +50 points (keeps PENDING status)
- **Lens**: +100 points, upgraded to PRIORITY_LENS status

Queue position determined by:
1. Priority score (descending)
2. Join date (ascending)

## Deployment

### Frontend (Vercel)
1. Connect GitHub repo to Vercel
2. Set environment variables in Vercel dashboard
3. Deploy automatically on push to main

### Backend (Railway)
1. Connect GitHub repo to Railway
2. Set environment variables in Railway dashboard
3. Add PostgreSQL database service
4. Configure Redis service for rate limiting
5. Deploy automatically on push to main

### Environment Variables

**Backend (Railway)**:
```
NODE_ENV=production
DATABASE_URL=postgresql://... (from Railway)
REDIS_URL=redis://... (from Railway)
NEYNAR_API_KEY=your_key
SENTRY_DSN=your_backend_dsn
ADMIN_KEY=your_admin_key
SESSION_SECRET=production_secret
```

**Frontend (Vercel)**:
```
NEXT_PUBLIC_API_URL=https://your-backend.railway.app
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_project_id
NEXT_PUBLIC_SENTRY_DSN=your_frontend_dsn
```

## UI Design

Inspired by orb.club aesthetic:
- **Dark gradient background**: Gray-900 to blue-900 to purple-900
- **Glass morphism cards**: White/5 background with backdrop blur
- **Generous spacing**: Large padding and margins
- **Rounded corners**: Rounded-3xl for cards, rounded-full for buttons
- **Gradient accents**: Blue to purple gradients for CTAs
- **Clean typography**: Inter font family
- **Subtle borders**: White/10 borders with hover states

## Security

- **SIWE Authentication**: Cryptographic wallet verification
- **Rate Limiting**: Redis-backed protection against abuse
- **Admin Authorization**: Secret key protection for admin endpoints
- **Input Validation**: All user inputs validated
- **Session Security**: Secure session configuration
- **Error Monitoring**: Sentry tracks production issues

## Development Scripts

**Backend**:
- `npm run dev` - Development server with hot reload
- `npm run build` - Build TypeScript
- `npm run start` - Production server
- `npm run db:migrate` - Run database migrations
- `npm run db:studio` - Open Prisma Studio

**Frontend**:
- `npm run dev` - Next.js development server
- `npm run build` - Build for production
- `npm run start` - Production server
- `npm run lint` - ESLint checking

## License

MIT License - see LICENSE file for details.