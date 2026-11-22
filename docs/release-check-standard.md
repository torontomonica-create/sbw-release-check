# Small Batch Works — Security Audit Standard

**Adopted 2026-11-20; compliance check added 2026-11-22.** Every application Small Batch Works builds (web app, PWA or native) goes through two checks before it is released to a client or the public, and again before any major change goes live:

1. **Part I — Security:** the Universal Application Security Audit (Miyoung's own text, used as written).
2. **Part II — Compliance (privacy, anti-spam, accessibility, store rules):** Miyoung's Claude Compliance & Security Guide, used as written, **with the Saskatchewan corrections at the start of Part II taking precedence** where they differ.

The notes below say how both are applied at Small Batch Works. Nothing here is legal advice; where a law question decides whether something can ship, confirm with the client, the regulator, or a lawyer.

## How it's applied

**When to run it**

- Before the first release of any client tool, any prototype demo that takes real sign-ups, and any website feature that collects data.
- Again before a release that adds login, a new database table, file uploads, payments, email/SMS sending, or a new third-party service.
- Prototypes with fake data only, and static pages with no forms: run Part 1 plus sections 1 (secrets), 8 (dependencies), 11 and 12 (browser and deployment). Mark the rest Not Applicable with the reason.

**Release rule**

- Ship only on **READY**. One combined decision covers both parts: any Part II **CRITICAL** counts as NOT READY; each Part II **WARN** is either fixed or accepted in writing with the reason.
- Part II's audit prompt asks for "actionable code fixes": at Small Batch Works these go in the report as proposals only. Part I's rule wins: no file is changed until Miyoung approves.
- **READY AFTER FIXES / NOT READY:** propose fixes, wait for Miyoung's approval, fix, then re-run the audit on the changed project.
- **INCONCLUSIVE:** get the missing access (below), or write down in plain words why the gap is acceptable for this tool, before shipping.
- Keep each report with the project as a dated file (e.g. `security-audit-2026-11-20.md`), so the next audit can compare.

**Who runs it**

- A fresh session or a separate agent, not the session that built the app, so the work isn't grading itself.

**What usually needs Miyoung's access** (code alone can't show these; the report must say which were actually checked)

- Supabase: RLS and policies as they exist in the live project (SQL editor output or a read-only connection), storage bucket settings, auth settings.
- Netlify (or other host): environment variables and which ones reach the build, deploy previews, custom headers.
- The live site: response headers and HTTPS redirect (checkable in a browser).
- GitHub: full git history, for secrets committed in the past.

**Look here first — the most common risks in how Small Batch Works builds**

- A Supabase `service_role` key, or any other secret, in frontend code or in an env variable with a public prefix (`PUBLIC_`, `VITE_`, `NEXT_PUBLIC_`). Anything with that prefix ends up in the browser.
- Supabase tables with RLS off, or policies with `USING (true)` that let anyone read sign-up or applicant data.
- Admin pages "protected" only by a hidden URL or a password checked in the browser.
- Public forms (sign-ups, vendor applications, contact) with no spam or rate protection, especially when they send email.
- Residents' or customers' personal data (names, addresses, phone numbers) shown on public pages or maps without the person agreeing to it.

**Limits**

- Passing the audit means the known risks were checked and handled, not that the app can never be attacked. For tools that handle payments or sensitive resident data, also consider a professional review, and tell the client so (see "Not the right fit" in `small-batch-works-foundation.md`).

---
---

# PART I — SECURITY AUDIT

# Universal Application Security Audit
## Pre-Deployment Security Review

Perform a comprehensive security audit of this entire project before deployment or release.

This audit applies to:
- Web Applications
- Progressive Web Apps (PWA)
- Native iOS Applications
- Native Android Applications
- React Native Applications
- Flutter Applications
- Applications using Supabase, Firebase, custom APIs, or other backend services.

Your first task is to identify the application's architecture, technologies, platforms, backend services, authentication methods, data storage, and deployment environment.

Based on the identified architecture, perform all relevant security checks.

Do not assume that this project is a web application or a native application without inspecting its structure first.

---

# IMPORTANT AUDIT RULES

