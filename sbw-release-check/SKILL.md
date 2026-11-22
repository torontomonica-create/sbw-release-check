---
name: "sbw-release-check"
description: "Run Small Batch Works' pre-release security and compliance audit (read-only report, one READY verdict) on a web app, PWA or native app before it ships or before a major change goes live."
---

# Small Batch Works — Release Check

Miyoung's standard for every application Small Batch Works builds. Two parts, one verdict:

- **Part I — Security** (her Universal Application Security Audit)
- **Part II — Compliance** (her Claude Compliance & Security Guide, with the Saskatchewan corrections below taking precedence)

The full source text lives in the claude.ai project doc `claude/security-audit-standard.md`. If that doc is reachable in this session, read it and follow it; this skill is the working version of it.

Miyoung is a UI/UX designer, not a career developer. Write the report so she can act on it: plain words, what it means, what to do, how urgent.

## Hard rules (audit-only)

1. Do NOT modify, delete or create files in the project. Do NOT fix anything automatically.
2. Do NOT change database settings, permissions, hosting or production configuration.
3. Never print secret values. Name the file path and variable only.
4. No destructive tests, no tests against live customer or resident data, no real transactions. Anything that writes data needs an isolated test environment and test accounts.
5. Frontend restrictions are not security. Verify server-side and database authorization wherever possible.
6. Never mark a check as passed without evidence. Never label an uninspected area SAFE.
7. Separate Confirmed issues from Potential ones, and tests actually run from code-review-only findings.
8. When something can't be verified, say exactly what access or test is needed.
9. Fixes go in the report as proposals only. Wait for Miyoung's review and explicit approval before changing anything.
10. Not legal advice. Where shipping depends on a legal question, say to confirm with the client, the regulator or a lawyer.

## Step 0 — Set up

- **Independence:** if this same session built the app, say so and recommend running the check in a fresh session so the work isn't grading itself. Continue only if Miyoung wants to.
- **Ask (one short message) only what the code can't tell you:**
  - Who is the client: private business, town/city, church/club/charity, or Small Batch Works itself? (Decides the privacy law.)
  - Does it take real data from real people yet, or is it a fake-data prototype?
  - Can she share Supabase / Netlify access or screenshots for the checks code can't show?
- **Tier:**
  - *Full check:* any tool taking real sign-ups, applications, orders, logins, uploads, payments or sending email/SMS.
  - *Light check:* fake-data prototypes and static pages with no forms. Run Step 1, Part I sections 1, 8, 11, 12, and Part II core rules. Mark the rest Not Applicable with the reason.

## Step 1 — Architecture discovery

Inspect the project before assuming anything (web vs native). Identify: app type and platforms; languages and frameworks; frontend/backend structure; database; auth provider; APIs and external services; file storage; third-party integrations; hosting and deployment; build and release config. Note whether it uses Supabase, Firebase, Netlify, Vercel, Cloudflare, Resend, Google Maps, Stripe, custom servers or other services.

Write a short architecture summary. Classify every checklist item as Applicable / Not Applicable / Requires Further Verification. Don't report irrelevant checks as vulnerabilities.

Set the **Part II profile**: Region = Canada (Global/US/EU only if the tool targets users outside Canada; South Korea off). Platform = Web unless native. Feature flags as found: Paid, Tracking/Ads, Marketing email/SMS, UGC, AI features.

## Look here first (most common risks in how Small Batch Works builds)

- Supabase `service_role` key or any secret in frontend code, or in an env variable with a public prefix (`PUBLIC_`, `VITE_`, `NEXT_PUBLIC_`): anything with that prefix ends up in the browser.
- Supabase tables with RLS off, or policies with `USING (true)` / `WITH CHECK (true)` exposing sign-up or applicant data.
- Admin/staff pages protected only by a hidden URL or a password checked in the browser.
- Public forms (sign-ups, vendor applications, contact) with no spam/rate protection, especially when they trigger emails.
- Residents' or customers' names, addresses or phone numbers on public pages or maps without their agreement.
- Storage buckets (e.g. photo uploads) readable or deletable more broadly than intended; orphaned files left after a row is deleted.

## Part I — Security checks

**1. Secrets & env vars:** hardcoded keys/passwords/tokens/private keys; exposed Supabase service_role or Firebase Admin credentials; sensitive env vars reaching client code; `.gitignore` covers env and config files; git history for past secrets; secrets in logs, errors, build artifacts, source maps; production secrets stored securely; dev/prod credentials separated.

**2. Authentication & sessions:** protected resources require auth; login, logout, reset, email verification flows; session expiry and refresh; revoked/expired tokens rejected; password storage and auth config; tokens not in URLs or logs; account recovery abuse; MFA where appropriate.

