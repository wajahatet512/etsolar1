# ET Group of Companies — Google-secured Admin gate

This package takes the current ET Group static website preview and adds a Vercel Node.js API login gate for the Admin demo.

## What this package does

- Keeps the six public business portals in `index.html`.
- The Admin link opens a sign-in screen instead of rendering the Admin workspace immediately.
- Shows Google's **Continue with Google** button. No separate Admin password is used.
- The backend verifies the Google ID token signature, issuer, expiry and audience with Google's official `google-auth-library`.
- Only the exact, verified `ADMIN_EMAIL` is accepted (`wajahat.et512@gmail.com` in `.env.example`).
- Creates an eight-hour HMAC-signed session in an `HttpOnly; Secure; SameSite=Lax` cookie.
- Provides session check, logout and a protected example Admin API route.
- No Google access token or ID token is stored in localStorage.

## Important security boundary

The current Admin content is still a front-end demonstration. This login gate controls entry to the Admin UI, and `api/admin/ping.js` demonstrates server-side protection. Any future real Admin data, file upload, publishing, user management, enquiry, database or payment API MUST call `requireAdmin()` (and enforce the user's division/role) on the server. Never treat hiding a client-side panel as data authorization.

## 1. Create Google OAuth Web Client ID

In Google Cloud Console:

1. Create/select a Google Cloud project.
2. Configure the OAuth consent screen and add `wajahat.et512@gmail.com` as a test user if the app remains in Testing mode.
3. Create an OAuth client ID of type **Web application**.
4. Under Authorized JavaScript origins, add the exact deployed HTTPS origin, e.g. `https://YOUR-DEPLOYMENT.vercel.app` and your final custom domain origin when ready.
5. This package uses Google Identity Services' JavaScript ID-token callback, not an authorization-code redirect; no redirect URI is used by this flow.
6. Copy the Client ID. Do not put any client secret in this project; this ID-token flow does not need one.

The production origin must be HTTPS. Google Identity Services returns a credential to the browser; the browser POSTs it to this site's API over HTTPS, and the server verifies it.

## 2. Deploy to Vercel

1. Put this folder in a GitHub repository (or import it using the Vercel CLI).
2. In Vercel, import the repository as a project. Framework preset: **Other**. No build command is required. The root directory should contain `index.html`, `api/` and `package.json`.
3. Add these Environment Variables in Vercel for Production (and Preview only if you deliberately want Admin enabled there):

   - `GOOGLE_CLIENT_ID` — Google OAuth Web Client ID.
   - `ADMIN_EMAIL` — exactly `wajahat.et512@gmail.com`.
   - `SESSION_SECRET` — a cryptographically random secret of at least 32 bytes.
   - `APP_ORIGIN` — exact deployed origin, e.g. `https://YOUR-DEPLOYMENT.vercel.app` (no path, no trailing slash).

4. Redeploy after adding/changing environment variables.
5. Add that exact `APP_ORIGIN` to the Google OAuth client's Authorized JavaScript origins.

To generate a secret locally, run `node -e "console.log(require('node:crypto').randomBytes(48).toString('base64url'))"`. Enter the output directly into Vercel's Environment Variables. Do not paste it into chat, source code or Git.

## 3. Local development

```bash
npm install
npx vercel dev
```

Create a local `.env.local` (never commit it) with the four values. For local Google testing, configure the matching localhost JavaScript origin in Google Cloud and set `APP_ORIGIN` to the exact local origin used by `vercel dev`.

## 4. Test

```bash
npm run check
npm run test:session
npm run test:auth-handlers
```

Manual tests:

- Open `/#admin` while signed out: only the Google sign-in gate should show.
- Sign in as `wajahat.et512@gmail.com`: the server should create a session and show the Admin demo.
- Sign in with another Google account: the server should reject it and not create a session.
- Call `/api/admin/ping` without a session: expect HTTP 401.
- Call `/api/admin/ping` with the approved session: expect HTTP 200.
- Sign out: the session cookie should be cleared and the Admin gate should return.
- Wait past eight hours or clear cookies: access should require sign-in again.

## 5. Before production data is added

- Implement authorization checks for every Admin API, including division scoping and role membership.
- Apply the database migrations only after DBA review and staging tests.
- Add durable rate limiting / abuse controls at Vercel Firewall or a suitable server-side store. Do not rely on per-instance in-memory counters in serverless functions.
- Add security headers/CSP and audit logging appropriate to the deployed domain.
- Consider Vercel Deployment Protection for Preview deployments separately; app-level Google sign-in is the Admin account gate.
- Ensure no private data is embedded in the public static HTML. Static HTML can be downloaded by any visitor.