1. Do NOT modify, delete, or create project files during this audit.
2. Do NOT automatically fix any security issues.
3. Do NOT change database settings, permissions, or production configurations.
4. First inspect the project and produce a detailed security audit report.
5. Never expose actual secret values in the report. Only identify the file path and variable name.
6. Distinguish confirmed vulnerabilities from potential risks that require further verification.
7. Do not assume that frontend restrictions provide actual security.
8. Verify server-side and database authorization wherever possible.
9. Do not run destructive tests against production databases or live customer data.
10. Use read-only inspection whenever possible.
11. If a test requires modifying data, use an isolated test environment and test accounts.
12. Do not claim that a security check passed unless there is sufficient evidence.
13. If something cannot be verified, clearly explain what additional access or testing is required.

# PART 1 — PROJECT ARCHITECTURE DISCOVERY

Before performing the audit, identify:

- Application type and supported platforms.
- Programming languages and frameworks.
- Frontend and backend architecture.
- Database technology.
- Authentication provider.
- API architecture and external services.
- File storage services.
- Third-party integrations.
- Hosting and deployment environment.
- Build and release configuration.

Identify whether the application uses:
- Supabase
- Firebase
- Netlify
- Vercel
- Cloudflare
- Custom backend servers
- Other cloud services.

Create a short architecture summary.

Classify each security checklist item as:
- Applicable
- Not Applicable
- Requires Further Verification

Do not report irrelevant checks as vulnerabilities.

# PART 2 — CORE SECURITY AUDIT
## Applicable to ALL application types

### 1. Secrets and Environment Variables

- Check for hardcoded API keys, passwords, access tokens, private keys, and credentials.
- Check for exposed Supabase service_role keys or Firebase Admin credentials.
- Verify that sensitive environment variables are not exposed to frontend or mobile client code.
- Check `.gitignore` for environment files and sensitive configuration files.
- Inspect Git history for previously committed secrets.
- Check whether secrets are exposed through application logs, error messages, build artifacts, or source maps.
- Verify that production secrets are stored securely.
- Check whether development and production credentials are properly separated.

### 2. Authentication and Session Management

- Verify that protected resources require authentication.
- Check login, logout, password reset, and email verification workflows.
- Review session expiration and token refresh behavior.
- Check whether revoked or expired tokens can still access protected resources.
- Check for insecure password storage or weak authentication configurations.
- Verify that authentication tokens are not exposed through URLs or logs.
- Check account recovery processes for possible abuse.
- Verify that multi-factor authentication is used where appropriate.

### 3. Authorization and Access Control

- Verify that authorization is enforced on the server or database level.
- Check whether users can access another user's private data.
- Identify IDOR (Insecure Direct Object Reference) vulnerabilities.
- Check whether users can change request parameters to access unauthorized resources.
- Verify role-based access control.
- Check whether users can escalate their own privileges.
- Verify that admin permissions cannot be granted or modified by ordinary users.
- Check authorization for every sensitive operation, not just page access.

### 4. API and Backend Security

- Identify publicly accessible endpoints.
- Verify authentication and authorization requirements for sensitive endpoints.
- Check for SQL Injection, NoSQL Injection, and command injection vulnerabilities.
- Verify safe parameterized database queries.
- Check input validation on the server side.
- Review API rate limiting and abuse protection.
- Check CORS configuration where applicable.
- Check for insecure error messages that expose internal information.
- Review server-side functions and background jobs for permission bypasses.
- Check for SSRF (Server-Side Request Forgery) where external URLs or server-side fetch requests are accepted.
- Verify that sensitive operations cannot be triggered through unauthorized direct API calls.

### 5. Database Security

For Supabase, Firebase, or other database services:

- Identify all database tables, collections, and sensitive records.
- Check access permissions and security rules.
- For Supabase, verify RLS is enabled on all relevant tables.
- Review RLS policies for overly permissive rules such as `USING (true)` and `WITH CHECK (true)`.
- Check whether anonymous users have unnecessary permissions.
- Review database functions, triggers, and RPC functions.
- Check `SECURITY DEFINER` functions for privilege escalation risks and unsafe search paths.
- Review database grants and default privileges.
- Verify that users cannot read, modify, or delete other users' data without authorization.
- Check whether database backups and exports are appropriately protected.

### 6. Personal Data and Privacy Protection

- Identify personal information collected by the application.
- Check whether unnecessary personal information is being collected.
- Verify access restrictions for sensitive data.
- Check whether personal information is exposed in logs or error messages.
- Review data retention and deletion processes.
- Check whether associated files and database records are deleted when appropriate.
- Verify that test data does not contain real customer information.
- Check whether external services receive personal information unnecessarily.

