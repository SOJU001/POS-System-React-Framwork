# POS-System-React-Framwork

# SmartPOS — Point of Sale & Management System

A full-featured retail POS system: POS terminal, products, inventory, sales, purchases, customers, suppliers, expenses, returns, users, dashboard analytics, Telegram alarms, and AI insights.

## Architecture

- **Frontend:** React 19 + Vite + Tailwind CSS (in `src/`)
- **Backend:** Express server (`server.ts`) serving the app + REST API
- **Database:** **MySQL 8+** (local or remote). All data is persisted in MySQL — there is no in-memory or mock storage at runtime. The demo data is auto-seeded into MySQL on the first run.

## Prerequisites

- Node.js 18+
- MySQL 8+ running locally (XAMPP, WAMP, MySQL Workbench, or standalone)

## Setup

1. Install dependencies:

   `npm install`

2. Create a `.env` file (copy from `.env.example`) and set your MySQL credentials:

   ```
   MYSQL_HOST=127.0.0.1
   MYSQL_PORT=3306
   MYSQL_USER=root
   MYSQL_PASSWORD=yourpassword
   MYSQL_DATABASE=smartpos_db
   ```

   > If your root user has an empty password (default XAMPP), leave `MYSQL_PASSWORD=` empty.

3. (Optional) Telegram alarm bot:

   ```
   TELEGRAM_BOT_TOKEN=
   TELEGRAM_CHAT_ID=
   ```

4. Run the app:

   `npm run dev`

On startup the server:

- connects to your MySQL server and creates the `smartpos_db` database (if missing) with all tables,
- **seeds the demo data once** (products, categories, customers, sales, ...) only if a table is empty,
- serves the app at http://127.0.0.1:3000.

## Database Migration

Your data is stored in MySQL. When you update the app to a newer version, the schema is kept **up to date automatically**:

- Every server start runs lightweight migrations for older installs (for example, it adds the `khr_exchange_rate` column to the `settings` table if it's missing).
- `CREATE TABLE IF NOT EXISTS` + seed-only-if-empty means **your existing data is never overwritten**.
- You can also run the migration manually at any time (safe, idempotent):

  `npm run migrate`

  This connects to MySQL, creates any missing tables, applies pending column migrations, and seeds demo data only into empty tables.

  > If you get a migration error, make sure MySQL is running and your `MYSQL_*` values in `.env` are correct.

## MySQL in Settings

The **Settings → Local & Remote MySQL Database Connection** panel lets you:

- **Test Connection** — performs a real connection/ping to your MySQL server.
- **Sync MySQL Schema** — (re)creates the schema and seeds demo data if a table is empty.
- **Download .sql Export** — dumps current data as a portable SQL script.
- **Backup / Restore** — JSON snapshot backup of all tables.

## Commands

| Command | Description |
| --- | --- |
| `npm run dev` | Start dev server (Express + Vite, port 3000) |
| `npm run build` | Production build (client + bundled server) |
| `npm start` | Run the production build |
| `npm run migrate` | Create/sync MySQL schema, apply pending migrations, seed empty tables |
| `npm run lint` | TypeScript typecheck (`tsc --noEmit`) |

## Login & Security

Login is now secured:

- **Hashed passwords** — all passwords are stored as bcrypt hashes and verified on sign-in (no more "any password" demo behavior).
- **Two-Factor Authentication (2FA)** — any user can enable TOTP-based 2FA from **Settings → Two-Factor Authentication** (scan the QR code with Google Authenticator / Authy). Once enabled, sign-in requires your password plus a 6-digit code.
- **Session persistence** — after signing in you stay logged in until you log out (session is stored in the browser).

Default passwords for the seeded demo accounts (change them after first login!):

| Role | Email | Default password |
| --- | --- | --- |
| Admin | `admin@smartpos.com` | `Admin@123` |
| Manager | `manager@smartpos.com` | `Manager@123` |
| Cashier | `cashier@smartpos.com` | `Cashier@123` |

> On upgrade, existing installs are migrated automatically: the 2FA columns are added and legacy users without a password are assigned the role-based default above.

### Google Sign-In (optional)

1. Go to the [Google Cloud Console](https://console.cloud.google.com/) → **APIs & Services** → **Credentials** → **Create Credentials** → **OAuth client ID** (type: *Web application*).
2. Under **Authorized JavaScript origins**, add `http://localhost:3000` (and your production URL).
3. Copy the Client ID into **both** of these lines in your `.env`:
   ```
   GOOGLE_CLIENT_ID=your_client_id.apps.googleusercontent.com
   VITE_GOOGLE_CLIENT_ID=your_client_id.apps.googleusercontent.com
   ```
4. Restart the server (`npm run dev`). The **Sign in with Google** button appears on the login page.

Google sign-in only works for **emails that already exist in the users table** — the account's role and name are used from SmartPOS. If an email is not a staff member, sign-in is rejected.
