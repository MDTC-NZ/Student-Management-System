# MDTC Social Club

A responsive social club app for planning get-togethers, RSVPing, getting to know members, and sharing updates. The site is published at `mdtc.kiaoragaming.xyz`; Supabase provides accounts and shared club data.

The domain is served by iFastNet static hosting. Accounts and shared club records use Supabase; this repository does not include a self-hosted authentication/database server.

## Development

Prerequisite: Node.js 20.19+ or 22.12+ (with npm).

```powershell
npm install
Copy-Item .env.example .env.local
npm run dev
```

Fill in `.env.local` as described below before using registration or sign-in. `npm run dev` is for development only; the public website is hosted by iFastNet at `https://mdtc.kiaoragaming.xyz`.

## Publish to iFastNet

1. Run `npm run build` to create the production files in `dist` (or use the prepared `mdtc-site-upload.zip`).
2. In iFastNet File Manager, back up the website folder and upload/extract the **contents of `dist`** (or the prepared zip) into the document root for `mdtc.kiaoragaming.xyz` (usually `public_html`). `index.html`, `assets`, `manifest.webmanifest`, `sw.js`, and `app-icon.svg` must be directly inside the document root. Do not upload the project root or its development `index.html`: that file points to `/src/main.tsx`, which is not a production JavaScript bundle.
3. In iFastNet's DNS Zone Editor, make sure the `mdtc` host points to the iFastNet web server using the destination/IP supplied in the hosting control panel. Do not use a Cloudflare tunnel or a Cloudflare proxy.
4. If removing Cloudflare DNS entirely, first copy the active DNS records from Cloudflare into iFastNet's DNS Zone Editor. Preserve all email-related MX, SPF, DKIM, DMARC, and mail-host records. Then change the domain's nameservers at the registrar to the exact iFastNet nameserver pair shown in the iFastNet account. A nameserver change replaces the active DNS zone; missing mail records can interrupt email.
5. Wait for DNS propagation, then open `https://mdtc.kiaoragaming.xyz`. If an old installed PWA is shown, unregister its service worker or clear that site's cached data and reload.

The website is served by iFastNet and remains available without your computer running. Supabase separately provides sign-in and shared club data.

## Configure Supabase

1. Create a Supabase project.
2. In the SQL Editor, run [`supabase/migrations/20261016000000_club_sharing.sql`](./supabase/migrations/20261016000000_club_sharing.sql) once. It creates the shared workspace and membership tables, row-level security policies, and the RPC functions used by the app.
   For an existing project with a 36-character invite code, also run [`supabase/migrations/20261019001000_case_insensitive_club_invites.sql`](./supabase/migrations/20261019001000_case_insensitive_club_invites.sql). This updates the join function to accept codes regardless of letter case.
   Run [`supabase/migrations/20261019002000_member_status.sql`](./supabase/migrations/20261019002000_member_status.sql) to enable member approval and active/inactive status management. Run [`supabase/migrations/20261020000000_custom_club_roles.sql`](./supabase/migrations/20261020000000_custom_club_roles.sql) to enable club positions and their permissions. Run [`supabase/migrations/20261021000000_club_chat.sql`](./supabase/migrations/20261021000000_club_chat.sql) to create persistent, realtime club chat, then run [`supabase/migrations/20261022000000_club_finances.sql`](./supabase/migrations/20261022000000_club_finances.sql) to create the secured finance ledger, budgets, member dues, and private receipt storage. Run these migrations before deploying the updated site and `admin-accounts` function. Pending or inactive memberships cannot access the club workspace.
3. In **Authentication → URL Configuration**, set the Site URL to `https://mdtc.kiaoragaming.xyz` and add `https://mdtc.kiaoragaming.xyz/**` plus the local development URL (for example `http://localhost:5173/**`) to the redirect URL allow list.
4. To automatically confirm every new email address, open **Authentication → Providers → Email** and turn off **Confirm email**. Supabase then creates a signed-in session immediately after registration, and this app opens the new account's club setup. This deliberately skips proof that the person controls the email address; users can register with an address belonging to someone else. For safer production accounts, keep confirmation enabled and configure the email templates/SMTP instead.
5. Leave Supabase's **Reset Password** email template enabled; its button/link must use `{{ .ConfirmationURL }}` so it preserves the redirect sent by the app.
6. Copy the project URL and public anon/publishable key from **Project Settings → API**. Never use the `service_role` key in this client app.

Create `.env.local` from `.env.example` and set:

```env
VITE_SUPABASE_URL=https://your-project-id.supabase.co
VITE_SUPABASE_ANON_KEY=your-public-anon-or-publishable-key
VITE_PUBLIC_SITE_URL=https://mdtc.kiaoragaming.xyz
```

`VITE_PUBLIC_SITE_URL` ensures account confirmation and password recovery redirect to the public site even when the request started at localhost. The Supabase **Reset Password** email template must use `{{ .ConfirmationURL }}`, not `{{ .SiteURL }}`, so the redirect selected by the app is honored.

### Club administrator

