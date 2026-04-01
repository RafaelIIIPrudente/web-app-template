# 📘 Baseplate Documentation

This guide walks through setting up and running the Baseplate, which consists of:

- A Next.js frontend (stock-app)
- A NestJS backend (stock-app-api)
- Supabase for authentication, database, and storage
- Docker-based deployment

---

## ⚙️ Prerequisites

Ensure you have the following installed:

- Docker + Docker Compose
- Git
- Node.js (v22)
- pnpm (for frontend)
- NVM (optional, for managing Node versions)

---

## 🗂️ Project Structure

```
.
├── .env.api.template
├── .env.app.template
├── stock-app/          # Next.js frontend
├── stock-app-api/      # NestJS backend
├── Dockerfile.app      # Dockerfile for frontend
├── Dockerfile.api      # Dockerfile for backend
├── docker-compose.yml  # Compose config
└── README.md
```

---

## 🚀 Getting Started

### 1. Clone Repositories

```bash
git clone https://github.com/1-to-100/baseplate.git

cd baseplate/

git clone -b dev https://github.com/1-to-100/stock-app.git
cp .env.app.template ./stock-app/.env

git clone -b dev https://github.com/1-to-100/stock-app-api.git
cp .env.api.template ./stock-app-api/.env
```

### 2. Configure Supabase

This README assumes that you have setup a Supabase tenant and have a Supabase project created. If you haven't go do that first. Then:

#### 🔐 Connect to Supabase Database

1. Go to Supabase project → **Connect** (top bar)
2. Go to **ORMs** tab and select **Prisma** from Tool dropdown:
   - Copy the **Direct connection** and replace the value of your `DIRECT_URL` key in `.env` for `stock-app-api`
   - Copy the **Transaction pooler** and replace the value of your `DATABASE_URL` key in `.env` for `stock-app-api`
3. Replace `[YOUR-PASSWORD]` token for both `.env` files

#### 🌐 Connect App Frameworks

1. Go to **Settings > Data API**
2. Copy the **Project URL**:
   - Replace the value of the `SUPABASE_URL` key in `.env` for `stock-app-api`
   - Replace the value of the `NEXT_PUBLIC_SUPABASE_URL` key in `.env` for `stock-app`
3. Go to **Settings > API Keys**
4. Copy the **Legacy API Keys**:
   - `anon/public`: Replace the value of the `NEXT_PUBLIC_SUPABASE_ANON_KEY` key in `.env` for `stock-app`
   - `service_role/secret`: Replace the value of the `SUPABASE_SERVICE_ROLE_KEY` key in `.env` for `stock-app-api`

#### 🔑 JWT Secret

1. Go to **Settings > JWT Keys**
2. In the **Legacy JWT Secret** tab, click **Reveal** to show the JWT secret
3. Copy the revealed JWT secret and replace the value of your `SUPABASE_JWT_SECRET` key in `.env` for `stock-app-api`

---

## 🔐 Setting up OAuth in Supabase

Supabase provides easy OAuth integration under **Authentication > Providers**.

### 🌐 Configure URL Settings

1. Go to **Authentication > URL Configuration**
2. Set your **Site URL**:
   - For local development: `http://localhost:3000`
   - For production: `https://yourdomain.com`
3. Add **Redirect URLs**:
   - For local development: `http://localhost:3000/**`
   - For production: `https://yourdomain.com/**`
4. Click **Save changes**

> **Important:** These URLs must match your application domain and the redirect URLs you configure in your OAuth providers.

### ✅ Google OAuth

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project
3. Complete the wizard to setup the OAuth basics
4. Activate the "Create OAuth Client" — Select **Web Application** for application type
5. Add the Supabase callback URL under authorized redirect URIs (from **Project Settings -> Authentication -> Google -> Callback URL**)
6. Navigate to **APIs & Services > Credentials** and copy **Client ID** and **Client Secret**
7. In Supabase: Go to **Authentication > Providers > Google**, paste credentials, and enable