### 7. File Upload and Storage Security

- Check file size, extension, and MIME type restrictions.
- Verify that uploaded file contents are validated, not just their extensions.
- Check whether file uploads can overwrite existing files.
- Review storage bucket permissions and access policies.
- Verify that private files cannot be accessed without authorization.
- Check for path traversal and file name manipulation.
- Check whether uploaded files could contain malicious scripts or executable content.
- Verify that public download URLs do not expose sensitive files.

### 8. Third-Party Dependencies

- Review package manifests and lockfiles.
- Check for known vulnerabilities in installed dependencies.
- Run `npm audit` when applicable.
- Review outdated or abandoned packages.
- Check for unnecessary dependencies.
- Review third-party SDKs and plugins for security risks.
- Identify dependencies with known critical or high-severity vulnerabilities.
- Verify that production builds do not include unnecessary development tools.

### 9. Logging and Monitoring

- Check whether sensitive information is written to logs.
- Verify that authentication failures and suspicious activity can be monitored.
- Review administrative action logging.
- Check whether logs are accessible to unauthorized users.
- Verify that errors do not expose database credentials, tokens, or internal system information.
- Identify missing monitoring for sensitive operations.

### 10. Abuse and Resource Protection

- Check whether public endpoints can be abused through automated requests.
- Review rate limits for authentication, forms, APIs, and expensive operations.
- Check for duplicate submissions and repeated transactions.
- Review potential abuse of paid services such as email, SMS, AI APIs, and cloud functions.
- Check whether excessive requests could exhaust database or server resources.
- Review idempotency protection for payment and transaction operations where applicable.

---

# PART 3 — WEB APPLICATION SECURITY

Perform these checks when the project includes a web application or web-based interface.

### 11. Browser and Frontend Security

- Check for XSS vulnerabilities.
- Review usage of `dangerouslySetInnerHTML`, `innerHTML`, and unsafe HTML rendering.
- Check for CSRF vulnerabilities where cookie-based authentication is used.
- Review Content Security Policy (CSP).
- Check security headers:
  - Strict-Transport-Security
  - X-Content-Type-Options
  - Content-Security-Policy
  - Referrer-Policy
  - Permissions-Policy
- Verify secure cookie attributes where applicable:
  - HttpOnly
  - Secure
  - SameSite
- Check whether sensitive information is stored in localStorage or sessionStorage unnecessarily.
- Check whether frontend source maps expose sensitive information.
- Review authentication redirects and open redirect vulnerabilities.
- Check whether third-party scripts introduce unnecessary security risks.

### 12. Web Deployment Security

- Verify HTTPS enforcement.
- Check production versus development configurations.
- Verify that debug tools and development endpoints are disabled in production.
- Review Netlify, Vercel, Cloudflare, or other hosting configurations.
- Check environment variable exposure in build settings.
- Verify that preview deployments cannot unintentionally access production data.
- Review DNS and domain configuration where relevant.

### 13. PWA-Specific Security

- Review service worker registration and update behavior.
- Check whether cached data contains sensitive user information.
- Verify that private API responses are not stored in public caches.
- Check service worker scope and permissions.
- Review offline functionality for unauthorized access to previously cached data.
- Verify that the application does not continue using outdated or vulnerable cached resources.

---

# PART 4 — NATIVE MOBILE APPLICATION SECURITY

Perform these checks when the project includes native iOS, Android, React Native, or Flutter applications.

### 14. Local Data Storage

- Check whether passwords, access tokens, or sensitive personal information are stored in plaintext.
- Verify appropriate use of:
  - iOS Keychain
  - Android Keystore
  - Secure Storage libraries
- Review SQLite and local database encryption where sensitive data is stored.
- Check whether sensitive data is exposed through application logs.
- Review temporary files, cached data, and downloaded documents.
- Verify that sensitive information is not unnecessarily retained after logout.

### 15. Mobile Network Security

- Verify that API communication uses HTTPS.
- Check for insecure HTTP connections.
- Review certificate validation.
- Check for disabled TLS certificate validation.
- Identify insecure custom certificate handling.
- Review certificate pinning if used and assess whether its implementation is appropriate.
- Verify that authentication tokens are transmitted securely.

