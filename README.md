# app.wshnetwork.com redirector

Firebase auth emails were sending users to `app.wshnetwork.com`, which is a
retired subdomain that no longer resolves. This is a standalone GitHub Pages
site whose only job is to redirect any request from `app.wshnetwork.com` to
the same path/query/hash on `wshnetwork.com`.

`index.html` and `404.html` are identical — `404.html` acts as the catch-all
since GitHub Pages has no server-side routing, so any path (`/verify_email`,
`/reset_password`, with all query params) gets forwarded intact via a
client-side `location.replace`.

## Setup

1. Push this folder as its own GitHub repo (public, since GitHub Pages custom
   domains require a public repo unless you're on GitHub Enterprise).
2. In the repo's Settings → Pages:
   - Source: deploy from branch, `main` / root.
   - Custom domain: `app.wshnetwork.com` (matches the `CNAME` file already
     committed here).
   - Once DNS below is in place, check "Enforce HTTPS".
3. In Google Cloud DNS (the zone serving `wshnetwork.com`), add:
   - `CNAME` record: `app` → `<github-username-or-org>.github.io.`
     (must be a CNAME, not an A record, since this is a subdomain — the
     apex uses A/AAAA records to GitHub's IPs, but `app` should point at
     the `github.io` hostname instead).
4. Wait for DNS to propagate and for GitHub to issue the Let's Encrypt cert
   for the custom domain (usually minutes, occasionally up to an hour).
5. Verify: `https://app.wshnetwork.com/verify_email?mode=verifyEmail&oobCode=test`
   should land on `https://wshnetwork.com/verify_email?mode=verifyEmail&oobCode=test`.

This is independent of whatever is happening on the Firebase side with the
action URL / authorized domains settings — it's a safety net so any stale or
slow-to-update links still land somewhere real.
