# Shared Ecosystem Architecture & Authentication Service – Full Plan

**Project Context**  
Two separate applications with distinct domains:  
- **Tour & Rentals Management** (recurring activities, short-term / long-term rentals – STR/LTR)  
- **Lesson Planning for Teachers**  

Goal: Share common foundational components (auth, RBAC, frontend base, etc.) without tight coupling and **without a monorepo**.

Current date reference: January 2026  
CSS framework of choice: **Tabler.io** (based on Bootstrap 5, heavy use of CSS variables)

## 1. Overall Architecture Principles

- **Bounded Contexts** (DDD): Each app is its own domain. Shared concerns form a minimal shared kernel.  
- **Independent Repositories**: Separate Git repos for:  
  - tour-app  
  - lesson-app  
  - auth-backend (Rust + PostgreSQL)  
  - shared-frontend (TypeScript/NPM package(s))  
- **Loose Coupling**: API contracts only (REST/GraphQL/gRPC), no shared DB between apps.  
- **Versioning & Delivery**: Semantic versioning for NPM packages & Docker images. Contract testing (Pact/OpenAPI).  
- **Deployment**: Shared services independent (Kubernetes/serverless). Apps use env vars for endpoints.

### Shared Components Overview

| Component               | Tech                              | Delivery                  | Customization / White-labeling |
|-------------------------|-----------------------------------|---------------------------|--------------------------------|
| Frontend base           | Tabler.io + custom wrappers       | NPM package               | CSS variables override         |
| Auth frontend forms     | TypeScript/React/Vue/Svelte       | NPM package               | Theme injection                |
| Auth backend            | Rust + PostgreSQL 16+             | Docker + API              | —                              |
| RBAC                    | JWT claims + endpoint             | Part of auth service      | App-specific roles via claims  |
| Other utils             | Logging, monitoring, etc.         | NPM / lightweight services| —                              |

## 2. Authentication Service – Core Specification

**Backend**: Rust service  
**Database**: PostgreSQL 16+  
**Frontend**: TypeScript library (provides forms + client flows)  
**Supported auth methods**:
- Legacy: email + password (Argon2id)
- OAuth 2.0 / OIDC (Google, Apple, GitHub, …)
- Passkeys (WebAuthn)
- Hardware tokens (YubiKey / FIDO2)
- 2FA: TOTP, backup codes (SMS optional)

## 3. User Stories – Authentication Flows

### Sign Up

1. **As a new user, I want to sign up with email and password**  
   AC: email validation, strong password, hash (Argon2), verification email, JWT after verify  
   Tests: unit (validation, hash, duplicate), E2E (form → API → DB → email → verify)

2. **As a new user, I want to sign up via OAuth**  
   AC: redirect → callback → create/link account → token  
   Tests: unit (URL gen, callback parse, upsert), E2E (mock provider flow)

3. **As a new user, I want to sign up with a passkey**  
   AC: WebAuthn registration, store credential, associate with email  
   Tests: unit (challenge, storage), E2E (browser ceremony → backend)

4. **As a new user, I want to sign up with YubiKey/hardware token**  
   (Similar to passkey – multi-device support)

5. **As a new user, I want to enable 2FA during signup**  
   AC: TOTP QR/secret, verify code before finish  
   Tests: unit (secret + code validation), E2E (setup → verify)

### Sign In

1. **As an existing user, I want to sign in with email/password** (rate limiting)  
2. **As an existing user, I want to sign in via OAuth**  
3. **As an existing user, I want to sign in with passkey**  
4. **As an existing user, I want to sign in with YubiKey**  
5. **As a 2FA user, I want to complete login with second factor**  

Tests for each: unit (core logic), E2E (full browser/API flow)

### Account Recovery

1. **Forgot password** → email reset link → new password → invalidate sessions  
2. **OAuth relink/recovery** → email verify → add/remove provider  
3. **Lost passkey/YubiKey** → email/backup → register new device  
4. **2FA recovery** → use backup codes (one-time)

Tests for each: unit + E2E

### Invitation Scenario (Multi-tenant Onboarding)