### 16. Application Permissions

- Review requested device permissions.
- Check camera, microphone, location, contacts, photos, and file access.
- Identify unnecessary permissions.
- Verify that permissions are requested only when required.
- Check application behavior when users deny permissions.
- Review background location and background execution permissions.

### 17. Mobile Code and Reverse Engineering

- Check whether API keys, credentials, or secrets can be extracted from application binaries.
- Review debug and release build configurations.
- Check whether debugging is enabled in production.
- Review code obfuscation where appropriate.
- Check for sensitive information exposed through application resources or configuration files.
- Review protection against unauthorized application modification where appropriate.
- Check whether security-critical decisions are made only inside the mobile client.

Note: Code obfuscation and anti-tampering measures are additional defenses. They do not replace server-side authorization.

### 18. Deep Links and Inter-App Communication

- Review Android Intent Filters and iOS URL schemes.
- Check Universal Links and Android App Links configuration.
- Verify that incoming deep links are validated.
- Check whether malicious URLs can trigger sensitive actions.
- Review exported Android components.
- Check whether authentication can be bypassed through deep links.
- Review WebView JavaScript bridges and external content handling.

### 19. Mobile Platform Configuration

For Android:
- Review AndroidManifest.xml.
- Check exported components.
- Review network security configuration.
- Check backup and data extraction rules.
- Review release signing configuration.
- Check whether sensitive activities are improperly exposed.

For iOS:
- Review Info.plist.
- Check App Transport Security configuration.
- Review URL schemes and associated domains.
- Check Keychain access groups.
- Review application entitlements.
- Verify release signing and provisioning configuration.

---

# PART 5 — PAYMENT AND FINANCIAL SECURITY

Perform these checks if the application handles payments, subscriptions, donations, or financial transactions.

- Verify that payment amounts are calculated and validated server-side.
- Check whether users can manipulate prices or transaction values.
- Verify payment provider webhook signatures.
- Check whether payment confirmation can be forged.
- Review duplicate payment and replay attack protection.
- Verify that sensitive payment credentials are never stored unnecessarily.
- Check refund and cancellation authorization.
- Verify that payment status cannot be changed directly by ordinary users.
- Check whether financial records are protected against unauthorized modification.

Do not perform real transactions during the audit.

---

# PART 6 — DEPLOYMENT AND RELEASE READINESS

### 20. Production Configuration

- Verify production credentials and environment variables.
- Check that development and test configurations are not active.
- Review production database permissions.
- Check HTTPS and network security.
- Verify that unnecessary public endpoints are disabled.
- Check deployment access controls.
- Review backup and recovery arrangements.
- Verify that sensitive configuration files are not included in release artifacts.

### 21. Build and Release Integrity

- Check for unexpected scripts or commands in build configurations.
- Review CI/CD workflow permissions.
- Check whether deployment credentials are properly protected.
- Verify that production builds use trusted dependencies.
- Review application signing and release artifact integrity.
- Check whether unauthorized parties can modify or publish releases.

---

# PART 7 — SECURITY TESTING

Perform safe, non-destructive security tests where possible.

Examples:

- Run dependency vulnerability scans.
- Run secret scanning tools such as Gitleaks.
- Inspect Git history for exposed credentials.
- Run static analysis tools where available.
- Review Supabase RLS policies.
- Test authorization using isolated test accounts.
- Check whether unauthenticated users can access protected resources.
- Verify that ordinary users cannot access admin functionality.
- Inspect mobile build artifacts for exposed secrets when applicable.

Do not:
- Attack external systems.
- Perform destructive database operations.
- Modify production data.
- Expose or print secret values.
- Disable security protections to make tests easier.

Clearly identify tests that were actually executed versus checks based only on source code review.

---

# PART 8 — SECURITY AUDIT REPORT

Organize the final report into the following sections.

## A. Project Architecture Summary

Identify:
- Application type.
- Platforms.
- Frameworks and technologies.
- Backend and database services.
- Authentication system.
- Hosting and deployment environment.

## B. Security Findings by Severity

Use these severity levels:

**[DANGER] Critical / High**
- Unauthorized access to sensitive data.
- Exposed privileged credentials.
- Authentication or authorization bypass.
- Critical database or payment vulnerabilities.
- Other issues with serious security impact.