**3. Authorization:** enforced at server/database level; no access to other users' data; IDOR and parameter tampering; role-based access; no self-escalation; admin rights not grantable by ordinary users; every sensitive operation checked, not just page access.

**4. API & backend:** list public endpoints; auth on sensitive ones; SQL/NoSQL/command injection; parameterized queries; server-side validation; rate limiting; CORS; error messages leaking internals; edge/serverless functions and jobs for permission bypass; SSRF where URLs are fetched server-side; sensitive actions not callable directly without authorization.

**5. Database:** inventory tables/collections and sensitive records; RLS on every relevant table; overly permissive policies; anonymous permissions; functions, triggers, RPCs; `SECURITY DEFINER` functions and search paths; grants and default privileges; no cross-user read/modify/delete; backups and exports protected.

**6. Personal data:** what is collected and whether all of it is needed; access restrictions; PII in logs or errors; retention and deletion (including related files); no real customer data in test data; third parties receiving PII unnecessarily.

**7. File uploads & storage:** size/extension/MIME limits; content validated, not just extension; overwrite risk; bucket policies; private files not reachable without auth; path traversal and filename tricks; executable/script content; public URLs not exposing sensitive files.

**8. Dependencies:** manifests and lockfiles; `npm audit` (or equivalent); outdated/abandoned packages; unnecessary dependencies; third-party SDKs; critical/high CVEs; no dev tools in production builds.

**9. Logging & monitoring:** no sensitive data in logs; auth failures and suspicious activity visible; admin actions logged; logs not publicly accessible; errors don't expose credentials or internals; missing monitoring on sensitive operations.

**10. Abuse & cost:** automated abuse of public endpoints; rate limits on auth, forms, APIs, expensive calls; duplicate submissions; abuse of paid services (email, SMS, AI APIs, Maps, functions); resource exhaustion; idempotency for payments.

**11. Browser (web):** XSS; `innerHTML` / `dangerouslySetInnerHTML` / unsafe HTML; CSRF with cookie auth; CSP; headers (HSTS, X-Content-Type-Options, CSP, Referrer-Policy, Permissions-Policy); cookie flags (HttpOnly, Secure, SameSite); sensitive data in localStorage/sessionStorage; source maps; open redirects; risky third-party scripts.

**12. Web deployment:** HTTPS enforced; prod vs dev config; debug tools and dev endpoints off; Netlify/Vercel/Cloudflare config; env var exposure in build settings; deploy previews can't reach production data; DNS/domain where relevant.

**13. PWA:** service worker scope and updates; no sensitive data in caches; private API responses not cached publicly; offline access to cached private data; stale vulnerable cached assets.

**14–19. Native (only if native):** plaintext tokens/PII on device vs Keychain/Keystore/secure storage; local DB encryption; logs, temp files, caches; data cleared on logout. HTTPS only, certificate validation not disabled, pinning done properly. Permissions minimal, requested in context, denial handled, background location justified. Secrets extractable from binaries; debug off in release; obfuscation where useful; no security decisions only in the client. Deep links/URL schemes/App Links validated, no auth bypass, exported Android components, WebView bridges. AndroidManifest, network security config, backup rules, signing; Info.plist, ATS, associated domains, Keychain groups, entitlements, provisioning.

**Payments (only if money moves):** amounts calculated server-side; price tampering; webhook signatures verified; confirmations unforgeable; replay/duplicate protection; no unnecessary card data stored; refund/cancel authorization; payment status not user-editable; financial records protected. No real transactions.

**20–21. Release readiness:** production credentials and env; no test config active; production DB permissions; unnecessary public endpoints off; deploy access control; backups and recovery; no sensitive files in release artifacts; build scripts free of surprises; CI/CD permissions; deploy credentials protected; trusted dependencies; signing and artifact integrity; who can publish releases.

**Safe tests to run where possible:** dependency scan, secret scan (e.g. Gitleaks) including git history, static analysis, RLS policy review, unauthenticated access to protected resources, ordinary user vs admin functions with isolated test accounts. Never attack external systems or disable protections to make testing easier.

## Part II — Compliance checks

**Core (always):** no hardcoded secrets; server-side authN/authZ; no PII in logs; dependency licenses (warn on GPL/AGPL); data minimization (no unnecessary form fields); consent checkboxes default unchecked; self-service account and data deletion if accounts exist (for no-account tools: a clear way to ask for removal); no fake reviews/testimonials or unsupported claims, placeholder copy marked; no hidden or obstructed cancel/reject/opt-out; fonts and icons commercially licensed (Google Fonts OFL, Lucide).