[Supabase Doc](https://supabase.com/docs/guides/auth/social-login/auth-google)

### ✅ LinkedIn OAuth

1. Go to [LinkedIn Developer Portal](https://www.linkedin.com/developers/) and create an app
2. Under **Auth > Redirect URLs**, add: `https://<your-supabase-project-ref>.supabase.co/auth/v1/callback`
3. Copy **Client ID** and **Client Secret**
4. In Supabase: Go to **Authentication > Providers > LinkedIn**, paste credentials, and enable

[Supabase Doc](https://supabase.com/docs/guides/auth/social-login/auth-linkedin)

### ✅ Microsoft OAuth

1. Go to [Azure Portal](https://portal.azure.com) then **Azure Active Directory > App Registrations**
2. Register a new app with redirect URI: `https://<your-supabase-project-ref>.supabase.co/auth/v1/callback`
3. Go to **Certificates & Secrets** to create a new secret
4. Copy **Application (client) ID** and secret
5. In Supabase: Go to **Authentication > Providers > Microsoft**, paste credentials, and enable

[Supabase Doc](https://supabase.com/docs/guides/auth/social-login/auth-azure)

---

## 🐳 Running with Docker

```bash
docker compose up
```

Then run database migration:

```bash
docker compose exec stock-app-api npx prisma migrate deploy
```

Services will be available at:

- **Frontend:** http://localhost:3000
- **Backend API:** http://localhost:3001

---

## 🧪 Running Without Docker

### 1. Install Node.js 22 using NVM

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc  # or ~/.zshrc
nvm install 22
nvm use 22
```

### 2. Install pnpm

```bash
npm install -g pnpm
```

### 3. Start Frontend

```bash
cd stock-app
pnpm install
pnpm dev
```

### 4. Start Backend

```bash
cd stock-app-api
npm install
npm run prisma:generate
npx prisma migrate deploy
npm run start:dev
# Alternative: npm run start:dev:env (if start:dev does not work on your device)
```

---

## 🌱 Data Seeding & Cleanup

### Docker

```bash
# Seed test data
docker compose exec stock-app-api npm run docker:seed

# Cleanup test data
docker compose exec stock-app-api npm run docker:cleanup
```

### Non-Docker

```bash
# Seed test data
cd stock-app-api
npm run cli:seed

# Cleanup test data
cd stock-app-api
npm run cli:cleanup
```

---

## 📘 API Documentation

The backend provides Swagger documentation.

After running the backend, access Swagger UI at: http://localhost:3001/api

To rebuild the docs:

```bash
cd stock-app-api
npm run build
```

---

## 📤 Custom SMTP for Supabase Emails

By default, Supabase sends transactional emails using its built-in service. To use your own SMTP server:

### 🛠️ Steps to Enable Custom SMTP

1. Open your Supabase project
2. Go to **Authentication → Emails → SMTP Settings**
3. Toggle **Enable custom SMTP**
4. Fill in the configuration fields:

| Field | Value |
|-------|-------|
| Sender Email | Your "From" email address |
| Sender Name | Display name for the sender |
| Host | `smtp.gmail.com` |
| Port | `587` |
| Username | Your Gmail address |
| Password | App Password (see below) |

### 🔑 Generating a Gmail App Password

1. Go to [Google Account Security Settings](https://myaccount.google.com/security)
2. Open **App Passwords**
3. Generate a new password and copy the 16-character value
4. Paste it into the **Password** field in Supabase SMTP settings

---

## 👤 Setup a Super User Account

1. Run the app and log in (user/pass or federated login)
2. Log out — you should now have a record in the users table
3. Go to Supabase → **[Your Project] → Database → Tables → Users**
4. Select the three vertical dots → **"View In Table Editor"**
5. Find your user and set `is_superadmin = true`
6. Log back in to access the extended system management options

---

## 🖥️ Setting up Cursor

Baseplate uses Cursor as our IDE of choice.

1. Download Cursor at https://cursor.com/download
2. Set up a Cursor account or be added to a pre-existing team
3. Set up Supabase as an MCP for Cursor via https://supabase.com/docs/guides/getting-started/mcp and press **"Add to Cursor"**
4. In Cursor: go to **Settings → Cursor Settings → Background Agents**
   - Connect your GitHub and Slack accounts for git commands and Slack integration
5. Go to **Settings → Cursor Settings → Indexing and Docs**
   - Add the developer guide URL: https://1to100.com/baseplate/developer-guide/