**[WARNING] Medium**
- Weak security configurations.
- Potential vulnerabilities requiring additional verification.
- Missing protections that could enable abuse.
- Risks that depend on application usage or deployment settings.

**[LOW] Low**
- Minor security weaknesses.
- Defense-in-depth improvements.
- Non-critical configuration issues.

**[SAFE] No Issues Found**
- Areas inspected with sufficient evidence of appropriate security controls.

Do not label uninspected areas as SAFE.

For every finding, include:

1. Finding ID.
2. Severity.
3. Confirmation status:
   - Confirmed
   - Potential
   - Not Verified
4. Exact file path and code location.
5. Description of the issue.
6. Why it matters.
7. Recommended solution.
8. Whether additional testing is required.

## C. Platform-Specific Findings

Separate findings into:
- Core / Backend
- Web / PWA
- Native iOS
- Native Android

Only include relevant sections.

## D. Security Test Results

Report:
- Tests performed.
- Tools used.
- Tests passed.
- Tests failed.
- Tests not performed.
- Limitations and required access.

## E. Top 5 Priority Fixes

List the five most important issues to address before deployment, ranked by risk.

## F. Final Deployment Assessment

Choose one:

- **READY:** No identified blocking security issues. Remaining risks are acceptable for the intended use.
- **READY AFTER FIXES:** Deployment should wait until identified blocking issues are resolved.
- **NOT READY:** Critical or high-risk vulnerabilities remain.
- **INCONCLUSIVE:** Important areas could not be inspected or verified.

Explain the reasoning behind the assessment.

---

# FINAL INSTRUCTION

This is an audit-only task.

Do not modify any code, configuration, database policy, or deployment setting.

Wait for my review and explicit approval before proposing or implementing any security fixes.

The goal is to identify security risks before deployment, not to automatically change the application.

---
---

# PART II — COMPLIANCE CHECK

## Saskatchewan corrections (take precedence over the guide below)

Reviewed 2026-11-22. The guide below was written for Canada-based developers in general and leans on Ontario and federal rules. For Small Batch Works clients in Saskatchewan, read it with these corrections. Sources checked are listed at the end of this section; re-check them once a year, because these rules change.

**Default project profile for Small Batch Works**

