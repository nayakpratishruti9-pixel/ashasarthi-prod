# AshaSarthi

AshaSarthi is a multilingual health and community-care management portal for ASHA workers and Anganwadi workers (AWCS) in Odisha. It provides authenticated access to supply inventory, distribution records, health-related workflows, help content, and an AI support assistant.

The application is built as an offline-first Progressive Web App (PWA). Navigation, the application shell, and data explicitly stored in the browser can remain available offline. Operations that read or write the server-side PostgreSQL database still require a network connection.

## Features

- Credential-based registration and sign-in
- Optional Google and GitHub OAuth sign-in
- Supply inventory management with low-stock thresholds
- Distribution recording for beneficiaries and wards
- Dashboard summaries and health-workflow pages
- AI help chat powered by Groq
- English, Hindi, Odia, Tamil, Telugu, Malayalam, and Spanish translations
- Installable PWA with service-worker caching in production
- IndexedDB storage for offline dashboard data, settings, and drafts
- Responsive interface for desktop and mobile devices

## Technology stack

- [Next.js](https://nextjs.org/) 16 with the App Router
- React 19 and TypeScript
- Tailwind CSS 4
- NextAuth 4
- PostgreSQL with Sequelize and `sequelize-typescript`
- `next-intl` for translations
- `@ducanh2912/next-pwa` and Workbox for PWA support
- `idb` for IndexedDB access
- Groq SDK for the AI assistant

## Prerequisites

Install the following before starting:

- Node.js 20 or newer
- npm 10 or newer
- A PostgreSQL database accessible from the application
- OAuth applications only if Google or GitHub sign-in is required
- A Groq API key only if the AI assistant is required

The required Node and npm versions are also declared in [`package.json`](./package.json).

## Getting started

### 1. Install dependencies

```bash
npm install
```

### 2. Configure environment variables

Create a `.env.local` file in the repository root. Do not commit this file or put secrets in client-side code.

```dotenv
# PostgreSQL connection
DATABASE_NAME=your_database_name
DATABASE_USERNAME=your_database_user
DATABASE_PASSWORD=your_database_password
DATABASE_HOST=your_database_host
DATABASE_PORT=6543

# NextAuth
NEXTAUTH_SECRET=replace_with_a_long_random_secret
NEXTAUTH_URL=http://localhost:3000

# Optional: use an external API base URL instead of the local /api routes.
# Leave unset for the local Next.js API routes.
NEXT_PUBLIC_APP_API_ENDPOINT=

# Optional OAuth providers
GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# Optional AI assistant
GROQ_API_KEY=
```

`DATABASE_NAME`, `DATABASE_USERNAME`, `DATABASE_PASSWORD`, and `DATABASE_HOST` are required when a database-backed route is used. The application defaults `DATABASE_PORT` to `6543`, but set it explicitly when your provider uses another port. Production database connections enable SSL.

Generate a strong `NEXTAUTH_SECRET` rather than using the example value. For example, with OpenSSL:

```bash
openssl rand -base64 32
```

### 3. Prepare the database

The application uses PostgreSQL through Sequelize models in [`backend/database/models/`](./backend/database/models/). The current code expects the database schema to already exist; it does not include a migration or seed command.

The main models are:

- `users` — account, contact, role, and password data
- `supplies` — inventory items, units, quantities, and low-stock thresholds
- `distributions` — supply distribution records
- `awcs` — Anganwadi center data

Create or migrate these tables using your PostgreSQL workflow before testing sign-up, sign-in, supplies, or distributions.

### 4. Start the development server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). PWA service-worker generation is disabled in development to avoid stale-cache issues.

### 5. Create an account

Open `/signup` and register with:

- A name between 2 and 100 characters
- A valid phone number
- A valid email address
- A password of at least 8 characters containing an uppercase letter, lowercase letter, and number
- The `asha` or `awcs` role

After registration, sign in at `/signin`. Protected pages redirect unauthenticated users to the sign-in page.

## Available commands

```bash
npm run dev      # Start the development server
npm run lint     # Run ESLint
npm run build    # Create a production build
npm start        # Start the production server
```

Run the production build locally with:

```bash
npm run build
npm start
```

Then open [http://localhost:3000](http://localhost:3000).

## Application routes

### User-facing pages

| Route | Purpose |
| --- | --- |
| `/` | Home page and supply overview |
| `/signin` | Credentials and OAuth sign-in |
| `/signup` | Account registration |
| `/dashboard` | Dashboard, distributions, and inventory summaries |
| `/supplies` | Create, edit, and delete supply items |
| `/ncd-data` | Non-communicable disease data workflow |
| `/preview` | Preview workflow |
| `/contact` | Contact information |
| `/help` | Feature help and guidance |
| `/settings` | Application settings |
| `/offline-test` | Offline storage testing page |

### API routes

| Endpoint | Methods | Purpose |
| --- | --- | --- |
| `/api/auth/[...nextauth]` | `GET`, `POST` | NextAuth sessions and providers |
| `/api/signup` | `POST` | Create a local user account |
| `/api/supplies` | `GET`, `POST`, `PUT`, `DELETE` | Manage the signed-in user's supplies |
| `/api/distributions` | `GET`, `POST` | Read and create distribution records |
| `/api/aichat` | `POST` | Send a question to the Groq support assistant |

API requests are authenticated with the NextAuth JWT. The client uses relative `/api` URLs by default; set `NEXT_PUBLIC_APP_API_ENDPOINT` only when the API is hosted separately.

## Internationalization

Translations live in [`messages/`](./messages/). Supported locales are:

| Code | Language |
| --- | --- |
| `en` | English |
| `hi` | Hindi |
| `or` | Odia |
| `ta` | Tamil |
| `te` | Telugu |
| `ml` | Malayalam |
| `es` | Spanish |

English is the default locale. The language switcher stores the selection in the `NEXT_LOCALE` cookie for one year. When adding UI text, add the corresponding key to every supported message file.

## Offline and PWA behavior

Production builds generate a service worker and cache the application shell, pages, static assets, and selected data requests. The service worker is disabled during `npm run dev`.

Browser-local data is stored in the IndexedDB database `ashasarthi-offline-db`:

- `dashboard` — locally saved dashboard entries
- `settings` — local user preferences
- `drafts` — offline form drafts

To test offline behavior:

1. Run a production build and start the server.
2. Open the app in Chrome or Edge.
3. Open DevTools and use **Application** to inspect the service worker and IndexedDB.
4. Use the **Network** tab to select **Offline**.
5. Verify local navigation and locally stored data.

The browser cache does not replace the PostgreSQL database. Server-backed authentication, AI chat, and supply/distribution API operations may fail while offline.

## Project structure

```text
backend/database/       PostgreSQL connection and Sequelize models
messages/                Translation dictionaries
public/                  PWA manifest and application icons
src/app/                 Next.js pages and API route handlers
src/components/          Shared navigation, layout, chat, and language UI
src/hooks/               Client hooks, including online-status detection
src/lib/                 API client, IndexedDB, and i18n helpers
src/types/               Shared TypeScript declarations
i18n.ts                  Locale configuration
middleware.ts            Authentication route protection
next.config.ts           Next.js, next-intl, and PWA configuration
```

## Deployment

Build and run the application on a Node.js host:

```bash
npm ci
npm run build
npm start
```

Configure the production environment variables from the setup section in the hosting provider. In particular:

- Set `NEXTAUTH_URL` to the public HTTPS URL.
- Use a unique production `NEXTAUTH_SECRET`.
- Use production PostgreSQL credentials.
- Configure OAuth callback URLs in Google/GitHub if those providers are enabled.
- Set `GROQ_API_KEY` if the AI assistant is enabled.

The repository includes [`netlify.toml`](./netlify.toml) with Netlify secret-scan configuration. Confirm that the hosting platform supports the Next.js runtime and server-side API routes before deploying; the application is not a static export because authentication and database APIs run on the server.

## Troubleshooting

### Database connection errors

Confirm all four required database variables are present and correct, that the database accepts connections from the host, and that `DATABASE_PORT` matches the provider. Restart the development server after changing `.env.local`.

### Authentication redirects to `/signin`

Confirm `NEXTAUTH_SECRET` is set and consistent across server restarts. In production, verify that `NEXTAUTH_URL` exactly matches the public URL and that cookies are not being blocked by an incorrect HTTPS/domain configuration.

### OAuth sign-in fails

Confirm the provider client ID and secret are set. Add the correct callback URL for the deployment, for example:

```text
https://your-domain.example.com/api/auth/callback/google
https://your-domain.example.com/api/auth/callback/github
```

### Changes are not visible after a deployment

Unregister the service worker in DevTools under **Application > Service Workers**, clear site data, and reload. PWA caching is intentionally disabled in development.

## Contributing

1. Create a focused branch for your change.
2. Keep server-backed and offline behavior clear and separate.
3. Update all locale files when adding translatable UI text.
4. Run `npm run lint` and `npm run build` before opening a pull request.
5. Do not commit `.env*` files, credentials, or generated local build output.

## License

No license file is currently included in the repository. Confirm the intended license with the project maintainers before distributing the application.
