# MDTC Clubhouse

A responsive student club workspace for events, RSVPs, a member directory, and announcements. Supabase provides accounts and shared club data across devices; Netlify hosts the website and installable PWA.

## Run locally

Prerequisite: Node.js 20.19+ or 22.12+ (with npm).

```sh
npm install
copy .env.example .env.local
npm run dev
```

Fill in `.env.local` as described below before using registration or sign-in. To verify a production build, run `npm run build`.

## Configure Supabase

1. Create a Supabase project.
2. In the SQL Editor, run [`supabase/migrations/20261016000000_club_sharing.sql`](./supabase/migrations/20261016000000_club_sharing.sql) once. It creates the shared workspace and membership tables, row-level security policies, and the RPC functions used by the app.
3. In **Authentication → URL Configuration**, set the Site URL to your deployed site and add both the deployed URL and local development URL (for example `http://localhost:5173/**`) to the redirect URL allow list.
4. If email confirmation is enabled, configure the confirmation email redirect to return to your site. New members confirm their account and then sign in to create or join the club chosen during registration.
5. Copy the project URL and public anon/publishable key from **Project Settings → API**. Never use the `service_role` key in this client app.

Create `.env.local` from `.env.example` and set:

```env
VITE_SUPABASE_URL=https://your-project-id.supabase.co
VITE_SUPABASE_ANON_KEY=your-public-anon-or-publishable-key
```

Restart Vite after changing environment variables. Only the public anon/publishable key belongs in browser-facing configuration; RLS and the included database functions enforce access.

## Deploy to Netlify

1. Push this project to a Git provider and import the repository in Netlify.
2. Netlify uses the included [`netlify.toml`](./netlify.toml) to run `npm run build` and publish `dist`.
3. Set `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` in **Site configuration → Environment variables**. Redeploy after setting them or changing them.
4. Update Supabase's Site URL and allowed redirect URLs for the Netlify deployment URL (and any custom domain).
5. Open the deployed site, register the club owner, and share the invite code from Settings with members.

## Included features

- Sign up, sign in, sign out, and change password through Supabase Auth
- Create a shared club or register with an invite code
- Shared event calendar with per-account RSVPs
- Shared member directory, announcements, reactions, and club settings
- Realtime workspace updates and backup export/import
- Owner-only club profile/member management and workspace reset
- Responsive desktop/mobile layouts and installable PWA

## Data and migration notes

The workspace is stored as shared JSON data in Supabase; database membership records remain authoritative for account access. Concurrent edits to the shared workspace are last-write-wins, so members should avoid editing the same item at the same time. JSON backups do not contain account credentials and restoring one replaces shared club content.

The former local-only demo accounts and data are not automatically imported. Download any local backup before switching versions. Cloud-version backups can be restored with **Settings → Restore from backup**; older local-only backups may need manual conversion because RSVPs are now per-account. Clearing the site browser cache does not delete the Supabase account or workspace. Offline app-shell caching is provided, but editing requires an internet connection.