- Target region: **Canada** checked. **Global / US / EU** only when the tool is meant for users outside Canada (not the default; the guide's template has it pre-checked). **South Korea** unchecked.
- Platform: **Web App** checked unless the project is native.
- Feature flags: set per project. Note: smallbatchworks.ca itself runs GA4, so **Tracking** is checked for the site.

**Which privacy law applies depends on the client, not on the app**

- **Private business** (shop, salon, chef service, daycare that charges fees): PIPEDA. A non-profit daycare was found subject to PIPEDA because fees for child care are commercial activity.
- **Town or city** (e.g. a City event tool that stores residents' names and addresses): The Local Authority Freedom of Information and Protection of Privacy Act (LA FOIP). The client is the local authority; Small Batch Works is likely its *information management service provider*, and LA FOIP has a section on agreements between a local authority and such a provider. Before launch, ask the town whether it needs a written agreement covering what data is kept, where, who can see it, and when it is deleted.
- **Church, club, community group, charity:** usually not covered by PIPEDA unless the activity is commercial (selling or sharing member or donor lists is commercial). Follow PIPEDA-style practice anyway.
- **Health information:** avoid collecting it. If a tool would hold it, stop and check The Health Information Protection Act (HIPA) first.
- **Quebec Law 25:** only when the tool handles Quebec residents' data. Not a default requirement.

**Accessibility**

- The guide's **ACA** applies to federally regulated organizations and **AODA** is Ontario law; neither applies to a Saskatchewan small business.
- **The Accessible Saskatchewan Act** (in force Dec 3, 2023) requires listed public sector bodies, including larger municipalities (reported as over 10,000 people), to publish accessibility plans. It does not yet set a web standard. Check the regulation's list when the client is a town.
- The Saskatchewan Human Rights Code's duty to accommodate applies to everyone serving the public.
- **Small Batch Works keeps WCAG 2.1 AA as its own quality bar** (good UX, and safe for public-sector clients), but reports it as a quality check, not a legal requirement, unless the client is bound by one.

**Breaches (missing from the guide)**

- For clients under PIPEDA: keep a record of **every** breach of security safeguards involving personal information, and report breaches that create a real risk of significant harm to the Privacy Commissioner and notify the people affected. Knowingly failing to do this can bring fines.
- Each tool that holds personal data needs a one-paragraph incident plan: who tells the client, how to switch the tool off, how to find out what was exposed.

**Anti-spam (CASL)**

- The guide is right that marketing email or SMS needs consent, sender identification and a working unsubscribe. CASL also has implied consent in some cases (e.g. an existing business relationship), so "express opt-in only" is stricter than the law; it's a fine default.
- Confirmation emails that people asked for (a sign-up confirmation) are not marketing. Don't add promotions to them without checking CASL.

**Raffles and lotteries (missing from the guide; affects the Community Fundraising Hub)**

- All gaming in Saskatchewan must be licensed by SLGA. Selling raffle tickets online is allowed only through an **Electronic Raffle System** that is independently certified, supplied by an SLGA-registered gaming supplier, approved by SLGA in advance and hosted in Saskatchewan, with buyers in Saskatchewan. (The hosting and certification details come from secondary sources; confirm directly with SLGA before any real-money work.)
- Until then, raffle tools stay demo-only with simulated payments, as already decided.

**Where data is stored**

- Supabase, Netlify and Resend run outside Canada by default. PIPEDA allows this when the privacy notice says so. For town clients, ask the town whether storing residents' data outside Canada is acceptable.

**Prototypes and demo data**

- Fake data in prototypes is fine when it's clearly a demo (consistent with the guide's "no fake content" rule and the foundation's rule that prototypes stay generic or fictional). Never put real residents' or customers' data in a demo.

**Sources checked (2026-11-22)**

- Office of the Privacy Commissioner of Canada: mandatory breach reporting under PIPEDA; how PIPEDA applies to charitable and non-profit organizations.
- Canadian Bar Association, Charities and Not-for-Profit Law: PIPEDA and non-profits (daycare finding).
- LA FOIP (Saskatchewan Publications Centre), sections 23.1–23.2 and LA FOIP Regulations s. 8.2.
- Accessible Saskatchewan Regulations (accessiblesk.saskatchewan.ca); MLT Aikins; Global News (municipalities over 10,000).
- SLGA raffle terms and conditions (Electronic Raffle Systems); Zeffy guide to Saskatchewan charitable lottery rules.

---

# Claude Development Compliance & Security Guide (Multi-Region & Platform)

> A modular `CLAUDE.md` rule set and pre-launch audit prompt designed for Canada-based developers targeting **Canada, Global (US/EU), or South Korea** across **Web Apps and Native Mobile Apps**.
>
> **How to Use:** Copy **Section 2** into your project's root `CLAUDE.md` file and toggle the checkboxes (`[x]`) under `Project Profile`. Claude will strictly enforce only the rules relevant to your selected profile without over-engineering.

---

## 1. Key Checkpoints by Region & Platform

### 🌍 Regional Compliance Highlights
* **🇨🇦 Canada (Default / Local):**
  * **PIPEDA & Quebec Law 25:** Requires clear privacy disclosures, explicit opt-in consent, data deletion rights, and designated **Privacy Officer** contact information.
  * **CASL (Canada's Anti-Spam Legislation):** One of the strictest anti-spam laws globally. Requires **Express Consent (Opt-in)** before sending marketing emails/SMS, clear sender identification, and a working 1-click unsubscribe mechanism.
  * **Accessibility (ACA / AODA):** Strong compliance requirements aligned with WCAG 2.0/2.1 AA standards.
* **🌐 Global (US / EU):**
  * **US:** CCPA/CPRA (California "Do Not Sell or Share" opt-out), COPPA (children under 13), and ADA (web/app accessibility—the #1 source of digital compliance lawsuits).
  * **EU:** GDPR & ePrivacy Directive (prior opt-in consent banner blocking non-essential tracking cookies, right to erasure, data portability).
* **🇰🇷 South Korea (Optional):**
  * **PIPA & E-Commerce Act:** Restricts users under 14 without parental consent, separates `[Required]` vs. `[Optional]` consent checkboxes, and mandates displaying business registration details (Company Name, CEO, Business Registration No., Mail-Order Sales No., Address, Contact) in the footer.

### 📱 Platform-Specific Highlights
* **🌐 Web App:** Cookie consent banner (if using tracking/ads), legal footer links, full keyboard navigation, WCAG color contrast, and transparent checkout/cancellation flows.
* **📱 Native Mobile App (iOS / Android):**
  * **In-App Account Deletion (Store Mandatory):** If account creation is supported, users **must** be able to permanently delete their account inside the app (top rejection reason on App Store & Google Play).
  * **App Tracking Transparency (iOS ATT):** Required before initializing ad/tracking SDKs on iOS.
  * **UGC Moderation:** Apps with posts, comments, or chat **must** include Report and Block features to pass store review.
  * **In-App Purchases (IAP):** Digital goods/subscriptions must follow store billing rules and include a **Restore Purchases** button.

---

## 2. Claude Instructions (`CLAUDE.md` Template)

Copy the block below into your project's `CLAUDE.md` and check `[x]` the applicable boxes.

```markdown
# Project Compliance, Security & Accessibility Rules

## 0. Project Profile
<!-- Claude MUST strictly enforce rules for [x] checked items and skip unchecked items to avoid unnecessary overhead. -->

### Target Region (Select all that apply)
- [x] **Canada** (PIPEDA, Quebec Law 25, CASL, ACA/AODA)
- [x] **Global / US / EU** (GDPR, CCPA/CPRA, COPPA, ADA)
- [ ] **South Korea** (PIPA, E-Commerce Act)

### Platform (Select all that apply)
- [x] **Web App** (Browser / PWA / SaaS)
- [ ] **Native Mobile App** (iOS / Android / React Native / Flutter)

### Feature Flags (Select if applicable)
- [ ] **Paid / Subscription** (One-time payments or recurring billing)
- [ ] **Tracking / Ads** (GA4, Meta Pixel, third-party ad/analytics SDKs)
- [ ] **Marketing Email / SMS** (Newsletters, promotional outreach)
- [ ] **UGC (User Generated Content)** (Posts, comments, chat, public profiles)
- [ ] **AI Features** (LLM-generated outputs, summaries, recommendations)

---

## 1. Core Mandatory Rules (Always Apply)

### Security & Data Protection
- **No Hardcoded Secrets:** Never hardcode API keys, secrets, or database credentials. Use environment variables (`.env`) and ensure server-only secrets are never exposed to client bundles.
- **Server-Side AuthN & AuthZ:** Enforce authentication and authorization checks on all backend API endpoints and database policies (e.g., RLS). Never rely solely on client-side checks.
- **PII-Free Logging:** Never log Personally Identifiable Information (passwords, tokens, emails, phone numbers) in `console.log` or server logs.
- **Open-Source Licensing:** Prefer permissive licenses (`MIT`, `Apache-2.0`, `BSD`) when adding dependencies. Explicitly warn before introducing copyleft (`GPL`, `AGPL`) packages.

### Privacy & Anti-Dark Patterns
- **Data Minimization:** Collect only the minimum user data strictly required for the feature to function.
- **Explicit Opt-In:** All terms, privacy, and marketing consent checkboxes MUST default to **unchecked**.
- **Account & Data Deletion:** If user accounts exist, implement a self-service flow for users to permanently delete their account and associated personal data.
- **No Deceptive UI / Fake Content:** Never generate fake reviews, fabricated testimonials, or unsubstantiated marketing claims (mark placeholder copy with `TODO: Replace with real data`). Never hide, obscure, or obstruct cancel/reject/opt-out actions.
- **Asset Licensing:** Use only commercially free, open-licensed fonts and icons/assets (e.g., Google Fonts OFL, Lucide).

---

## 2. Region-Specific Conditional Rules (Apply if Checked)

### [IF: Canada]
- **Privacy (PIPEDA / Quebec Law 25):** Include placeholders/sections in the Privacy Policy for data collection purpose, retention period, third-party disclosures, and **Privacy Officer contact information**.
- **Anti-Spam (CASL):** If `[Marketing Email / SMS]` is checked, record explicit opt-in consent timestamps in the database and include clear sender identification plus a working 1-click **Unsubscribe** link in every message.
- **Accessibility (ACA / AODA):** Conform to WCAG 2.1 AA standards.

### [IF: Global / US / EU]
- **Cookies & Tracking (GDPR / ePrivacy):** If `[Tracking / Ads]` is checked, implement a Cookie Consent Banner with equal prominence for "Accept" and "Reject" buttons, blocking all non-essential tracking scripts until explicit consent is granted, alongside a Cookie Policy link. (Skip if only strictly necessary session cookies are used.)
- **Children's Privacy (COPPA):** Unless explicitly built for children, state and enforce an age restriction for users under 13 during sign-up.
- **California Privacy (CCPA/CPRA):** If user data is shared or sold to third parties, provide a "Do Not Sell or Share My Personal Information" footer link.
- **Accessibility (ADA):** Strictly follow WCAG 2.1 AA standards across all user flows.

### [IF: South Korea]
- **Age Restriction:** Enforce and display a restriction for users under 14 (without legal guardian consent).
- **Granular Consent:** Clearly separate `[Required]` and `[Optional]` consent items on sign-up forms.
- **Business Information Footer:** Include footer fields for Company Name, CEO Name, Business Registration Number, Mail-Order Business Registration Number (if paid), Business Address, and Customer Support Contact.

---

## 3. Platform-Specific Conditional Rules (Apply if Checked)

### [IF: Web App]
- **Web Accessibility (A11y):** Provide meaningful `alt` attributes on all `<img>` tags (`alt=""` for decorative images), maintain a minimum 4.5:1 text-to-background contrast ratio, and ensure full keyboard navigation (`Tab`, `Enter`, `Esc`) with visible `focus-visible` states on all interactive elements (modals, dropdowns, forms).
- **Legal Footer Links:** Include footer links to Privacy Policy, Terms of Service, Refund Policy (if paid), and Cookie Policy (if tracking is enabled).

### [IF: Native Mobile App]
- **In-App Account Deletion:** If sign-up is supported, implement an in-app UI and backend API allowing users to delete their account without leaving the app.
- **iOS ATT Permission:** If `[Tracking / Ads]` is checked, configure `NSUserTrackingUsageDescription` and implement the App Tracking Transparency (ATT) prompt flow on iOS before initializing tracking SDKs.
- **Mobile Accessibility:** Add `accessibilityLabel` and `accessibilityRole` (for VoiceOver / TalkBack) to interactive buttons, icons, and images.
- **In-App Purchases:** If `[Paid / Subscription]` is checked, follow Apple/Google IAP policies, clearly display price, renewal cycle, and cancellation terms, and include a **Restore Purchases** button.

---

## 4. Feature-Specific Conditional Rules (Apply if Checked)
- **[IF: Paid / Subscription]:** Display transparent pricing (including taxes, fees, and auto-renewal terms) prior to checkout, link to a Refund Policy, and provide a straightforward cancellation flow.
- **[IF: UGC]:** Provide **Report Content** and **Block User** controls on all screens displaying user-generated posts, comments, or messages.
- **[IF: AI Features]:** Display a clear disclaimer near AI-generated outputs stating that responses may be inaccurate and should be independently verified.
```

---

## 3. Pre-Launch Audit Prompt for Claude (Copy & Paste)

Run this prompt with Claude before deploying to audit your codebase against the active conditions in your `CLAUDE.md`:

```text
Audit this entire codebase for legal, compliance, security, and accessibility risks strictly based on the checked items under [Project Profile] in CLAUDE.md.
Mark unchecked conditions as "N/A" and evaluate all checked conditions using a [PASS / WARN / CRITICAL] summary table across the 5 categories below. For any WARN or CRITICAL items, list the exact affected files and provide actionable code fixes:

1. Security & Licensing: Hardcoded secrets, missing server-side AuthN/AuthZ, PII in logs, GPL/AGPL dependencies, and unverified font/asset licenses.
2. Regional Compliance: Missing requirements for checked regions (Canada PIPEDA/Law 25/CASL, Global GDPR/CCPA/COPPA/ADA, South Korea PIPA/E-Commerce Act).
3. Platform Requirements: (Web) WCAG 2.1 AA accessibility, legal footer links, cookie consent blocking / (Native) In-app account deletion, iOS ATT flow, screen-reader labels, Restore Purchases button.
4. Privacy & Anti-Dark Patterns: Unnecessary form fields, pre-checked consent boxes, fake reviews/placeholder claims, hidden fees, or obstructive cancellation flows.
5. Feature Requirements: (Paid) Transparent pricing & refund policy / (UGC) Report & Block flows / (AI) Output disclaimers / (Marketing) Express opt-in & 1-click unsubscribe.
```