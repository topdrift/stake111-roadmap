# Stake111 - Cricket Betting Platform

<div align="center">

**A full-stack, production-ready cricket betting platform with agent hierarchy, casino games, and real-time updates.**

`Next.js 14` | `Express.js` | `PostgreSQL 16` | `Redis 7` | `Docker` | `Socket.io`

</div>

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [User Roles & Hierarchy](#user-roles--hierarchy)
- [Panel Features](#panel-features)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Docker Deployment](#docker-deployment)
- [API Reference](#api-reference)
- [Database Schema](#database-schema)
- [Frontend Routes](#frontend-routes)
- [Casino Engine](#casino-engine)
- [Cron Jobs](#cron-jobs)
- [Troubleshooting](#troubleshooting)

---

## Overview

Stake111 is an agent-based cricket betting platform with three distinct panels:

| Panel | Domain | Role |
|-------|--------|------|
| **Player Panel** | `stake111.co` | Players place bets, play casino games |
| **Agent Panel** | `agent.stake111.co` | Agents manage players, credits, reports |
| **Master Panel** | `master.stake111.co` | Master admin controls the entire platform |

---

## Architecture

```
                    +-----------+
                    |   Nginx   |  (Reverse Proxy)
                    +-----+-----+
                          |
            +-------------+-------------+
            |                           |
      +-----+-----+             +------+------+
      |  Frontend  |             |   Backend   |
      |  (Next.js) |             |  (Express)  |
      |  Port 3000 |             |  Port 5000  |
      +-----+-----+             +------+------+
            |                      |         |
            |                +-----+--+  +---+----+
            |                |Postgres|  | Redis  |
            |                | :5432  |  | :6379  |
            +----------------+--------+  +--------+
```

### Key Architecture Decisions

- **MVC Pattern**: Controllers -> Services -> Prisma ORM -> PostgreSQL
- **File-based Routing**: Next.js App Router with grouped layouts per role
- **Real-time**: Socket.io for live match updates and notifications
- **State Management**: Zustand stores (auth, bets, matches, notifications, theme)
- **Service Layer**: Axios-based API services mirror backend routes
- **Background Jobs**: node-cron for settlements, match sync, cleanup
- **Caching**: Redis for sessions and frequently accessed data
- **Security**: Helmet, CORS, rate limiting, input sanitization, bcrypt

---

## Tech Stack

### Backend
| Technology | Version | Purpose |
|-----------|---------|---------|
| Node.js | 18+ | Runtime |
| Express.js | 4.18 | HTTP framework |
| TypeScript | 5.x | Type safety |
| Prisma | 5.7 | ORM & migrations |
| PostgreSQL | 16 | Primary database |
| Redis | 7 | Caching & sessions |
| Socket.io | 4.6 | Real-time WebSocket |
| Joi | 17.11 | Request validation |
| Winston | 3.11 | Logging |
| node-cron | 3.0 | Scheduled jobs |
| bcryptjs | 2.4 | Password hashing |
| jsonwebtoken | - | JWT auth |

### Frontend
| Technology | Version | Purpose |
|-----------|---------|---------|
| Next.js | 14.2 | React framework (App Router) |
| React | 18 | UI library |
| TypeScript | 5.x | Type safety |
| Tailwind CSS | 3.4 | Styling |
| Zustand | 5.0 | State management |
| Radix UI | - | Accessible UI primitives |
| Recharts | 3.7 | Charts & graphs |
| Lucide React | - | Icon library |
| Axios | 1.13 | HTTP client |
| socket.io-client | 4.8 | WebSocket client |

### Infrastructure
| Technology | Purpose |
|-----------|---------|
| Docker & Docker Compose | Containerization |
| Nginx | Reverse proxy, SSL termination |
| dumb-init | PID 1 process management |

---

## User Roles & Hierarchy

```
MASTER_ADMIN
    |
    +-- SUPER_MASTER (Agent)
    |       |
    |       +-- MASTER (Agent)
    |               |
    |               +-- AGENT
    |                     |
    |                     +-- PLAYER
    |                     +-- PLAYER
    |
    +-- AGENT (Direct under master)
            |
            +-- PLAYER
```

### Role Permissions

| Permission | Player | Agent | Master | Master Admin |
|-----------|--------|-------|--------|--------------|
| Place bets | Yes | Yes (agent bets) | No | No |
| Play casino | Yes | No | No | No |
| Create players | No | Yes | Yes | Yes |
| Transfer credits | No | Yes | Yes | Yes |
| View reports | Own only | Downline | All | All |
| Manage matches | No | View | Import/Manage | Full control |
| Platform settings | No | No | No | Yes |
| Commission config | No | No | No | Yes |
| Betting limits | No | No | No | Yes |
| Settlement management | No | No | Generate/Approve | Full control |
| Audit logs | No | No | No | View |

---

## Panel Features

### Player Panel (`stake111.co`)

| Page | Route | Description |
|------|-------|-------------|
| Dashboard | `/dashboard` | Balance, recent bets, live matches |
| Matches | `/matches` | Browse all available matches |
| Match Detail | `/matches/[id]` | Bookmaker odds, fancy markets, bet slip |
| In-Play | `/inplay` | Live matches currently accepting bets |
| My Bets | `/bets` | Bet history with status filters |
| Casino | `/casino` | All casino games (Coin Flip, Dice, Hi-Lo, Teen Patti, Indian Poker) |
| Casino Game | `/casino/[gameId]` | Play individual casino game |
| Deposit | `/deposit` | Submit deposit request |
| Withdraw | `/withdraw` | Submit withdrawal request |
| Ledger | `/ledger` | Transaction history |
| Account | `/account` | Account statement |
| Profile | `/profile` | User profile settings |
| Support | `/support` | Support tickets |

### Agent Panel (`agent.stake111.co`)

| Page | Route | Description |
|------|-------|-------------|
| Dashboard | `/agent/dashboard` | Stats overview, quick actions |
| Client Master | `/agent/clients` | All players list with lock/unlock, credit actions |
| Client Detail | `/agent/clients/[id]` | Individual player management |
| Create Player | `/agent/clients` (form) | Create new player with auto-password generation |
| Current Matches | `/agent/matches/current` | Live & upcoming matches |
| Match Detail | `/agent/matches/[id]` | Agent betting interface (bookmaker + fancy) |
| Completed Matches | `/agent/matches/completed` | Match history |
| Casino Management | `/agent/casino` | Create/toggle casino games (includes BLACKJACK) |
| Credit Management | `/agent/credits` | Transfer/deduct credits |
| Deposits | `/agent/transactions/deposits` | Approve/reject player deposits |
| Withdrawals | `/agent/transactions/withdrawals` | Approve/reject player withdrawals |
| Reports | `/agent/reports` | Financial reports, account statements |
| Ledger | `/agent/ledger` | Agent's own transaction ledger |
| User Ledger | `/agent/ledger/[userId]` | Player transaction ledger |
| Support | `/agent/support` | Support ticket management |

### Master Panel (`master.stake111.co`)

| Page | Route | Description |
|------|-------|-------------|
| Dashboard | `/master/dashboard` | Platform overview, totals, system health |
| All Admins/Agents | `/master/admins` | List all agents with status, balance, commissions |
| Create Admin | `/master/admins/create` | Create new admin/agent with hierarchy config |
| Admin Detail | `/master/admins/[id]` | Edit admin settings |
| Hierarchy Tree | `/master/admins/hierarchy` | Visual tree with role filters |
| All Matches | `/master/matches` | All matches with status filters, Import from API |
| Completed Matches | `/master/matches/completed` | Match results & settlement status |
| Settlements | `/master/settlements` | Generate, approve, pay, reject settlements |
| All Transactions | `/master/transactions` | Platform-wide transaction history |
| Deposits | `/master/deposits` | All pending deposit requests |
| Withdrawals | `/master/withdrawals` | All pending withdrawal requests |
| Reports | `/master/reports` | Financial, Users, Matches, Agents analytics |
| Audit Logs | `/master/audit-logs` | System audit trail with filters |
| Platform Settings | `/master/settings` | Maintenance mode, features, security config |
| Commission Structure | `/master/settings/commission` | Role-based & game-type commission rates |
| Betting Limits | `/master/settings/betting-limits` | Global min/max bet and payout limits |

---

## Getting Started

### Prerequisites

- Node.js 18+
- Docker & Docker Compose
- Git

### Local Development

```bash
# Clone the repository
git clone https://github.com/topdrift/stake111-roadmap.git
cd stake111-roadmap

# Install backend dependencies
cd backend
npm install
npx prisma generate
npx prisma migrate dev
npx prisma db seed
cd ..

# Install frontend dependencies
cd frontend
npm install
cd ..

# Start infrastructure (Postgres + Redis)
docker compose up -d

# Start backend (in one terminal)
cd backend
npm run dev

# Start frontend (in another terminal)
cd frontend
npm run dev
```

### Default Credentials

| Role | Username | Password |
|------|----------|----------|
| Master Admin | `admin` | `Demo@123` |
| Agent | `agent` | `Demo@123` |
| Player | `player` | `Demo@123` |

---

## Environment Variables

Create a `.env` file in the project root. See `.env.example` for a template.

### Backend Variables

```env
# Server
NODE_ENV=production
PORT=5000

# Database (PostgreSQL)
DATABASE_URL=postgresql://stake_user:YOUR_DB_PASSWORD@postgres:5432/stake111?schema=public
POSTGRES_USER=stake_user
POSTGRES_PASSWORD=YOUR_DB_PASSWORD
POSTGRES_DB=stake111

# Redis
REDIS_URL=redis://:YOUR_REDIS_PASSWORD@redis:6379
REDIS_PASSWORD=YOUR_REDIS_PASSWORD

# JWT Authentication
JWT_SECRET=your-64-char-hex-secret
REFRESH_TOKEN_SECRET=your-64-char-hex-refresh-secret
JWT_EXPIRES_IN=7d
REFRESH_TOKEN_EXPIRES_IN=30d

# CORS (comma-separated origins)
CORS_ORIGIN=https://stake111.co,https://agent.stake111.co,https://master.stake111.co

# Cricket API (optional - for live match data)
CRICKET_API_KEY=your-cricapi-key
CRICKET_API_URL=https://api.cricapi.com/v1

# Security
BCRYPT_ROUNDS=10
MAX_LOGIN_ATTEMPTS=5
LOCKOUT_DURATION=15
SESSION_TIMEOUT=30
```

### Frontend Variables

```env
# API Connection
NEXT_PUBLIC_API_URL=https://stake111.co/api/v1
NEXT_PUBLIC_SOCKET_URL=https://stake111.co
NEXT_PUBLIC_WS_URL=https://stake111.co

# App Config
NEXT_PUBLIC_APP_NAME=Stake111
NEXT_PUBLIC_CURRENCY_SYMBOL=₹
HOSTNAME=0.0.0.0
```

---

## Docker Deployment

### Production Deployment

```bash
# Build and start all services
cd /opt/stake111
docker compose -f docker-compose.prod.yml build --no-cache
docker compose -f docker-compose.prod.yml up -d

# Check status
docker compose -f docker-compose.prod.yml ps

# View logs
docker compose -f docker-compose.prod.yml logs -f backend
docker compose -f docker-compose.prod.yml logs -f frontend
```

### Services

| Service | Container | Port | Health Check |
|---------|-----------|------|-------------|
| PostgreSQL | `stake-postgres` | 5432 | `pg_isready` |
| Redis | `stake-redis` | 6379 | `redis-cli ping` |
| Backend | `stake-backend` | 5000 | `GET /health` |
| Frontend | `stake-frontend` | 3000 | `GET /` |

### Common Commands

```bash
# Rebuild single service
docker compose -f docker-compose.prod.yml build --no-cache backend
docker compose -f docker-compose.prod.yml up -d backend

# Run Prisma migrations
docker exec stake-backend npx prisma migrate deploy

# Seed the database
docker exec stake-backend npx prisma db seed

# Database shell
docker exec -it stake-postgres psql -U stake_user -d stake111

# Redis CLI
docker exec -it stake-redis redis-cli

# View backend logs
docker logs -f stake-backend --tail 100
```

### Nginx Configuration

Three server blocks route traffic by subdomain:

```
stake111.co           -> frontend:3000  (Player panel)
agent.stake111.co     -> frontend:3000  (Agent panel)
master.stake111.co    -> frontend:3000  (Master panel)
*/api/*               -> backend:5000   (API)
*/socket.io/*         -> backend:5000   (WebSocket)
```

---

## API Reference

All endpoints are prefixed with `/api/v1`. Authentication uses JWT Bearer tokens.

### Authentication (`/auth`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/auth/login` | No | Login (returns JWT + refresh token) |
| `GET` | `/auth/me` | Yes | Get current user profile |
| `POST` | `/auth/logout` | Yes | Invalidate session |

**Login Request:**
```json
{ "username": "player", "password": "Demo@123" }
```

**Login Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbG...",
    "refreshToken": "eyJhbG...",
    "user": { "id": "uuid", "username": "player", "role": "PLAYER", "balance": 50000 }
  }
}
```

### Matches (`/matches`)

| Method | Endpoint | Auth | Role | Description |
|--------|----------|------|------|-------------|
| `GET` | `/matches` | Yes | Any | List matches (filter by status) |
| `GET` | `/matches/:id` | Yes | Any | Match details with odds |
| `POST` | `/matches/sync` | Yes | Admin+ | Sync from Cricket API |
| `PUT` | `/matches/:id/settle` | Yes | Agent+ | Settle match |
| `PUT` | `/matches/:id/void` | Yes | Agent+ | Void match & refund bets |

### Player Bets (`/bets`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/bets` | Yes | Place a bet |
| `GET` | `/bets` | Yes | Get bet history |
| `GET` | `/bets/:id` | Yes | Get bet details |

**Place Bet:**
```json
{
  "matchId": "uuid",
  "betType": "MATCH_WINNER",
  "betOn": "Team A",
  "amount": 1000,
  "odds": 1.85,
  "isBack": true
}
```

### Agent Bets (`/agent-bets`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/agent-bets/bets/place` | Agent | Place bookmaker bet |
| `POST` | `/agent-bets/fancy-bets/place` | Agent | Place fancy bet |
| `GET` | `/agent-bets/bets/my-bets` | Agent | Agent bet history |
| `GET` | `/agent-bets/bets/match/:matchId` | Agent | Bets for specific match |

### Agent Management (`/agents`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/agents/create-player` | Agent | Create new player |
| `POST` | `/agents/create-agent` | Agent+ | Create sub-agent |
| `POST` | `/agents/transfer-credit` | Agent | Transfer credit to player |
| `POST` | `/agents/deduct-credit` | Agent | Deduct credit from player |
| `GET` | `/agents/players` | Agent | List agent's players |
| `GET` | `/agents/stats` | Agent | Agent statistics |
| `GET` | `/agents/hierarchy` | Agent | Hierarchy tree |

**Create Player:**
```json
{
  "username": "newplayer",
  "password": "SecurePass@123",
  "displayName": "New Player",
  "creditLimit": 50000,
  "sportShare": 100,
  "matchCommission": 2,
  "sessionCommission": 3
}
```

### Admin Operations (`/admin`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/admin/dashboard/stats` | Agent+ | Dashboard statistics |
| `GET` | `/admin/clients` | Agent+ | List all clients |
| `PUT` | `/admin/clients/:id/toggle-lock` | Agent+ | Lock/unlock client |
| `PUT` | `/admin/clients/:id/toggle-bet-lock` | Agent+ | Lock/unlock betting |
| `GET` | `/admin/matches/current` | Agent+ | Current matches |
| `PUT` | `/admin/matches/:id/odds` | Agent+ | Update match odds |
| `PUT` | `/admin/matches/:id/toggle-betting` | Agent+ | Toggle match betting |
| `GET` | `/admin/reports` | Agent+ | Financial reports |

### Master Admin (`/master`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/master/admins` | Master | List all admins/agents |
| `POST` | `/master/admins` | Master | Create admin/agent |
| `GET` | `/master/admins/hierarchy` | Master | Full hierarchy |
| `PUT` | `/master/admins/:id` | Master | Update admin |
| `DELETE` | `/master/admins/:id` | Master | Delete admin |
| `POST` | `/master/settlements/generate` | Master | Generate settlement |
| `POST` | `/master/settlements/generate-all` | Master | Generate all settlements |
| `PUT` | `/master/settlements/:id/approve` | Master | Approve settlement |
| `PUT` | `/master/settlements/:id/pay` | Master | Mark settlement paid |
| `GET` | `/master/settings` | Master | Platform settings |
| `PUT` | `/master/settings` | Master | Update settings |
| `PUT` | `/master/settings/commission` | Master | Update commission rates |
| `PUT` | `/master/settings/betting-limits` | Master | Update betting limits |
| `GET` | `/master/transactions` | Master | All transactions |
| `GET` | `/master/audit-logs` | Master | Audit log |
| `GET` | `/master/reports/financial` | Master | Financial report |
| `GET` | `/master/reports/users` | Master | Users report |
| `GET` | `/master/reports/matches` | Master | Matches report |
| `GET` | `/master/reports/agents` | Master | Agents report |
| `GET` | `/master/dashboard/stats` | Master | Dashboard stats |

### Deposits & Withdrawals

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/deposits` | Player | Create deposit request |
| `GET` | `/deposits/my` | Player | My deposits |
| `GET` | `/deposits/all` | Agent+ | All deposits |
| `PUT` | `/deposits/:id/approve` | Agent+ | Approve deposit |
| `PUT` | `/deposits/:id/reject` | Agent+ | Reject deposit |
| `POST` | `/withdrawals` | Player | Create withdrawal request |
| `GET` | `/withdrawals/my` | Player | My withdrawals |
| `GET` | `/withdrawals/all` | Agent+ | All withdrawals |
| `PUT` | `/withdrawals/:id/approve` | Agent+ | Approve withdrawal |
| `PUT` | `/withdrawals/:id/reject` | Agent+ | Reject withdrawal |

### Casino (`/casino`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/casino/games` | Any | List enabled games |
| `POST` | `/casino/play` | Player | Instant play (bet + result) |
| `POST` | `/casino/bets` | Player | Place casino bet |
| `GET` | `/casino/bets/history` | Player | Casino bet history |
| `POST` | `/casino/games` | Agent+ | Create game |
| `PUT` | `/casino/games/:id` | Agent+ | Update game |
| `PUT` | `/casino/games/:id/toggle` | Agent+ | Enable/disable game |

### Notifications (`/notifications`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/notifications` | Yes | Get notifications |
| `GET` | `/notifications/unread-count` | Yes | Unread count |
| `PUT` | `/notifications/read-all` | Yes | Mark all read |
| `POST` | `/notifications/broadcast` | Agent | Broadcast notification |

### Support (`/support`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/support/player/tickets` | Player | Create ticket |
| `GET` | `/support/player/tickets` | Player | List my tickets |
| `POST` | `/support/player/tickets/:id/reply` | Player | Reply to ticket |
| `GET` | `/support/agent/tickets` | Agent | List tickets |
| `PUT` | `/support/agent/tickets/:id/status` | Agent | Update ticket status |
| `GET` | `/support/master/tickets` | Master | All platform tickets |

### Analytics (`/analytics`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/analytics/platform` | Master | Platform stats |
| `GET` | `/analytics/today` | Agent+ | Daily summary |
| `GET` | `/analytics/revenue` | Agent+ | Revenue data |
| `GET` | `/analytics/users/top-bettors` | Agent+ | Top bettors |
| `GET` | `/analytics/matches/pnl` | Agent+ | Match P&L |
| `GET` | `/analytics/agents/performance` | Master | Agent performance |
| `GET` | `/analytics/casino` | Master | Casino analytics |

---

## Database Schema

### Core Models

| Model | Key Fields |
|-------|-----------|
| **User** | id, username, password, displayName, role, status, balance, creditLimit, agentId, userLocked, betLocked, loginAttempts, lockedUntil |
| **Agent** | id, username, password, displayName, agentType, parentAgentId, balance, creditLimit, commissionRate, totalCommission |
| **Match** | id, name, team1, team2, startTime, status, team1BackOdds, team1LayOdds, team2BackOdds, team2LayOdds, bettingLocked, matchWinner, isSettled |
| **Bet** | id, userId, matchId, betType, betOn, amount, odds, potentialWin, status, isBack |
| **agent_bets** | id, agent_id, match_id, bet_type, bet_on, amount, odds, potential_win, liability, is_back, status *(raw SQL table)* |
| **Transaction** | id, userId, agentId, type, amount, balanceBefore, balanceAfter, description |
| **Commission** | id, betId, agentId, commissionAmount, commissionRate |
| **Settlement** | id, agentId, periodStart, periodEnd, netProfit, commissionAmount, settlementAmount, status |
| **CasinoGame** | id, gameName, gameType, minBet, maxBet, rtp, houseEdge, enabled |
| **CasinoRound** | id, gameId, serverSeed, serverSeedHash, clientSeed, nonce, result, status |
| **CasinoBet** | id, userId, roundId, betType, amount, odds, status |
| **SystemSettings** | platformName, maintenanceMode, globalMinBet, globalMaxBet, commissionStructure |
| **AuditLog** | id, userId, action, resource, module, changes, ipAddress |
| **SupportTicket** | id, senderId, subject, status, priority |
| **Notification** | id, userId, title, message, type, read |

### Key Enums

| Enum | Values |
|------|--------|
| UserRole | MASTER_ADMIN, SUPER_ADMIN, ADMIN, SUPER_MASTER_AGENT, MASTER_AGENT, AGENT, PLAYER |
| AgentType | SUPER_MASTER, MASTER, AGENT |
| MatchStatus | UPCOMING, LIVE, COMPLETED, CANCELLED, POSTPONED |
| BetType | MATCH_WINNER, TOP_BATSMAN, TOP_BOWLER, TOTAL_RUNS, SESSION, FANCY, OVER_UNDER, PARLAY |
| BetStatus | PENDING, WON, LOST, CANCELLED, VOID |
| TransactionType | DEPOSIT, WITHDRAWAL, BET_PLACED, BET_WON, BET_LOST, CREDIT_TRANSFER, DEBIT_TRANSFER, COMMISSION_EARNED |
| CasinoGameType | COIN_FLIP, DICE_ROLL, HI_LO, TEEN_PATTI, INDIAN_POKER, ROULETTE, ANDAR_BAHAR |
| SettlementStatus | PENDING, APPROVED, PAID, REJECTED, ON_HOLD |

---

## Casino Engine

The casino uses a **provably fair** system with server seeds, client seeds, and nonces.

### Supported Games

| Game | Type | Bet Options | Odds |
|------|------|-------------|------|
| Coin Flip | `COIN_FLIP` | HEADS, TAILS | 1.95x |
| Dice Roll | `DICE_ROLL` | OVER, UNDER, EXACT | 1.95x / 6.0x |
| Hi-Lo | `HI_LO` | HIGH, LOW, EXACT | 1.95x / 13.0x |
| Teen Patti | `TEEN_PATTI` | PLAYER_A, PLAYER_B | 1.95x |
| Indian Poker | `INDIAN_POKER` | PLAYER_A, PLAYER_B | 1.95x |
| Blackjack | `BLACKJACK` | BLACKJACK | 2.5x |

### How Provably Fair Works

1. Server generates a **server seed** and stores its SHA256 hash
2. Player provides a **client seed** (or uses default)
3. Combined seed + nonce generates deterministic result via HMAC-SHA256
4. After the round, server reveals the seed for verification
5. Anyone can verify: `SHA256(serverSeed) === serverSeedHash`

### Casino Files
```
backend/src/services/casino/
  index.ts         -- Routes game types to engines
  gameEngine.ts    -- Base interfaces & types
  coinFlip.ts      -- Coin flip logic
  dice.ts          -- Dice roll logic
  hiLo.ts          -- Hi-Lo logic
  teenPatti.ts     -- Teen Patti + Indian Poker logic
```

---

## Cron Jobs

| Job | Schedule | File | Description |
|-----|----------|------|-------------|
| Bet Settlement | Every 5 min | `betSettlement.job.ts` | Auto-settle bets for completed matches |
| Bet Void | Every 15 min | `betSettlement.job.ts` | Void bets for cancelled matches |
| Match Sync | Every 10 min | `matchSync.job.ts` | Sync live scores from Cricket API |
| Weekly Settlement | Sunday midnight | `settlementGeneration.job.ts` | Generate weekly agent settlements |
| Monthly Settlement | 1st of month | `settlementGeneration.job.ts` | Generate monthly agent settlements |
| Notification Cleanup | Daily 3 AM | `notificationCleanup.job.ts` | Delete old read notifications |

---

## Troubleshooting

### Player Account Locked

```sql
-- Connect to database
docker exec -it stake-postgres psql -U stake_user -d stake111

-- Unlock player
UPDATE users SET "loginAttempts"=0, "lockedUntil"=NULL WHERE username='player';
```

### Reset User Password

```bash
# Use Node.js inside the backend container (avoids shell escaping issues with bcrypt $ characters)
docker exec stake-backend node -e "
const bcrypt = require('bcryptjs');
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();
async function main() {
  const hash = await bcrypt.hash('NewPassword@123', 10);
  await prisma.user.update({ where: { username: 'player' }, data: { password: hash } });
  await prisma.\$disconnect();
  console.log('Password updated');
}
main();
"
```

### Redis AOF Corruption

```bash
docker exec stake-redis redis-check-aof --fix /data/appendonlydir/appendonly.aof.1.incr.aof
docker compose -f docker-compose.prod.yml restart redis
```

### Docker Disk Space Full

```bash
docker builder prune -f
docker system prune -f
docker image prune -a -f
```

### Backend Not Starting

```bash
docker logs stake-backend --tail 50
docker exec stake-backend npx prisma db push
docker exec stake-backend npx prisma generate
```

### Useful Database Queries

```sql
-- Check user balances
SELECT username, "displayName", role, balance, "creditLimit" FROM users;

-- Check agent details
SELECT username, "displayName", "agentType", balance, "commissionRate" FROM agents;

-- Recent transactions
SELECT u.username, t.type, t.amount, t."balanceAfter", t.description
FROM transactions t JOIN users u ON t."userId" = u.id
ORDER BY t."createdAt" DESC LIMIT 20;

-- Check bets for a match
SELECT u.username, b."betType", b."betOn", b.amount, b.odds, b.status
FROM bets b JOIN users u ON b."userId" = u.id
WHERE b."matchId" = 'MATCH_UUID';
```

---

## Project Structure

```
stake111/
├── backend/
│   ├── prisma/
│   │   ├── schema.prisma          # Database schema
│   │   └── seed.ts                # Database seeder
│   ├── src/
│   │   ├── config/                # App config & logger
│   │   ├── controllers/           # 13 controllers
│   │   ├── db/                    # Prisma client & Redis
│   │   ├── jobs/                  # 4 cron job files
│   │   ├── middleware/            # Auth, validation, security, rate limiting
│   │   ├── routes/                # 13 route files
│   │   ├── services/              # 15+ services + casino engine
│   │   ├── utils/                 # JWT, password, financial helpers
│   │   └── server.ts              # Express entry point
│   ├── Dockerfile
│   └── package.json
├── frontend/
│   ├── public/                    # PWA manifest, service worker, icons
│   ├── src/
│   │   ├── app/                   # Next.js App Router
│   │   │   ├── (user)/            # 14 player routes
│   │   │   ├── agent/             # 15+ agent routes
│   │   │   └── master/            # 16 master routes
│   │   ├── components/            # UI, betting, casino, layout, admin, master
│   │   ├── hooks/                 # usePWA, useSocket
│   │   ├── lib/                   # API client, utilities
│   │   ├── services/              # 13 API services
│   │   └── store/                 # 5 Zustand stores
│   ├── Dockerfile
│   └── package.json
├── nginx/
│   └── nginx.conf                 # Reverse proxy config
├── docker-compose.yml             # Development
├── docker-compose.prod.yml        # Production
├── deploy.sh                      # Deployment script
└── .env.example                   # Environment template
```

---

## License

Private - All rights reserved.