The account that creates a club is its administrator. The signup form defaults the club name to **Manukau Digital Tech Collective**. Administrators manage the club profile, events, announcements, member directory, and shared workspace; members can view club content and RSVP to events. To apply the administrator-only data protections to an existing Supabase project, run [`supabase/migrations/20261019000000_admin_managed_social_club.sql`](./supabase/migrations/20261019000000_admin_managed_social_club.sql) in the SQL Editor. To grant the existing Manukau Digital Tech Collective club to `mdtc@kiaoragaming.xyz`, create or invite that account in **Authentication → Users** first, then run [`supabase/setup/promote_mdtc_club_admin.sql`](./supabase/setup/promote_mdtc_club_admin.sql) in the SQL Editor. The script only assigns club-owner permissions for that existing club; it does not create a login or grant access to other clubs.

Club admins and members with **Create roles and assign positions** permission can manage positions in **Settings → Club roles & permissions**. The initial positions are President (all permissions), Vice-President (events and announcements), Secretary (announcements), Treasurer (financials), Events Coordinator (events), and Technical Officer (member directory). Assign a position to an account from the **Members** directory; the superadmin account panel also has a position selector. These position permissions are enforced by the database as well as the interface. The separate **Club admin / Member** access role remains unchanged.

Members can chat in **General**, **Events**, and **Announcements** from the **Chat** page. Messages are stored in Supabase and updates appear live. Authors can delete their own messages; club admins can delete any message. Chat access is limited to active members of the same club.

The **Finance** page records income and expenses in NZD, calculates the balance from approved entries, tracks budgets against actual spending, stores private receipt uploads, and exports approved ledger entries as CSV. Active members can view the ledger and submit transactions for review. Club admins and members with the **Financials** permission (Treasurer by default) can approve or decline requests and manage entries and budgets. The member-dues tab can prepare monthly dues for active member accounts; members report payments and an authorized reviewer confirms them. Confirmed dues are added to the income ledger. This feature tracks records and approvals; it does not process payments or connect to a bank.

The finance tables are stored separately from the JSON workspace backup. Use **Export approved entries** on the Finance page for a CSV ledger report; it does not include pending requests, budgets, or dues.

#### Superadmin account management

The `mdtc@kiaoragaming.xyz` account can manage only accounts linked to Manukau Digital Tech Collective. The **Settings → Superadmin · account management** panel supports viewing/searching accounts, editing display names and emails, manually confirming email addresses, sending password-reset emails, suspending/reactivating accounts, deleting accounts, changing the separate Club admin/Member access role, assigning club positions, and setting membership status to pending, active, or inactive. Pending and inactive memberships are blocked from the club workspace; activating a membership restores access. Confirm an email only after independently verifying that the person controls it; confirmation marks the address as verified and does not send an email. Passwords cannot be viewed. Changing an email may require confirmation from the user. The superadmin account cannot be deleted, suspended, self-confirmed, have its membership deactivated, or be demoted; the club must retain at least one active administrator. Supabase bans prevent future sign-ins/token refreshes, but an already-issued access token may remain usable until it expires.

This panel requires the trusted Supabase Edge Function; the browser never receives the service-role key:

1. From a terminal with the Supabase CLI, log in and link the project:

   ```powershell
   npx supabase login
   npx supabase link --project-ref YOUR_PROJECT_REF
   ```

2. Set the superadmin identity and site URL as Edge Function secrets:

   ```powershell
   npx supabase secrets set SUPERADMIN_EMAIL=mdtc@kiaoragaming.xyz PUBLIC_SITE_URL=https://mdtc.kiaoragaming.xyz
   ```

   Supabase supplies the project URL, anon key, and service-role key to Edge Functions. Never put the service-role key in the site, repository, or chat.
3. Deploy [`supabase/functions/admin-accounts/index.ts`](./supabase/functions/admin-accounts/index.ts):

   ```powershell
   npx supabase functions deploy admin-accounts
   ```

4. Build and upload the latest site package as described under **Publish to iFastNet**. Sign in as `mdtc@kiaoragaming.xyz` and open **Settings** to use the account panel. The function looks up the unique club workspace by name; `MDTC_WORKSPACE_ID` is not required.

Restart/rebuild after changing environment variables. Only the public anon/publishable key belongs in browser-facing configuration; RLS and the included database functions enforce access.

## Included features

- Sign up, sign in, sign out, and change password through Supabase Auth
- Request a password-reset email and set a new password from its secure recovery link
- Create a shared social club or join an existing account with an invite code
- Social calendar for meetups, outings, meals, and game nights with per-account RSVPs
- Shared member directory, announcements, reactions, and club settings
- Persistent, realtime chat with General, Events, and Announcements channels
- Finance ledger, budgets, private receipts, CSV reports, and member dues
- Realtime workspace updates and backup export/import
- Owner-only club profile/member management and workspace reset
- Responsive desktop/mobile layouts and installable PWA

## Data and migration notes

The workspace is stored as shared JSON data in Supabase; database membership records remain authoritative for account access. Concurrent edits to the shared workspace are last-write-wins, so members should avoid editing the same item at the same time. JSON backups do not contain account credentials and restoring one replaces shared club content.

The former local-only demo accounts and data are not automatically imported. Download any local backup before switching versions. Cloud-version backups can be restored with **Settings → Restore from backup**; older local-only backups may need manual conversion because RSVPs are now per-account. Clearing the site browser cache does not delete the Supabase account or workspace. Offline app-shell caching is provided, but editing requires an internet connection.