**Saskatchewan corrections (take precedence over the original guide):**

- **Privacy law follows the client:**
  - Private business (incl. fee-charging daycare, even non-profit): PIPEDA. Needs a privacy notice: purpose, retention, third parties (incl. services outside Canada), and a privacy contact.
  - Town or city: LA FOIP. Small Batch Works is likely the town's information management service provider; flag that the town should confirm whether a written agreement is needed (what data, where stored, who sees it, when deleted) and whether storage outside Canada is acceptable.
  - Church, club, charity: usually outside PIPEDA unless commercial (e.g. selling/sharing member or donor lists). Follow PIPEDA-style practice anyway.
  - Health information: avoid. If present, stop and flag HIPA.
  - Quebec Law 25: only if handling Quebec residents' data.
- **Accessibility:** ACA (federal) and AODA (Ontario) don't apply to Saskatchewan small businesses. The Accessible Saskatchewan Act requires listed public sector bodies (incl. larger municipalities) to publish accessibility plans; no web standard set yet. Human Rights Code duty to accommodate applies to all. Check WCAG 2.1 AA anyway as Small Batch Works' quality bar (alt text, 4.5:1 contrast, full keyboard use, visible focus) and report it as quality, not law, unless the client is bound by a standard.
- **Breaches:** for PIPEDA clients, every breach of personal information must be recorded and serious ones reported and notified. Check that a short incident plan exists (who tells the client, how to switch the tool off, how to see what was exposed).
- **CASL:** marketing email/SMS needs consent (express opt-in is the safe default), sender identification and a working unsubscribe; record consent with a timestamp. Requested confirmation emails aren't marketing; flag any promotional content added to them.
- **Raffles/lotteries:** any real-money raffle needs an SLGA licence, and online ticket sales need an SLGA-approved, certified Electronic Raffle System from a registered supplier (reported to require Saskatchewan hosting; confirm with SLGA). A self-built real-money raffle is CRITICAL; demo-only with simulated payments is fine.
- **Tracking:** if analytics/ads run (e.g. GA4), the privacy notice must say so. A cookie consent banner is required only for the Global/EU profile; for Canada, disclosure plus ad features off is the baseline.
- **Demos:** fake data must be clearly a demo; never real residents' or customers' data in a prototype.

**Only if the matching profile item is on:**
- Global/US/EU: cookie banner with equal Accept/Reject, trackers blocked until consent, Cookie Policy; under-13 age gate unless built for children; "Do Not Sell or Share" link if data is shared/sold; WCAG 2.1 AA.
- Web: legal footer links (Privacy; Terms; Refund if paid; Cookie Policy if tracking under Global/EU).
- Native: in-app account deletion; iOS ATT before tracking SDKs; accessibilityLabel/Role; IAP rules with price, renewal and cancellation shown and a Restore Purchases button.
- Paid: full price incl. taxes/fees before checkout, refund policy, easy cancellation.
- UGC: Report and Block on every screen showing user content.
- AI features: disclaimer near AI output that it may be inaccurate.

## Report

Write the report in plain English. Sections:

**A. Architecture summary** — type, platforms, frameworks, backend/database, auth, hosting; client type and Part II profile; tier (full/light).

**B. Findings by severity** — [DANGER] Critical/High · [WARNING] Medium · [LOW] Low · [SAFE] (only with evidence). Each finding: ID (S-01 security, C-01 compliance), severity, status (Confirmed / Potential / Not Verified), exact file and location, what's wrong, why it matters (one plain sentence), proposed fix (not applied), whether more testing is needed.

**C. By platform** — Core/Backend, Web/PWA, iOS, Android (only relevant ones).

**D. Compliance table** — the Part II items as PASS / WARN / CRITICAL / N/A with one-line reasons.

**E. Test results** — tests run and tools used; passed; failed; not performed; limits and access needed (e.g. live RLS, Netlify env, live headers, full git history).

**F. Top 5 fixes** — ranked by risk.

**G. Verdict (one, covering both parts)** — READY / READY AFTER FIXES / NOT READY / INCONCLUSIVE, with reasoning. Any Critical/High security finding or Part II CRITICAL = NOT READY. Each WARN is fixed or accepted in writing with the reason. Ship only on READY.

**Delivering it:** don't write the report into the project folder during the audit. Write it as `release-check-YYYY-MM-DD.md` in the working directory and send it to Miyoung; offer to save a copy with the project (or to the claude.ai project) once she has reviewed it.

## After the report

Stop and wait. When Miyoung approves specific fixes, make only those, then re-run this check on the changed project and issue a new verdict.