**Flow**  
1. Inviter (in app) → generate invite → auth service creates token (expiry, role, app_id, email)
2. Email with magic link  
3. Invitee clicks → auth form verifies token → signup/sign-in → link account  
4. Auth service → webhook/event to app → grant domain access (team/group/role)

**Additional Stories**  
- Generate invite link  
- Accept invite & onboard  
- View pending/accepted invites  

Tests: unit (token gen/validation), E2E (full invite → onboard → access)

## 4. Frontend – Tabler.io Integration & White-Labeling

**Why Tabler?**  
- Modern, clean design  
- Bootstrap 5 foundation  
- Extensive CSS custom properties (variables) for theming  
- Built-in light/dark modes

**Shared Frontend Repo Structure** (NPM package)

shared-frontend/
├── src/
│   ├── components/         # Thin wrappers: Button, Card, Modal, etc.
│   ├── auth/               # SignInForm, SignUpForm, RecoveryForm, etc.
│   ├── styles/
│   │   ├── base.css        # @import tabler.min.css + resets
│   │   ├── variables.css   # Default :root overrides
│   └── hooks/
│       └── useTheme.ts
├── package.json
└── vite.config.ts

**White-Label / Theme Customization (Very Easy)**

Override via CSS variables – no need to fork Tabler.

```css
/* Example overrides – inject per app/customer */

:root,
[data-bs-theme="light"] {
  --tblr-primary:           #206bc4;     /* Change this for brand */
  --tblr-primary-fg:        white;
  --tblr-body-bg:           #ffffff;
  --tblr-body-color:        #182433;
  --tblr-font-sans-serif:   "Inter", system-ui, sans-serif;
  --tblr-border-color:      rgba(24, 36, 51, 0.12);
}

[data-bs-theme="dark"] {
  --tblr-primary:           #5c8dff;
  --tblr-body-bg:           #0f172a;
  --tblr-body-color:        #e2e8f0;
}
```


## Injection Methods
A. Build-time / static (recommended for white-label SaaS)
```
HTML<!-- index.html -->
<link rel="stylesheet" href="tabler.min.css">
<link rel="stylesheet" href="shared-frontend/base.css">

<style>
  :root {
    --tblr-primary: #8b5cf6;           /* Purple for this tenant */
    --tblr-font-sans-serif: "Poppins", sans-serif;
  }
</style>
```
B. Runtime (multi-tenant preview / user toggle)

```tsx// useTheme hook or dynamic <style> injection
const brand = { primary: "#ef4444", font: "Roboto" };

useEffect(() => {
  const style = document.createElement("style");
  style.textContent = `
    :root {
      --tblr-primary: ${brand.primary};
      --tblr-font-sans-serif: "${brand.font}", system-ui, sans-serif;
    }
  `;
  document.head.appendChild(style);
}, []);
```
## Example Auth Form (using standard Tabler classes)
```tsx
<div className="container-tight py-4">
  <div className="text-center mb-4">
    <img src="/logo.svg" alt="Brand" style={{ height: "var(--tblr-logo-height, 40px)" }} />
  </div>
  <Card className="card-md">
    <Card.Body>
      <h2 className="h2 text-center mb-4">Login to your account</h2>
      <form>
        <div className="mb-3">
          <label className="form-label">Email</label>
          <input type="email" className="form-control" placeholder="your@email.com" />
        </div>
        <div className="mb-2">
          <label className="form-label">Password</label>
          <input type="password" className="form-control" />
        </div>
        <Button color="primary" className="w-100">Sign in</Button>
      </form>
    </Card.Body>
  </Card>
</div>
```
## White-Label Checklist

```
Brand color        → --tblr-primary
Fonts              → --tblr-font-sans-serif
Dark/light mode    → data-bs-theme attribute
Logo               → CSS var or direct <img> override
Accents            → --tblr-success, --tblr-info, etc.
Per-tenant         → DB → API → dynamic <style>
```

This plan keeps domains isolated, auth reusable & secure, UI modern & brandable with minimal effort.
Ready for implementation – let me know which part to drill down into next (e.g. Rust service endpoints, OpenAPI spec, detailed theme hook, etc.).
