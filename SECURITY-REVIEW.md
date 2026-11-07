# Security review notes

## Implemented in this starter

- Google's signed ID token is verified on the Node.js server with the configured OAuth client ID as audience.
- Google token signature, issuer, audience and expiration are validated by `google-auth-library`.
- The server accepts only the one exact configured email and requires `email_verified === true`.
- The server checks the exact configured request Origin for login/logout POST requests.
- A separate HMAC-signed eight-hour application session is held in an HttpOnly, Secure, SameSite=Lax cookie.
- Session signatures are compared with `timingSafeEqual`; expiry, issued-at time, subject and email are checked.
- Session endpoints return no-store headers. Logout clears the cookie.
- The Admin UI remains hidden until the server session endpoint confirms authentication.
- An example protected API route demonstrates server-side `requireAdmin` enforcement.

## Remaining before real production administration

- Add `requireAdmin` and role/division authorization to every real Admin API route.
- No real Admin data may be embedded in the public HTML; static files are downloadable.
- Implement durable rate limiting for authentication and sensitive routes using an appropriate edge/WAF or persistent store. Serverless in-memory state is not a reliable rate limiter.
- Add audit logging for privileged actions and session revocation support if immediate revocation is required.
- Configure a restrictive Content Security Policy after checking Google Identity Services requirements and all page assets.
- Add monitoring/alerting for authentication failures and deployment changes.
- Apply database migrations only after a staging test and security review.

This package is a deployment starter, not a claim of a completed production security audit.
