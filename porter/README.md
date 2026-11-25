# Porter Construction Services LLC

Implementation snapshot — not cleared for production launch. Cloudflare Pages is the only deployment target. No credentials are bundled.

## Local preview

Requires Node.js 22 or compatible modern Node. No packages are needed. Extract the ZIP, open Terminal in the extracted `porter` folder and run:

```sh
npm run dev
```

Open http://127.0.0.1:4173/ and http://127.0.0.1:4173/admin/projects. The server binds to localhost only. The admin preview has no authentication and uses browser localStorage; use fictitious test submissions only. All ten provisional project records are visible here with labeled architectural SVG placeholders. Changes remain in that browser. To reset, clear site data for localhost. Stop with Ctrl+C.

## Production build

```sh
npm run lint
npm test
npm run build
```

Deploy only `dist/` produced by `npm run build` (never the preview build). This strips unverified project records and draft placeholder assets from client files. The actual Cloudflare configuration endpoint always returns preview:false. The application uses local SVG icons and the unmodified supplied PNG logo. The illustrated hero is intentionally not represented as a photograph.

## Supabase configuration (requires review and integration tests)

1. Create a Supabase project, disable public user signup in Authentication settings, and apply `supabase/migrations/001_initial.sql` exactly once using SQL editor or the CLI. This migration uses one JSONB record per entity with status, timestamps and RLS; it is a preliminary schema, not the complete normalized relational model requested.
2. Verify RLS policies with anon, unauthorized authenticated, and each role using test accounts. Do not use the service-role key in a browser. Verify the private buckets `project-media` and `private-attachments` exist and are not public.
3. Create the first administrator in Supabase Authentication > Users using a verified owner email. Copy the user's UUID and execute as the database administrator (replace the placeholder):

```sql
insert into public.admin_profiles(id,role) values ('YOUR_AUTH_USER_UUID','owner');
```

4. Provision subsequent administrator accounts in the same console, assigning one of `administrator`, `content_manager`, `recruitment_manager`, `inquiry_manager`. Revoke access by deleting the appropriate profile via SQL, not the last owner. In-app invitation, MFA, recovery, final-owner protection and account lifecycle are not implemented; use Supabase's console and review these requirements before launch.
5. Configure Cloudflare server environment variables listed in `.env.example`. URL and anonymous key are public by design; service-role, Turnstile secret and Resend key are secrets. Never prefix secret keys with `VITE_` or put them in static assets.
6. Configure Turnstile for the intended domain. Submit endpoints reject production submissions until Turnstile and Supabase are configured. They validate text fields, verify Turnstile and use a database rate-limit function. Configure and test Resend with a verified sending domain; notifications contain no submitted personal data. Email delivery failures are not retried or surfaced in the current admin interface.
7. Do not enter real applicant or client information until privacy, retention, storage upload/attachment and permissions workflows are reviewed and tested.

## GitHub and Cloudflare Pages

1. Create a private GitHub repository (draft project data is in source and migration). Push from the extracted project folder:

```sh
git init
git add .
git commit -m "Initial Porter website"
git branch -M main
git remote add origin YOUR_GITHUB_REPOSITORY_URL
git push -u origin main
```

2. Cloudflare dashboard > Workers & Pages > Create > Pages > Connect to Git. Select the GitHub repository and `main` branch. Framework preset: None. Build command: `npm run build`. Build output directory: `dist`. Root directory: repository root. Set Node version to 22 if needed. Pages Functions live in `/functions`; retain this directory in Git. Configure environment variables in the project's settings; use encrypted secrets for private keys. Use a separate staging Supabase project for preview deployments.
3. Deploy a staging branch first; verify `/api/config` returns preview:false and `/api/content` returns only published data. Check direct navigation to `/services`, `/projects` and `/admin/login`. Static SPA fallback is provided in `_redirects`; `_headers` supplies security headers for static pages, while JSON endpoints have their own headers.
4. After launch approval, attach `porterconstructions.com` in Pages > Custom domains. Follow Cloudflare DNS instructions and verify HTTPS. Preserve existing mail-related DNS records. Configure www redirect in Cloudflare Redirect Rules only after confirming the preferred canonical host. Do not modify existing DNS blindly.
5. Verify sitemap, robots, security response headers, direct URLs, form submissions, access control and owner account before enabling public traffic. Dynamic prerendering, dynamic sitemap, and social metadata are outstanding.

## Files

- `src/app.js`, `src/style.css`, `src/seed.js`: public and administration application.
- `public/assets/`: original logo, local icons, original architectural illustrations and draft placeholders.
- `functions/api/`: configuration, content, administrator and submission endpoints.
- `supabase/migrations/001_initial.sql`: preliminary schema, seeds, RLS, storage policies, audit and rate limit functions.
- `tests/`: automated static/build checks.
- `docs/STATUS.md`: implementation gaps, test results and launch gates.

## Important limitations

This is a functional-development foundation rather than the complete production-ready system specified in the original prompt. npm registry DNS was unavailable, so the frontend uses native JavaScript modules and CSS rather than React/TypeScript/Vite. No live Supabase, Cloudflare, Resend or Turnstile integration was executed. Browser navigation was blocked by managed Chromium policy. See `docs/STATUS.md` for outstanding work. Do not publish until launch gates are satisfied.
