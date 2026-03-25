# Keycloak Hands-On IAM Labs

> A comprehensive lab guide for learning Identity & Access Management concepts using Keycloak running locally. Each lab is structured as a self-contained deep-dive, blog-ready exercise covering real-world IAM scenarios.

---

## Prerequisites

Before starting any lab, ensure you have Keycloak running locally:

```powershell
# Docker (recommended)
docker run -p 8080:8080 `
-e KC_BOOTSTRAP_ADMIN_USERNAME=admin `
-e KC_BOOTSTRAP_ADMIN_PASSWORD=admin `
-v keycloak_data:/opt/keycloak/data `
quay.io/keycloak/keycloak:latest start-dev

# Or you can use the docker-compose file provided with this repo
git clone https://github.com/shaheerkj/keycloak-iam-labs
cd keycloak-iam-labs/
docker compose up -d

# Admin Console
http://localhost:8080/admin
```

**Recommended free tools to have ready:**
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) — container runtime
- [Postman](https://www.postman.com/downloads/) or [Bruno](https://www.usebruno.com/) — API/token testing
- [jwt.io](https://jwt.io) — JWT inspection (browser-based)
- [Node.js](https://nodejs.org/) — for sample app clients
- [Python 3](https://www.python.org/) — for scripting and CLI clients
- [ngrok](https://ngrok.com/) (free tier) — expose localhost for webhooks/external IdPs

---

## Lab Index

| # | Lab Title | IAM Domain | Difficulty |
|---|-----------|------------|------------|
| 1 | Realm Setup & First Login Flow | Authentication Basics | ⭐ Beginner |
| 2 | Username/Password Authentication & Brute-Force Protection | Authentication | ⭐ Beginner |
| 3 | Multi-Factor Authentication (TOTP & WebAuthn) | MFA | ⭐⭐ Intermediate |
| 4 | OAuth 2.0 Authorization Code Flow | OAuth 2.0 | ⭐⭐ Intermediate |
| 5 | OAuth 2.0 Client Credentials Flow | OAuth 2.0 / Machine Identity | ⭐⭐ Intermediate |
| 6 | OpenID Connect (OIDC) — ID Tokens & UserInfo | OIDC | ⭐⭐ Intermediate |
| 7 | Role-Based Access Control (RBAC) | Authorization | ⭐⭐ Intermediate |
| 8 | Attribute-Based Access Control (ABAC) with Claims | Authorization | ⭐⭐⭐ Advanced |
| 9 | Single Sign-On (SSO) Across Multiple Apps | SSO | ⭐⭐ Intermediate |
| 10 | Single Logout (SLO) | SSO / Session Management | ⭐⭐ Intermediate |
| 11 | Social Login — Google OAuth | Social Identity | ⭐⭐ Intermediate |
| 12 | Social Login — GitHub OAuth | Social Identity | ⭐⭐ Intermediate |
| 13 | SAML 2.0 Service Provider Integration | SAML / Federation | ⭐⭐⭐ Advanced |
| 14 | Identity Brokering & External IdP Federation | Federation | ⭐⭐⭐ Advanced |
| 15 | User Federation with LDAP (OpenLDAP) | Directory Services | ⭐⭐⭐ Advanced |
| 16 | SCIM 2.0 User Provisioning | Provisioning | ⭐⭐⭐ Advanced |
| 17 | Custom Authentication Flow (OTP via Email) | Auth Flows | ⭐⭐⭐ Advanced |
| 18 | Passwordless Authentication (Magic Link / WebAuthn) | Passwordless | ⭐⭐⭐ Advanced |
| 19 | Fine-Grained Authorization with Keycloak Authorization Services | Policy Engine | ⭐⭐⭐⭐ Expert |
| 20 | Token Exchange & Delegation | Advanced OAuth | ⭐⭐⭐⭐ Expert |
| 21 | JWT Customization with Protocol Mappers | Token Engineering | ⭐⭐ Intermediate |
| 22 | Refresh Token Rotation & Token Revocation | Session Security | ⭐⭐ Intermediate |
| 23 | PKCE — Securing Public Clients | OAuth 2.0 Security | ⭐⭐ Intermediate |
| 24 | Device Authorization Flow (OAuth 2.0 Device Flow) | OAuth 2.0 | ⭐⭐⭐ Advanced |
| 25 | User Self-Service — Registration, Profile & Password Reset | UX & Lifecycle | ⭐ Beginner |
| 26 | Admin REST API Automation | Provisioning / IaC | ⭐⭐ Intermediate |
| 27 | Event Listeners & Audit Logging | Observability | ⭐⭐ Intermediate |
| 28 | Session Management & Concurrent Session Limiting | Session Policy | ⭐⭐ Intermediate |
| 29 | Securing a React SPA with Keycloak.js | App Integration | ⭐⭐ Intermediate |
| 30 | Securing a Node.js / Express API | App Integration | ⭐⭐ Intermediate |

---

## Lab 1 — Realm Setup & First Login Flow

### Goal
Bootstrap a clean Keycloak environment by creating a dedicated realm, registering your first client, creating a test user, and completing a full login cycle through the browser.

### Tools Required
- Keycloak Admin Console (`http://localhost:8080/admin`)
- Any modern web browser

### Key IAM Concepts
- **Realm**: An isolated namespace of IAM configuration — users, clients, roles, and policies live inside a realm. Like a tenant in a multi-tenant system.
- **Client**: A registered application that delegates authentication to Keycloak.
- **User**: A managed identity with credentials, attributes, and role assignments.
- **Login Flow**: The ordered sequence of authentication steps a user must complete.

### Step-by-Step Outline
1. Log into the master realm as admin.
2. Create a new realm: `lab-realm`.
3. Create a client: `my-web-app` (OpenID Connect, public client type).
4. Set valid redirect URIs: `http://localhost:3000/*`.
5. Create a user: `testuser` with a temporary password.
6. Force a password reset on first login.
7. Visit the Keycloak Account Console: `http://localhost:8080/realms/lab-realm/account`.
8. Log in as `testuser` and complete the required password change.
9. Inspect the session in Admin Console → Sessions.

### Lab Exercises
- Change the login theme and observe the result.
- Set a realm display name and logo URL.
- Enable/disable user self-registration and see the effect on the login page.

### Learning Outcomes
- Understand the hierarchy: Keycloak instance → Realm → Client → User.
- Understand that the master realm is for administration only — never use it for application users.
- Recognize how login flows enforce credential policies.

### Blog Post Angle
*"Keycloak from Zero: Setting Up Your First Identity Realm"* — walk readers through the mental model of realms and clients, why isolation matters, and how Keycloak compares to hosted alternatives like Auth0 or Okta.

---

## Lab 2 — Username/Password Authentication & Brute-Force Protection

### Goal
Understand how Keycloak handles credential validation, failed login attempts, account lockouts, and security policies around passwords.

### Tools Required
- Keycloak Admin Console
- Browser (incognito recommended)
- Postman or Bruno (for ROPC token endpoint testing)

### Key IAM Concepts
- **Credential Policy**: Rules governing password length, complexity, expiry, and history.
- **Brute-Force Detection**: Rate limiting and temporary lockout after repeated failures.
- **Account Lockout**: Temporarily disabling a user account after N failed attempts.
- **Resource Owner Password Credentials (ROPC)**: An OAuth 2.0 grant type that accepts username/password directly (legacy; useful for testing).

### Step-by-Step Outline
1. Configure password policy: minimum 8 characters, 1 uppercase, 1 digit, no repeat last 3.
2. Enable Brute Force Protection in Realm Settings → Security Defenses.
3. Set: max login failures = 3, wait increment = 30 seconds.
4. Attempt login with wrong password 3 times and observe the lockout behavior.
5. As admin, unlock the account manually in Users → `testuser` → Credentials.
6. Use Postman to call the ROPC token endpoint and get an access token:
   ```
   POST http://localhost:8080/realms/lab-realm/protocol/openid-connect/token
   Body: grant_type=password&client_id=my-web-app&username=testuser&password=...
   ```

### Lab Exercises
- Enable "Require SSL" and observe how Keycloak handles plain HTTP requests.
- Set password expiry to 1 day and trigger the forced reset flow.
- Test the difference in error messages with and without "prevent information disclosure" enabled.

### Learning Outcomes
- How Keycloak decouples credential storage from application logic.
- The security implications of ROPC vs. redirect-based flows.
- How brute-force protection works at the identity provider level.

### Blog Post Angle
*"Password Security at the Identity Layer: Policies, Lockouts, and Why ROPC Is Dangerous"* — discuss why centralizing credential policy in an IdP is better than per-app password validation.

---

## Lab 3 — Multi-Factor Authentication (TOTP & WebAuthn)

### Goal
Configure and test two forms of MFA: Time-based One-Time Passwords (TOTP/HOTP) and WebAuthn (passkeys / hardware security keys). Understand how Keycloak enforces MFA conditionally.

### Tools Required
- Google Authenticator, Aegis (Android), or Authy (TOTP apps)
- Chrome or Edge (for WebAuthn/passkey support)
- Keycloak Admin Console

### Key IAM Concepts
- **MFA / 2FA**: Adding a second verification factor beyond the password (something you know + something you have).
- **TOTP (RFC 6238)**: Time-based one-time passwords — shared secret generates a 6-digit code every 30 seconds.
- **WebAuthn (FIDO2)**: Browser-native authentication using cryptographic key pairs stored on device or security key.
- **Authentication Required Action**: A Keycloak mechanism that forces users to perform an action (like enrolling MFA) on next login.
- **Conditional MFA**: Enforcing MFA only for certain users, roles, or contexts (e.g., admin users always, standard users optionally).

### Step-by-Step Outline
1. **TOTP Setup:**
   - In Authentication → Policies → OTP Policy, review TOTP settings (algorithm, period, digits).
   - Assign the `Configure OTP` required action to `testuser`.
   - Log in as `testuser` and scan the QR code with your authenticator app.
   - Verify subsequent logins require the OTP code.

2. **Conditional MFA Flow:**
   - Go to Authentication → Flows → Browser.
   - Duplicate the flow, add a Condition – User Role sub-flow requiring OTP only for users with role `premium`.
   - Bind the new flow as the realm's browser flow.

3. **WebAuthn Setup:**
   - In Authentication → Policies → WebAuthn Policy, configure for platform authenticators.
   - Add the WebAuthn Authenticator as an alternative factor.
   - Register a passkey using Chrome on your local machine.
   - Log in using the passkey.

### Lab Exercises
- Set MFA as required for all users via Required Actions (realm default).
- Test the grace period for TOTP (how many clock drift steps Keycloak tolerates).
- Simulate a user losing their TOTP device: remove credentials as admin and re-enroll.

### Learning Outcomes
- The difference between TOTP (shared secret) and WebAuthn (asymmetric cryptography).
- How Keycloak authentication flows are modular and composable.
- The operational challenge of MFA recovery and admin credential management.

### Blog Post Angle
*"MFA Deep Dive: TOTP vs. WebAuthn — How Keycloak Implements Both"* — explain TOTP's RFC 6238 mechanics, WebAuthn's key pair model, and the UX/security tradeoffs of each.

---

## Lab 4 — OAuth 2.0 Authorization Code Flow

### Goal
Build a minimal web application that uses the OAuth 2.0 Authorization Code Flow to obtain an access token from Keycloak. Inspect each step of the redirect-based handshake.

### Tools Required
- Node.js (`express`, `express-session`, `openid-client` npm packages)
- Browser with developer tools open
- Postman (for token introspection)

### Key IAM Concepts
- **OAuth 2.0**: An authorization framework that allows apps to obtain limited access to user accounts. Keycloak is the Authorization Server (AS).
- **Authorization Code**: A short-lived, one-time code returned to the client after user consent. Exchanged server-side for tokens.
- **Access Token**: A credential (usually a JWT) that grants access to a resource.
- **Client ID / Client Secret**: Credentials identifying the registered application to the AS.
- **Redirect URI**: The callback URL where the authorization code is delivered.
- **State Parameter**: A random value to prevent CSRF attacks during the OAuth handshake.
- **Scope**: What access is being requested (e.g., `openid profile email`).

### Step-by-Step Outline
1. Create a `confidential` client in Keycloak: `webapp-confidential`.
2. Set redirect URI: `http://localhost:3000/callback`.
3. Scaffold a minimal Express app:
   ```bash
   npm init -y && npm install express express-session openid-client
   ```
4. Configure the OIDC client using Keycloak's discovery URL:
   ```
   http://localhost:8080/realms/lab-realm/.well-known/openid-configuration
   ```
5. Implement `/login` → redirect to Keycloak authorization endpoint.
6. Implement `/callback` → exchange code for tokens.
7. Display the decoded access token payload on a `/profile` page.
8. Use browser DevTools Network tab to observe every redirect step.

### Lab Exercises
- Add `state` parameter manually and validate it in the callback.
- Request different scopes and observe the token claims change.
- Try reusing an authorization code — observe Keycloak's one-time-use enforcement.
- Shorten the access token lifespan to 60 seconds and test expiry behavior.

### Learning Outcomes
- Understand every step of the 5-leg OAuth dance (user → client → AS → user → client → resource).
- Why the authorization code is exchanged server-side (keeping tokens out of the browser).
- The role of state, nonce, and redirect URI validation in preventing attacks.

### Blog Post Angle
*"OAuth 2.0 Authorization Code Flow, Explained Step-by-Step with Keycloak"* — walk through each HTTP redirect with real request/response examples. Great for developers new to OAuth.

---

## Lab 5 — OAuth 2.0 Client Credentials Flow

### Goal
Implement machine-to-machine (M2M) authentication where a backend service obtains an access token using its own credentials, with no user involved.

### Tools Required
- Postman or curl
- Python (optional — for a simulated microservice)

### Key IAM Concepts
- **Client Credentials Grant**: An OAuth 2.0 flow where the client authenticates directly with the AS using client_id + client_secret. No user context.
- **Service Account**: A Keycloak-generated user account automatically created for each confidential client, used to hold roles for M2M scenarios.
- **Machine Identity**: Representing a non-human entity (a service, daemon, or job) in IAM.
- **Audience Claim (`aud`)**: Restricts which resource servers can accept a token.

### Step-by-Step Outline
1. Create a new confidential client: `billing-service`.
2. Enable "Service Accounts Enabled" in client settings.
3. Assign a realm role `billing-reader` to the service account user.
4. Request a token using curl:
   ```bash
   curl -X POST http://localhost:8080/realms/lab-realm/protocol/openid-connect/token \
     -d "grant_type=client_credentials" \
     -d "client_id=billing-service" \
     -d "client_secret=<secret>"
   ```
5. Decode the JWT at jwt.io and confirm `sub` is the service account, not a human user.
6. Create a second service `inventory-service` and verify its token has different claims.
7. Write a Python script that refreshes the token before it expires.

### Lab Exercises
- Rotate the client secret and update the script — observe authentication failure and recovery.
- Scope the access token to a specific audience using Keycloak's Audience mapper.
- Set a very short token TTL (60s) and implement auto-refresh in your client.

### Learning Outcomes
- When to use client credentials vs. authorization code (user context vs. no user context).
- How service accounts provide an identity layer for non-human actors.
- Why M2M tokens should have minimal scope and short lifetimes.

### Blog Post Angle
*"Machine Identity 101: OAuth 2.0 Client Credentials and Service Accounts in Keycloak"* — explain M2M auth patterns, contrast with API keys, and cover token rotation best practices.

---

## Lab 6 — OpenID Connect (OIDC) — ID Tokens & UserInfo Endpoint

### Goal
Understand how OIDC extends OAuth 2.0 with identity assertions. Work with ID tokens, the UserInfo endpoint, and the discovery document.

### Tools Required
- Postman or Bruno
- jwt.io
- Node.js or Python client from Lab 4

### Key IAM Concepts
- **OpenID Connect (OIDC)**: An identity layer on top of OAuth 2.0. Adds the ID token and UserInfo endpoint to prove *who* authenticated.
- **ID Token**: A JWT issued alongside the access token, asserting identity claims (sub, name, email, etc.) signed by the IdP.
- **UserInfo Endpoint**: A protected endpoint returning user claims, fetched using the access token.
- **Discovery Document (`.well-known`)**: A JSON document describing the IdP's capabilities, endpoints, and public keys.
- **JWKS (JSON Web Key Set)**: The IdP's public keys, used by clients to verify token signatures.
- **Nonce**: A random value embedded in the auth request and ID token to prevent replay attacks.

### Step-by-Step Outline
1. Using the app from Lab 4, log in and capture both the `id_token` and `access_token`.
2. Decode the ID token at jwt.io. Identify: `iss`, `sub`, `aud`, `exp`, `iat`, `nonce`.
3. Fetch Keycloak's discovery document:
   ```
   GET http://localhost:8080/realms/lab-realm/.well-known/openid-configuration
   ```
4. Call the UserInfo endpoint with the access token:
   ```bash
   curl -H "Authorization: Bearer <access_token>" \
     http://localhost:8080/realms/lab-realm/protocol/openid-connect/userinfo
   ```
5. Fetch Keycloak's JWKS and manually verify the ID token signature using the public key.
6. Compare claims in the ID token vs. the UserInfo response.

### Lab Exercises
- Add `email`, `profile`, and `address` scopes and see new claims appear.
- Set `nonce` in the authorization request and verify it appears in the ID token.
- Simulate token replay: reuse an expired ID token and verify verification fails.

### Learning Outcomes
- The difference between authentication (OIDC) and authorization (OAuth 2.0).
- Why ID tokens are verified client-side while access tokens are validated by the resource server.
- How JWKS-based signature verification enables stateless token validation.

### Blog Post Angle
*"OIDC vs. OAuth 2.0: What's the Difference and How Keycloak Implements Both"* — a concept-heavy post clarifying one of the most common confusions in IAM.

---

## Lab 7 — Role-Based Access Control (RBAC)

### Goal
Implement RBAC using Keycloak realm roles and client roles. Protect API endpoints so only users with the correct role can access them.

### Tools Required
- Node.js (Express API — a simple two-endpoint app)
- Postman
- Keycloak Admin Console

### Key IAM Concepts
- **RBAC**: Access control where permissions are grouped into roles and roles are assigned to users.
- **Realm Role**: A role defined at the realm level, available to all clients.
- **Client Role**: A role scoped to a specific client application.
- **Composite Role**: A role that includes other roles — useful for role hierarchies.
- **`realm_access` / `resource_access` claims**: JWT claims containing the user's assigned roles.

### Step-by-Step Outline
1. Create realm roles: `admin`, `editor`, `viewer`.
2. Create composite role `editor` that includes `viewer`.
3. Assign `admin` to `testuser`. Create `readeruser` and assign `viewer`.
4. Build a Node.js Express API with three endpoints:
   - `GET /public` — no auth required
   - `GET /data` — requires `viewer` role
   - `DELETE /data/:id` — requires `admin` role
5. Implement JWT middleware that:
   - Fetches Keycloak's JWKS to verify token signatures.
   - Checks `realm_access.roles` for the required role.
6. Test with tokens from both users.

### Lab Exercises
- Move roles to client-level (`resource_access`) and update the middleware.
- Implement a role hierarchy where `admin` inherits all `editor` permissions.
- Add a `moderator` composite role with `editor` + custom permissions.
- Test the token from Lab 5 (service account) against the API.

### Learning Outcomes
- The architectural difference between realm roles and client roles.
- How roles are encoded in JWTs and why fine-grained permissions should not all go in tokens.
- The limitation of RBAC: role explosion and why ABAC complements it.

### Blog Post Angle
*"RBAC with Keycloak: Realm Roles, Client Roles, and Protecting Your APIs"* — practical tutorial with full code examples for Express middleware.

---

## Lab 8 — Attribute-Based Access Control (ABAC) with Claims

### Goal
Extend RBAC with attribute-based policies where access decisions depend on user attributes (e.g., department, subscription tier, country) encoded as JWT claims.

### Tools Required
- Keycloak Admin Console
- Node.js API from Lab 7
- Postman

### Key IAM Concepts
- **ABAC**: Access control based on attributes of the user, resource, and environment — more flexible than RBAC.
- **User Attribute**: Custom key-value data attached to a Keycloak user (e.g., `department=engineering`).
- **Protocol Mapper**: A Keycloak component that maps user data (attributes, roles, group membership) into token claims.
- **Custom Claim**: A non-standard claim added to a JWT by a protocol mapper.
- **Policy as Code**: Evaluating access decisions based on declarative rules over claims.

### Step-by-Step Outline
1. Add user attributes to `testuser`: `department=engineering`, `subscription=premium`, `region=us`.
2. Create a User Attribute Protocol Mapper on the `my-web-app` client:
   - Map `department` → claim `department` in access token.
   - Map `subscription` → claim `subscription_tier`.
3. Log in as `testuser` and decode the token — verify the new claims appear.
4. Update the Express API to enforce ABAC:
   - `GET /premium-features` — requires `subscription_tier == premium`.
   - `GET /engineering-tools` — requires `department == engineering`.
5. Create `basicuser` with `subscription=free` and verify access is denied.

### Lab Exercises
- Combine RBAC + ABAC: require role `viewer` AND `region=us`.
- Use a JavaScript Protocol Mapper to compute a derived claim (e.g., `is_premium: true/false`).
- Implement environment-based ABAC: restrict access outside business hours using a server-side time check.

### Learning Outcomes
- Why ABAC enables more granular, context-aware access control than RBAC alone.
- How protocol mappers bridge user data and token claims.
- The tradeoff: putting too much in tokens vs. calling the IdP at runtime.

### Blog Post Angle
*"Beyond Roles: Implementing ABAC with Custom JWT Claims in Keycloak"* — show how user attributes flow from the directory into tokens into access decisions.

---

## Lab 9 — Single Sign-On (SSO) Across Multiple Apps

### Goal
Configure two separate applications to share a Keycloak session, demonstrating SSO so that logging into one app automatically authenticates the user in the other.

### Tools Required
- Node.js (two separate Express apps on different ports: 3000 and 3001)
- Browser (watch cookies between tabs)

### Key IAM Concepts
- **SSO**: A user authenticates once with the IdP and gains access to all connected applications without re-authenticating.
- **Keycloak Session**: Server-side state at the IdP recording the authenticated user, session expiry, and associated clients.
- **SSO Cookie**: A browser cookie (`KEYCLOAK_SESSION`) that carries the session identity across redirects.
- **Session Timeout vs. Token Lifetime**: The IdP session can outlive any individual access token.
- **Silent Authentication / Prompt=none**: An OIDC technique allowing an app to obtain tokens without user interaction if an SSO session exists.

### Step-by-Step Outline
1. Register two clients in Keycloak: `app-alpha` (port 3000) and `app-beta` (port 3001).
2. Build a minimal OIDC login flow in both apps.
3. Log in through `app-alpha` → you are redirected to Keycloak → you authenticate.
4. Navigate to `app-beta` in the same browser → you are redirected to Keycloak → you are **not** prompted for credentials.
5. Verify in Admin Console → Sessions that one session serves both clients.
6. Observe the `KEYCLOAK_SESSION` cookie in browser DevTools.

### Lab Exercises
- Open `app-beta` in a private/incognito window — confirm SSO does NOT carry over.
- Change the SSO session idle timeout to 2 minutes and observe automatic logout.
- Implement `prompt=login` in `app-beta` to force re-authentication even with an active session.
- Test cross-tab SSO vs. cross-browser (different user agents) behavior.

### Learning Outcomes
- How SSO is mediated by the IdP session, not by each application.
- The role of browser cookies in SSO and why SSO doesn't work across different IdP domains without federation.
- How `prompt` parameter gives applications control over the SSO experience.

### Blog Post Angle
*"How SSO Actually Works: Keycloak Sessions, Cookies, and the Silent Auth Flow"* — demystify SSO by walking through the HTTP-level mechanics with annotated screenshots.

---

## Lab 10 — Single Logout (SLO)

### Goal
Implement and test Single Logout so that logging out of one application terminates the SSO session and logs the user out of all connected apps.

### Tools Required
- Both apps from Lab 9
- Browser DevTools (Network tab)
- Postman (for back-channel logout testing)

### Key IAM Concepts
- **Single Logout (SLO)**: The counterpart to SSO — a logout in one app propagates to all apps sharing the session.
- **Front-Channel Logout**: The IdP redirects the user's browser to each app's logout endpoint sequentially (visible to the user).
- **Back-Channel Logout**: The IdP sends an HTTP POST directly to each app's logout endpoint server-to-server (invisible to the user, more reliable).
- **Logout Token**: A signed JWT sent in back-channel logout notifications, containing session and user identifiers.
- **Post-Logout Redirect URI**: Where the user is sent after the IdP confirms logout.

### Step-by-Step Outline
1. **Front-Channel Logout:**
   - Configure logout redirect URI on both clients.
   - Trigger logout from `app-alpha` via:
     ```
     GET http://localhost:8080/realms/lab-realm/protocol/openid-connect/logout?id_token_hint=<id_token>
     ```
   - Observe browser redirect to `app-beta`'s logout URL.
   - Verify the SSO session is terminated in Admin Console.

2. **Back-Channel Logout:**
   - Enable "Back-Channel Logout" in `app-beta` client settings.
   - Set back-channel logout URL: `http://host.docker.internal:3001/backchannel-logout`.
   - Implement the logout endpoint in `app-beta` to accept and validate the logout token.
   - Trigger logout from `app-alpha` and observe the silent POST to `app-beta`.

### Lab Exercises
- Simulate a back-channel logout failure (shut down app-beta) and check Keycloak's retry behavior.
- Implement in-memory session invalidation in the app on receiving a logout token.
- Test admin-initiated logout (terminate session from Admin Console) and verify SLO fires.

### Learning Outcomes
- Why front-channel logout is unreliable (popup blockers, network errors) vs. back-channel.
- How to implement a secure back-channel logout endpoint (validate logout token signature).
- Session management best practices: short-lived tokens + server-side session revocation.

### Blog Post Angle
*"Single Logout Is Hard: Front-Channel vs. Back-Channel SLO with Keycloak"* — a pragmatic post covering the reliability pitfalls of SLO and how back-channel logout solves them.

---

## Lab 11 — Social Login with Google OAuth

### Goal
Enable users to log into your Keycloak realm using their Google account. Configure identity linking so existing accounts can connect a social identity.

### Tools Required
- Google Cloud Console account (free) — [console.cloud.google.com](https://console.cloud.google.com)
- ngrok (to expose localhost to Google's redirect validation)
- Keycloak Admin Console

### Key IAM Concepts
- **Social Login**: Using a consumer IdP (Google, GitHub, Apple) as the identity provider, brokered through Keycloak.
- **Identity Brokering**: Keycloak acts as an SP (Service Provider) to external IdPs while being an IdP to its own clients.
- **Account Linking**: Connecting an externally authenticated identity to a local Keycloak account.
- **First Login Flow**: The authentication flow executed the first time a user logs in via a social provider (decide: create new account, link existing, require email verification).
- **Federated Identity**: A user identity sourced from an external system, not locally managed.

### Step-by-Step Outline
1. Create a Google OAuth 2.0 Client in Google Cloud Console:
   - Application type: Web Application.
   - Authorized redirect URI: `http://localhost:8080/realms/lab-realm/broker/google/endpoint`.
   - Copy Client ID and Secret.
2. In Keycloak → Identity Providers → Add → Google.
   - Paste Client ID and Secret.
   - Enable "Trust Email" if you want to skip email verification for Google accounts.
3. Start ngrok to expose Keycloak (needed for Google's redirect):
   ```bash
   ngrok http 8080
   ```
   Update the Google redirect URI with the ngrok URL.
4. Visit the Keycloak login page — a "Login with Google" button appears.
5. Complete the Google login flow and observe the new federated user in Admin Console.
6. Inspect the user's "Identity Provider Links" tab.

### Lab Exercises
- Configure the "First Broker Login" flow to require email verification.
- Disable automatic account linking and require users to manually link accounts.
- Map Google's `given_name` and `picture` claims to Keycloak user attributes using Identity Provider Mappers.
- Set a default role assigned to all Google-authenticated users.

### Learning Outcomes
- How Keycloak translates external OIDC tokens into local user sessions.
- The importance of the first-login flow for account merge security.
- Why "Trust Email" from social providers carries risk (Google doesn't verify email ownership in the same way as a corporate IdP).

### Blog Post Angle
*"Adding Google Login to Any App with Keycloak Identity Brokering"* — step-by-step with Google Cloud Console screenshots. Explain the security implications of trusting social email addresses.

---

## Lab 12 — Social Login with GitHub OAuth

### Goal
Configure GitHub as a social identity provider, demonstrating OAuth 2.0 brokering with a provider that uses scopes to control data access.

### Tools Required
- GitHub account (free) — [github.com/settings/developers](https://github.com/settings/developers)
- ngrok (same as Lab 11)

### Key IAM Concepts
- **GitHub as OAuth Provider**: GitHub implements OAuth 2.0 (not full OIDC) — it issues access tokens but not ID tokens. Keycloak bridges the gap.
- **Organization Membership as a Policy**: Restricting login to members of a specific GitHub organization.
- **Scope-Based Claim Extraction**: Using GitHub scopes (`read:user`, `user:email`) to fetch identity data.

### Step-by-Step Outline
1. Create a GitHub OAuth App at `github.com/settings/developers`.
   - Homepage URL: `http://localhost:8080`
   - Callback URL: `http://localhost:8080/realms/lab-realm/broker/github/endpoint`
2. In Keycloak → Identity Providers → Add → GitHub.
3. Paste GitHub Client ID and Secret.
4. Map GitHub's `login` to a Keycloak username attribute.
5. Log in via GitHub and inspect the resulting user in Keycloak.

### Lab Exercises
- Restrict login to members of a specific GitHub organization using a custom first-login authenticator (or a script-based flow).
- Map GitHub's `avatar_url` to a user attribute and display it in the Account Console.
- Compare the difference in flow between GitHub (OAuth 2.0 only) vs. Google (OIDC) at the protocol level.

### Learning Outcomes
- How Keycloak handles non-OIDC providers by wrapping OAuth 2.0 with user profile API calls.
- Why relying on GitHub organization membership for access control has limitations.

### Blog Post Angle
*"GitHub Login with Keycloak: OAuth 2.0 Brokering Without OIDC"* — explain what Keycloak does behind the scenes when the upstream IdP is pure OAuth 2.0 rather than OIDC.

---

## Lab 13 — SAML 2.0 Service Provider Integration

### Goal
Configure Keycloak as a SAML Identity Provider and integrate a SAML Service Provider application. Inspect SAML assertions and understand the XML-based federation protocol.

### Tools Required
- [SimpleSAMLphp](https://simplesamlphp.org/) or [SAMLtest.id](https://samltest.id) (free online SP tester)
- Docker (for SimpleSAMLphp)
- XML viewer / browser extension
- Postman

### Key IAM Concepts
- **SAML 2.0**: An XML-based open standard for exchanging authentication and authorization data between an IdP and an SP.
- **Identity Provider (IdP)**: Issues SAML assertions (Keycloak in this case).
- **Service Provider (SP)**: Relies on the IdP's assertions to grant access (your application).
- **SAML Assertion**: A signed XML document containing the user's identity and attributes.
- **SSO Binding**: How SAML messages are transported — HTTP POST (form submission) or HTTP Redirect (URL encoding).
- **Metadata Exchange**: IdP and SP share XML metadata documents to establish trust.
- **Attribute Mapping**: Mapping Keycloak user attributes to SAML attribute statements.

### Step-by-Step Outline
1. Download Keycloak's SAML IdP metadata:
   ```
   http://localhost:8080/realms/lab-realm/protocol/saml/descriptor
   ```
2. Use SAMLtest.id as the SP:
   - Visit `https://samltest.id/` and upload Keycloak's IdP metadata.
   - Download SAMLtest's SP metadata.
3. In Keycloak → Clients → Create → import SAMLtest's SP metadata.
4. Configure attribute mappers: map `email`, `firstName`, `lastName` to SAML attributes.
5. Initiate SP-initiated SSO from SAMLtest.id and complete authentication.
6. Inspect the SAML assertion using SAMLtest's assertion viewer.
7. Try IdP-initiated SSO from Keycloak's account console.

### Lab Exercises
- Configure SAML assertion encryption (EncryptedAssertion).
- Set up SAML attribute-based access using `Role` as a SAML attribute.
- Implement SAML SLO (Single Logout) with the SP.
- Compare SAML POST binding vs. Redirect binding performance and security.

### Learning Outcomes
- How SAML's XML-signature model compares to JWT-based OIDC tokens.
- Why SAML is still dominant in enterprise (Okta, ADFS) while OIDC dominates modern apps.
- The complexity cost of SAML vs. OIDC for new integrations.

### Blog Post Angle
*"SAML 2.0 with Keycloak: Assertions, Bindings, and Why Enterprises Still Use It"* — decode a real SAML assertion, explain the XML signature, and contrast with OIDC JWTs.

---

## Lab 14 — Identity Brokering & External IdP Federation

### Goal
Configure Keycloak to broker authentication through a second Keycloak instance acting as an upstream IdP. Demonstrate chained identity federation and claim transformation across IdP hops.

### Tools Required
- Two Keycloak instances on different ports (e.g., 8080 and 8180 via Docker)
- Browser
- Postman

### Key IAM Concepts
- **Identity Federation**: Trusting an external IdP's authentication assertions without managing the user's credentials locally.
- **IdP Chaining**: A Keycloak instance (SP role) brokering through another Keycloak (IdP role).
- **Claim Transformation**: Mapping, filtering, or enriching claims as they pass through the broker.
- **Home Realm Discovery**: Automatically routing users to the correct upstream IdP based on email domain or hint.
- **Trust Hierarchy**: The chain of trust from the resource server → local IdP → upstream IdP.

### Step-by-Step Outline
1. Run a second Keycloak on port 8180:
   ```bash
   docker run -p 8180:8080 \
     -e KC_BOOTSTRAP_ADMIN_USERNAME=admin \
     -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin \
     quay.io/keycloak/keycloak:latest start-dev
   ```
2. In the upstream Keycloak (8180): create realm `corp-idp`, a user, and an OIDC client for the downstream broker.
3. In the downstream Keycloak (8080): add an OIDC Identity Provider pointing to `http://localhost:8180/realms/corp-idp/.well-known/openid-configuration`.
4. Configure Home Realm Discovery: users with `@corp.example` email → route to upstream IdP.
5. Log in to an app registered at the downstream Keycloak using a `@corp.example` user.
6. Observe the claim flow: upstream token → downstream broker → new token for the app.

### Lab Exercises
- Add an Identity Provider Mapper that enriches the brokered token with a hardcoded `partner: true` claim.
- Block specific email domains from federated login.
- Configure the broker to require re-authentication every time (disable SSO at the broker layer).

### Learning Outcomes
- How enterprise SSO works across organizational boundaries (B2B identity federation).
- The security responsibility at each layer of an IdP chain.
- How Keycloak's broker rewrites tokens while preserving the original `sub` or creating a local one.

### Blog Post Angle
*"B2B Identity Federation: Chaining Keycloak as Both IdP and SP"* — a practical guide to partner/vendor identity integration patterns.

---

## Lab 15 — User Federation with LDAP (OpenLDAP)

### Goal
Connect Keycloak to an OpenLDAP directory, syncing users and groups. Authenticate via LDAP credentials while using Keycloak's session and token management.

### Tools Required
- [osixia/openldap](https://github.com/osixia/docker-openldap) Docker image (free)
- [phpLDAPadmin](https://github.com/osixia/docker-phpldapadmin) (browser-based LDAP admin)
- Keycloak Admin Console

### Key IAM Concepts
- **LDAP (Lightweight Directory Access Protocol)**: An open protocol for accessing directory services — the backbone of most enterprise user directories (Active Directory is an LDAP implementation).
- **User Federation**: Keycloak delegates user storage and credential validation to an external directory.
- **Sync Modes**: Full sync (import all users) vs. No sync (validate credentials on-demand without importing).
- **LDAP Mapper**: Maps LDAP attributes to Keycloak user attributes (e.g., `sn` → `lastName`).
- **Kerberos Integration**: Using Kerberos tickets for LDAP-backed SSO (advanced extension of this lab).

### Step-by-Step Outline
1. Start OpenLDAP:
   ```bash
   docker run --name ldap \
     -e LDAP_ORGANISATION="Lab Corp" \
     -e LDAP_DOMAIN="lab.local" \
     -e LDAP_ADMIN_PASSWORD="adminpass" \
     -p 389:389 osixia/openldap:latest
   ```
2. Start phpLDAPadmin on port 8090 and create organizational units and users.
3. In Keycloak → User Federation → Add → ldap.
4. Configure: connection URL `ldap://host.docker.internal:389`, Bind DN, Users DN.
5. Set Edit Mode to `READ_ONLY` first, then test with `WRITABLE`.
6. Click "Synchronize All Users" and verify users appear in Keycloak.
7. Log in with an LDAP user's credentials.

### Lab Exercises
- Configure LDAP group sync and map LDAP groups to Keycloak roles.
- Set up periodic sync (every 60 seconds) and add a new user to LDAP — watch it appear in Keycloak.
- Enable TLS/LDAPS for the LDAP connection.
- Test what happens when LDAP goes offline — can users with cached credentials still log in?

### Learning Outcomes
- How Keycloak sits in front of an existing directory without replacing it.
- The security implications of Keycloak storing LDAP bind credentials.
- The difference between LDAP as an authentication source vs. LDAP as a user store.

### Blog Post Angle
*"Connecting Keycloak to an Enterprise LDAP Directory"* — a practical post for organizations migrating from legacy LDAP-based auth to a modern IdP layer.

---

## Lab 16 — SCIM 2.0 User Provisioning

### Goal
Implement automated user provisioning using the SCIM 2.0 standard. Provision users from an external HR system (simulated) into Keycloak, and explore the Create-Read-Update-Delete (CRUD) provisioning lifecycle.

### Tools Required
- [Keycloak SCIM plugin](https://github.com/Captain-P-Goldfish/scim-for-keycloak) (open source, free) — or use Keycloak's built-in SCIM support (Keycloak 26+)
- Python or Node.js (to simulate the provisioning client / HR system)
- Postman

### Key IAM Concepts
- **SCIM 2.0 (System for Cross-domain Identity Management)**: An open standard (RFC 7642/7643/7644) for automating user provisioning across systems.
- **Provisioning vs. Authentication**: SCIM handles *who exists* in the system; OAuth/OIDC handles *who can log in*.
- **Just-in-Time (JIT) Provisioning**: Creating users on first login (contrast with pre-provisioning via SCIM).
- **Deprovisioning**: Disabling or deleting user accounts when they leave an organization — a critical security control.
- **SCIM Schema**: Standardized user attributes (`userName`, `emails`, `name`, `active`, `groups`).

### Step-by-Step Outline
1. Install and configure the SCIM for Keycloak extension.
2. Enable the SCIM endpoint for your realm and create an OAuth client for the provisioning agent.
3. Write a Python script to provision a user via SCIM:
   ```python
   import requests
   # POST /scim/v2/Users
   payload = {
     "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
     "userName": "alice",
     "name": {"givenName": "Alice", "familyName": "Smith"},
     "emails": [{"value": "alice@lab.local", "primary": True}],
     "active": True
   }
   ```
4. Verify the user appears in Keycloak Admin Console.
5. Update the user via SCIM PATCH (change email address).
6. Deprovision: set `active: false` via SCIM and verify Keycloak account is disabled.
7. Delete the user via SCIM DELETE.

### Lab Exercises
- Provision a group via SCIM and verify Keycloak group is created.
- Implement SCIM sync with a mock CSV-based HR system (read a CSV, provision via SCIM).
- Test what happens if SCIM provisioning fails midway (partial failure handling).
- Compare SCIM provisioning latency vs. LDAP sync.

### Learning Outcomes
- How SCIM decouples the identity lifecycle from authentication.
- Why deprovisioning is the hardest (and most critical) part of the identity lifecycle.
- How SCIM compares to proprietary HR system connectors (Workday, BambooHR).

### Blog Post Angle
*"Automated User Provisioning with SCIM 2.0 and Keycloak"* — explain why manual user creation doesn't scale and how SCIM solves the provisioning lifecycle problem.

---

## Lab 17 — Custom Authentication Flow (OTP via Email)

### Goal
Build a custom authentication flow in Keycloak that sends a one-time passcode to the user's email address as a second factor, replacing TOTP.

### Tools Required
- [Mailhog](https://github.com/mailhog/MailHog) (free local SMTP server + web UI)
- Keycloak Admin Console
- Keycloak SPI (Service Provider Interface) — Java or a script authenticator

### Key IAM Concepts
- **Authentication SPI**: Keycloak's extension point for building custom authentication steps in Java.
- **Script Authenticator**: A lighter-weight option to write authenticator logic in JavaScript within Keycloak (requires enabling the preview feature).
- **Required Action**: A mandatory step a user must complete during login.
- **Email as Second Factor**: Sending OTPs via email — convenient but lower assurance than TOTP (email account is a second attack surface).
- **Authentication Context**: The state passed between steps in an authentication flow.

### Step-by-Step Outline
1. Start Mailhog:
   ```bash
   docker run -p 1025:1025 -p 8025:8025 mailhog/mailhog
   ```
2. Configure Keycloak's email settings to use Mailhog as SMTP (host: `localhost`, port: `1025`).
3. Enable Script Authenticator preview feature in Keycloak.
4. Create a custom authentication flow:
   - Step 1: Username + Password (standard).
   - Step 2: Script authenticator that generates a 6-digit OTP, stores it in session notes, and emails it via Keycloak's email service.
   - Step 3: A form that accepts the OTP and validates against the session note.
5. Bind the custom flow as the realm's browser flow.
6. Log in, receive OTP in Mailhog, enter it, and complete auth.

### Lab Exercises
- Add OTP expiry (reject codes older than 5 minutes).
- Rate limit OTP resend requests (max 3 per session).
- Compare security posture: email OTP vs. TOTP vs. WebAuthn.
- Implement a "Remember this device" feature using a browser cookie.

### Learning Outcomes
- How Keycloak's flow engine is extensible via SPI.
- The security limitations of email OTP compared to app-based TOTP.
- How to build stateful multi-step authentication using session notes.

### Blog Post Angle
*"Building a Custom Email OTP Flow in Keycloak"* — hands-on Java/script tutorial for extending Keycloak's authentication with custom logic.

---

## Lab 18 — Passwordless Authentication (Magic Link / WebAuthn)

### Goal
Implement two forms of passwordless authentication: email magic links (login via a single-use link) and WebAuthn passkeys (biometric-backed cryptographic authentication).

### Tools Required
- Mailhog (from Lab 17)
- Chrome or Edge (for WebAuthn/passkey support)
- [keycloak-magic-link](https://github.com/p2-inc/keycloak-magic-link) extension (open source, free)

### Key IAM Concepts
- **Passwordless Authentication**: Proving identity without a memorized secret — using possession (passkey device) or a trusted channel (email link).
- **Magic Link**: A single-use, time-limited URL that authenticates the user when clicked — effectively a one-time token delivered via email.
- **Passkey (FIDO2/WebAuthn)**: A cryptographic credential stored on a device (phone, laptop, security key) that authenticates without a password.
- **Phishing Resistance**: Passkeys are bound to the origin (domain) — a fake phishing site cannot use a credential registered on the real site.
- **Discoverable Credentials**: Passkeys that don't require entering a username first — the authenticator presents stored credentials for the site.

### Step-by-Step Outline
1. **Magic Link:**
   - Install the keycloak-magic-link extension.
   - Create a custom flow using the magic link authenticator.
   - Trigger a magic link email → inspect the link format in Mailhog.
   - Click the link → observe automatic authentication.
   - Verify the link cannot be reused (one-time-use enforcement).

2. **WebAuthn Passkeys:**
   - Enable WebAuthn Passwordless Policy in Keycloak.
   - Build a login flow: Email/username only → WebAuthn Passwordless.
   - Register a passkey on Chrome (using the built-in device authenticator).
   - Log in: enter username, press a button, authenticate with biometric/PIN.

### Lab Exercises
- Set magic link TTL to 10 minutes and test expiry.
- Implement passkey-only authentication (no username field — full discoverable credential flow).
- Combine: allow both magic link and passkey as alternatives.
- Test passkey authentication on a mobile device via QR code cross-device flow.

### Learning Outcomes
- Why passwordless reduces phishing risk and credential stuffing attacks.
- The UX tradeoffs between magic links (email dependency) and passkeys (device dependency).
- How WebAuthn's origin binding prevents credential theft by phishing sites.

### Blog Post Angle
*"Goodbye Passwords: Implementing Magic Links and Passkeys with Keycloak"* — compare the threat models each approach addresses and their recovery challenges.

---

## Lab 19 — Fine-Grained Authorization with Keycloak Authorization Services

### Goal
Use Keycloak's built-in Authorization Services to implement policy-based access control (PBAC) with resources, scopes, and policies — evaluated by Keycloak at runtime.

### Tools Required
- Keycloak Admin Console
- Node.js API (to call the token introspection / authorization endpoint)
- Postman

### Key IAM Concepts
- **Keycloak Authorization Services**: An optional module providing a full policy engine within Keycloak — resources, scopes, policies, and permissions.
- **Resource**: A named protected asset (e.g., `document:123`, `api:/reports`).
- **Scope**: An action on a resource (e.g., `read`, `write`, `delete`).
- **Policy**: A rule that evaluates to permit or deny (role-based, JS-based, time-based, aggregated).
- **Permission**: The combination of a resource + scope + policies.
- **UMA 2.0 (User-Managed Access)**: An OAuth 2.0 extension where users can delegate access to their own resources.
- **RPT (Requesting Party Token)**: A token issued after authorization evaluation that carries permission grants.

### Step-by-Step Outline
1. Enable Authorization Services on the `my-web-app` client.
2. Create Resources: `Document`, `Report`, `Admin Panel`.
3. Create Scopes: `view`, `edit`, `delete`.
4. Create Policies:
   - Role Policy: `admin` role → permit.
   - Time Policy: permit only 9AM–6PM UTC.
   - JavaScript Policy: `$evaluation.grant()` if `user.attribute.department == 'finance'`.
5. Create Permissions combining resources + scopes + policies.
6. Use the Policy Evaluation Tool in Admin Console to test decisions.
7. Call the token endpoint with `grant_type=urn:ietf:params:oauth:grant-type:uma-ticket` to get an RPT.
8. Introspect the RPT and see the `authorization.permissions` claim.

### Lab Exercises
- Implement resource-owner-managed sharing via UMA.
- Create an aggregate policy (AND multiple policies).
- Build an API middleware that validates RPT permissions claim instead of role claims.
- Test the performance impact of authorization evaluation at login time.

### Learning Outcomes
- The difference between coarse-grained RBAC in tokens vs. fine-grained PBAC evaluated at runtime.
- When Keycloak Authorization Services is the right tool vs. implementing authorization in application code.
- The UMA 2.0 pattern for user-centric resource sharing.

### Blog Post Angle
*"Fine-Grained Authorization with Keycloak: Resources, Policies, and UMA 2.0"* — a deep-dive for architects evaluating policy-based access control solutions.

---

## Lab 20 — Token Exchange & Delegation

### Goal
Implement OAuth 2.0 Token Exchange (RFC 8693) where Service A exchanges its token for a different token to call Service B on behalf of the user.

### Tools Required
- Two Node.js microservices (Service A and Service B)
- Postman
- Keycloak Admin Console

### Key IAM Concepts
- **Token Exchange (RFC 8693)**: An OAuth 2.0 extension where a client presents one token and receives a different token — different audience, subject, or token type.
- **Impersonation**: A service acting as a specific user (subject token exchange).
- **Delegation**: A service acting on behalf of a user while still identified as itself (`act` claim).
- **Actor Claim (`act`)**: A JWT claim identifying who is acting on behalf of the subject.
- **Token Narrowing**: Exchanging a broad token for a narrower-scoped token for a downstream service.

### Step-by-Step Outline
1. Enable Token Exchange feature flag in Keycloak:
   ```
   --features=token-exchange
   ```
2. Create clients: `service-a` and `service-b`.
3. Grant `service-a` permission to exchange tokens for `service-b`.
4. Obtain a user token for Service A.
5. Service A calls the token endpoint with:
   ```
   grant_type=urn:ietf:params:oauth:grant-type:token-exchange
   subject_token=<user_access_token>
   requested_token_type=urn:ietf:params:oauth:token-type:access_token
   audience=service-b
   ```
6. Decode the returned token — verify `aud=service-b` and `act.sub=service-a`.
7. Service B validates the token and logs the delegation chain.

### Lab Exercises
- Implement impersonation: admin user exchanges token to act as another user.
- Chain three services (A → B → C) with progressive delegation.
- Narrow the scope on exchange: start with `read write delete`, exchange for `read` only.

### Learning Outcomes
- How microservices propagate user context through a call chain securely.
- The difference between token forwarding (just passing the same token) vs. exchange (getting a new scoped token).
- Security controls around impersonation: who can impersonate whom.

### Blog Post Angle
*"OAuth 2.0 Token Exchange: Secure Service-to-Service Delegation with Keycloak"* — a microservices-focused post on propagating identity through complex call chains.

---

## Lab 21 — JWT Customization with Protocol Mappers

### Goal
Master Keycloak's Protocol Mapper system to shape JWT claims for different audiences and use cases — adding, removing, renaming, and computing claims.

### Tools Required
- Keycloak Admin Console
- jwt.io
- Postman

### Key IAM Concepts
- **Protocol Mapper**: A Keycloak component that adds, transforms, or removes data from tokens and UserInfo responses.
- **Claim Types**: Included in ID token, access token, UserInfo, or all three — independently configurable.
- **Scope-Linked Mappers**: Mappers attached to a scope (not a client directly), enabled per-scope.
- **Hardcoded Claim**: A mapper that adds a fixed value to every token.
- **Script Mapper**: Custom JavaScript logic for dynamic claim computation.
- **Token Audience**: Controlling which services a token is valid for via the `aud` claim.

### Step-by-Step Outline
1. Add a **User Attribute Mapper**: map `department` user attribute → `dept` claim in access token only.
2. Add a **Group Membership Mapper**: add all user groups as a `groups` claim.
3. Add a **Hardcoded Claim**: `environment: production` in every token.
4. Add an **Audience Mapper**: add `api-gateway` to `aud` claim.
5. Add a **Script Mapper**:
   ```javascript
   // Compute is_premium based on subscription attribute
   var subscription = user.getAttribute('subscription');
   token.setOtherClaims('is_premium', subscription && subscription[0] === 'premium');
   ```
6. For each mapper, toggle "Add to ID token", "Add to access token", "Add to userinfo" and observe the effect.
7. Use Client Scopes to make mappers opt-in (only included when client requests a specific scope).

### Lab Exercises
- Remove the `email` claim from the access token (keep only in UserInfo) for privacy.
- Rename `sub` claim to `userId` for a legacy application's compatibility.
- Create a mapper that computes a user's full display name from `firstName` + `lastName`.
- Build a scoped mapper that only activates when `scope=billing` is requested.

### Learning Outcomes
- The architecture of Keycloak's mapper pipeline and when each mapper type is appropriate.
- Why tokens should contain only what each recipient needs (principle of minimal token claims).
- How to maintain backward compatibility when adding new claims.

### Blog Post Angle
*"Shaping JWTs: A Complete Guide to Keycloak Protocol Mappers"* — a mapper reference post with examples of every mapper type and when to use each.

---

## Lab 22 — Refresh Token Rotation & Token Revocation

### Goal
Implement and test refresh token rotation for enhanced security, and explore token revocation strategies to invalidate tokens before expiry.

### Tools Required
- Postman or Bruno
- Node.js client (to demonstrate automated refresh flow)

### Key IAM Concepts
- **Refresh Token**: A long-lived credential used to obtain new access tokens without re-authenticating.
- **Refresh Token Rotation**: Each use of a refresh token returns a new refresh token and invalidates the old one — prevents silent replay of stolen refresh tokens.
- **Refresh Token Reuse Detection**: If an already-used refresh token is presented again, Keycloak can invalidate the entire session (indicates theft).
- **Token Introspection (RFC 7662)**: A standard endpoint where a resource server can validate a token and get its metadata.
- **Token Revocation (RFC 7009)**: A standard endpoint to proactively invalidate a token.

### Step-by-Step Outline
1. Enable Refresh Token Rotation in Realm Settings → Tokens.
2. Obtain an initial token set (access + refresh).
3. Use the refresh token to get a new access token:
   ```bash
   curl -X POST http://localhost:8080/realms/lab-realm/protocol/openid-connect/token \
     -d "grant_type=refresh_token&client_id=my-web-app&refresh_token=<old_refresh>"
   ```
4. Observe that a new refresh token is returned and the old one is consumed.
5. Attempt to use the old refresh token again — confirm Keycloak rejects it.
6. **Revocation:** Revoke an access token:
   ```
   POST /realms/lab-realm/protocol/openid-connect/revoke
   token=<access_token>&token_type_hint=access_token
   ```
7. Introspect the revoked token:
   ```
   POST /realms/lab-realm/protocol/openid-connect/token/introspect
   ```
   Verify `active: false` in the response.

### Lab Exercises
- Simulate refresh token theft: capture the original refresh token, use it after rotation, and observe session revocation.
- Implement an auto-refresh loop in Node.js that silently refreshes 60 seconds before expiry.
- Configure different access token TTLs per client (short for public clients, longer for confidential).
- Test offline access tokens (special long-lived refresh tokens for offline scenarios).

### Learning Outcomes
- Why refresh token rotation is a defense-in-depth control against refresh token leakage.
- The difference between token expiry (natural) and token revocation (forced).
- Why stateless JWT validation doesn't catch revoked tokens — and how to address this.

### Blog Post Angle
*"Refresh Token Security: Rotation, Reuse Detection, and Revocation in Keycloak"* — a security-focused post on hardening the token refresh lifecycle.

---

## Lab 23 — PKCE — Securing Public Clients

### Goal
Implement Proof Key for Code Exchange (PKCE) to protect the authorization code flow for public clients (SPAs, mobile apps) that cannot securely store a client secret.

### Tools Required
- Browser (or a minimal SPA — plain HTML + vanilla JS)
- Browser DevTools (Network tab)
- Postman

### Key IAM Concepts
- **Public Client**: A client that cannot maintain confidentiality of credentials (browser JS, mobile apps). Has no client secret.
- **Authorization Code Interception Attack**: A threat where a malicious app intercepts the authorization code before the legitimate app can exchange it.
- **PKCE (RFC 7636)**: A mechanism where the client creates a random `code_verifier`, hashes it to a `code_challenge`, and sends the hash in the auth request. The verifier is presented in the token exchange — proving only the original client can use the code.
- **Code Verifier**: A cryptographically random string generated per-request.
- **Code Challenge Method**: `S256` (SHA-256 hash, recommended) or `plain` (not recommended).

### Step-by-Step Outline
1. Set `my-web-app` client to Public (remove client secret).
2. Enable "Proof Key for Code Exchange Code Challenge Method: S256" in client settings.
3. Build a plain HTML page that:
   - Generates a random `code_verifier` (64 random bytes, base64url-encoded).
   - Computes `code_challenge = BASE64URL(SHA256(code_verifier))`.
   - Redirects to the authorization endpoint with `code_challenge` and `code_challenge_method=S256`.
4. After redirect back with the code, exchange the code by POSTing both `code` and `code_verifier`.
5. Verify Keycloak rejects the exchange if the wrong `code_verifier` is sent.

### Lab Exercises
- Try the authorization code flow WITHOUT PKCE on a public client — observe if Keycloak rejects it (if enforced).
- Enforce PKCE-only for all public clients in realm settings.
- Simulate the code interception attack and show why PKCE prevents it.

### Learning Outcomes
- Why SPAs and mobile apps must use PKCE instead of client secrets.
- The cryptographic proof mechanism behind PKCE.
- Why `plain` code challenge method is insecure and `S256` is required.

### Blog Post Angle
*"PKCE Explained: Why SPAs Can't Use Client Secrets and How Keycloak Enforces It"* — a clear explanation of the threat PKCE solves with real request examples.

---

## Lab 24 — Device Authorization Flow (OAuth 2.0 Device Flow)

### Goal
Implement the OAuth 2.0 Device Authorization Grant for devices with limited input capability (smart TVs, CLIs, IoT devices) that cannot easily handle browser redirects.

### Tools Required
- Keycloak Admin Console
- curl or Postman (to simulate the device)
- Any browser (for the authorization step — simulating a phone/PC)

### Key IAM Concepts
- **Device Authorization Grant (RFC 8628)**: An OAuth 2.0 flow where a device displays a code and URL, the user visits the URL on another device (phone/PC), authenticates, and the original device polls for the token.
- **Device Code**: A long-lived code issued to the device, used for polling.
- **User Code**: A short, human-readable code displayed to the user.
- **Polling**: The device repeatedly checks a token endpoint until the user completes authorization or the code expires.
- **Activation URL**: The URL the user must visit to authorize the device.

### Step-by-Step Outline
1. Create a client `smart-tv-app` with Device Authorization Grant enabled.
2. Simulate the device requesting a code:
   ```bash
   curl -X POST http://localhost:8080/realms/lab-realm/protocol/openid-connect/auth/device \
     -d "client_id=smart-tv-app"
   ```
3. Observe the response: `device_code`, `user_code`, `verification_uri`, `expires_in`, `interval`.
4. Display the `user_code` and `verification_uri` (as a TV would).
5. In a browser, visit `verification_uri` and enter the `user_code`. Authenticate as `testuser`.
6. The device polls the token endpoint every `interval` seconds:
   ```bash
   curl -X POST http://localhost:8080/realms/lab-realm/protocol/openid-connect/token \
     -d "grant_type=urn:ietf:params:oauth:grant-type:device_code&device_code=<code>&client_id=smart-tv-app"
   ```
7. Once the user approves, the poll returns the access token.

### Lab Exercises
- Test the `authorization_pending` and `slow_down` polling errors.
- Simulate user denial and observe the `access_denied` error.
- Let the device code expire (wait past `expires_in`) and verify `expired_token` error.
- Add a QR code to the simulated TV UI for the `verification_uri`.

### Learning Outcomes
- How Device Flow decouples authorization UX from constrained devices.
- The security model: the device code is never shown to the user; the user code is human-typeable.
- Why polling interval matters for server load management.

### Blog Post Angle
*"OAuth 2.0 Device Flow: How Smart TVs and CLI Tools Authenticate Users"* — explain why standard redirect flows break for headless devices and how Device Flow solves it elegantly.

---

## Lab 25 — User Self-Service (Registration, Profile & Password Reset)

### Goal
Configure and test Keycloak's end-user self-service capabilities: self-registration, profile editing, password reset via email, and the Account Management Console.

### Tools Required
- Mailhog (from Lab 17) for email verification and password reset
- Browser

### Key IAM Concepts
- **Self-Registration**: Allowing users to create their own accounts without admin intervention.
- **Email Verification**: Requiring users to confirm their email address before the account is activated.
- **Forgot Password Flow**: A secure flow where a reset link is sent to the registered email.
- **Account Management Console**: Keycloak's built-in user-facing portal for profile management, MFA enrollment, and session management.
- **Required Actions**: Tasks a user must complete (update password, verify email, configure OTP) before accessing resources.

### Step-by-Step Outline
1. Enable self-registration in Realm Settings → Login.
2. Enable "Verify Email" as a default required action.
3. A new user registers → receives verification email in Mailhog → clicks link.
4. Enable "Forgot Password" in Login settings.
5. Trigger a password reset from the login page → receive reset email in Mailhog → complete reset.
6. Explore the Account Console:
   ```
   http://localhost:8080/realms/lab-realm/account
   ```
7. Configure which profile fields are editable in the Account Console.
8. In Realm Settings → User Profile, add a custom `phone_number` field with a regex validation.

### Lab Exercises
- Require email verification before self-registered accounts can log in.
- Add a CAPTCHA to the registration form (recaptcha integration).
- Create a custom registration form with additional required fields using Keycloak's User Profile feature.
- Test the password reset link expiry (default 5 minutes).

### Learning Outcomes
- How self-service reduces helpdesk load while maintaining security controls.
- Why email-based password reset is only as secure as email account security.
- How Keycloak's User Profile (declarative schema) replaces custom registration forms.

### Blog Post Angle
*"User Self-Service with Keycloak: Registration, Password Reset, and the Account Console"* — ideal for product teams evaluating authentication UX.

---

## Lab 26 — Admin REST API Automation

### Goal
Automate Keycloak administration tasks using the Admin REST API — creating realms, users, clients, and roles programmatically. Build a declarative realm provisioning script.

### Tools Required
- Python (`requests` library) or Node.js (`axios`)
- Postman (for API exploration)

### Key IAM Concepts
- **Infrastructure as Code (IaC) for IAM**: Treating identity configuration as versionable, repeatable code.
- **Admin API Client**: A confidential client with admin permissions used to call the REST API.
- **Realm Export/Import**: Keycloak's mechanism for backing up and restoring realm configuration.
- **Idempotent Provisioning**: Writing scripts that can be run multiple times without side effects.

### Step-by-Step Outline
1. Authenticate the admin CLI client to get an admin access token:
   ```python
   token = requests.post(
     "http://localhost:8080/realms/master/protocol/openid-connect/token",
     data={"grant_type": "password", "client_id": "admin-cli",
           "username": "admin", "password": "admin"}
   ).json()["access_token"]
   ```
2. Write a script to:
   - Create a new realm: `auto-realm`.
   - Create roles: `admin`, `viewer`.
   - Create users with attributes and role assignments.
   - Create a confidential client with redirect URIs.
3. Export the realm to JSON via API.
4. Delete the realm and re-import from the JSON file.
5. Add a health check: verify all expected resources exist after provisioning.

### Lab Exercises
- Build a CLI tool (`python provision.py --realm myapp --users users.csv`) that reads from a CSV and provisions users.
- Implement idempotency: if a user already exists, update rather than fail.
- Use Keycloak's Terraform provider (free, open source) to provision realm configuration.
- Write a deprovisioning script that safely removes a realm and all its clients.

### Learning Outcomes
- How Keycloak's API-first design enables GitOps-style IAM management.
- Why manual Keycloak configuration is a security risk in production (no audit trail, no repeatability).
- The role of realm export/import in disaster recovery.

### Blog Post Angle
*"Automating Keycloak with the Admin REST API: GitOps for Identity Management"* — a DevOps-focused post on treating IAM config as code.

---

## Lab 27 — Event Listeners & Audit Logging

### Goal
Configure Keycloak's event system to capture and forward authentication and admin events to an external system for security monitoring and audit purposes.

### Tools Required
- [Seq](https://datalust.co/seq) (free for local use) or Elasticsearch + Kibana (free tier) — or simply log to a file
- A simple HTTP listener (Python Flask app) to receive webhook-style events
- Keycloak Admin Console

### Key IAM Concepts
- **Authentication Events**: Login success/failure, logout, token issuance, code-to-token exchange.
- **Admin Events**: Changes to realm configuration, user creation/deletion, role assignment.
- **Event Listener SPI**: Keycloak's extension point for custom event handling (email, webhook, SIEM).
- **Security Information and Event Management (SIEM)**: Centralized collection and analysis of security events for threat detection.
- **Audit Trail**: An immutable record of who did what and when — a compliance requirement.

### Step-by-Step Outline
1. In Realm Settings → Events → Config:
   - Enable login events to save.
   - Enable admin events with representation (stores before/after state).
   - Set event expiry to 7 days.
2. Trigger several events: login, failed login, logout, create user.
3. View events in Admin Console → Events.
4. Write a Python Flask listener:
   ```python
   @app.route('/keycloak-events', methods=['POST'])
   def receive_event():
       event = request.json
       print(f"Event: {event['type']} | User: {event.get('userId')}")
       return '', 200
   ```
5. Install the [Keycloak Event Publisher](https://github.com/SnuK87/keycloak-kafka) or a custom HTTP event listener SPI to forward events to the Flask app.
6. Trigger login events and verify they arrive at the Flask listener.

### Lab Exercises
- Alert on 3+ failed login events from the same IP within 5 minutes.
- Ship events to Elasticsearch and build a Kibana dashboard showing login geography and failure rates.
- Audit every role assignment change and export to CSV for compliance.
- Implement GDPR-compliant event anonymization (hash user IDs in exported events).

### Learning Outcomes
- How Keycloak's event model supports compliance requirements (SOC2, ISO 27001, GDPR).
- The difference between authentication events (user-facing) and admin events (system-facing).
- Why event logs need to be shipped out of Keycloak — in-database event storage is not suitable for production.

### Blog Post Angle
*"Building a Security Audit Trail with Keycloak Events"* — a compliance and security operations post showing how to route Keycloak events to a SIEM.

---

## Lab 28 — Session Management & Concurrent Session Limiting

### Goal
Configure session policies to control session lifetime, idle timeouts, and concurrent session limits — and understand the security implications of each configuration.

### Tools Required
- Multiple browsers or browser profiles (to simulate concurrent sessions)
- Keycloak Admin Console

### Key IAM Concepts
- **SSO Session Lifetime**: The maximum duration of a Keycloak SSO session before forced re-authentication.
- **SSO Session Idle**: How long a session can be idle before expiring.
- **Offline Session**: A long-lived session backed by an offline refresh token — used for mobile apps that need to work offline.
- **Concurrent Session Limit**: Restricting how many simultaneous sessions a user can have (prevent account sharing).
- **Session Fixation**: An attack where an attacker pre-sets a session ID — Keycloak mitigates this by regenerating session IDs after authentication.

### Step-by-Step Outline
1. Configure session timeouts in Realm Settings → Sessions:
   - SSO Session Idle: 5 minutes.
   - SSO Session Max: 30 minutes.
   - Access Token Lifespan: 2 minutes.
2. Log in and wait 5+ minutes without activity — verify automatic logout.
3. View and manage all active sessions in Admin Console → Sessions.
4. Terminate a specific session from Admin Console → observe logout on the client.
5. **Concurrent Session Limiting:**
   - Install Keycloak's [Limiting Session Authenticator](https://github.com/wadahiro/keycloak-limiting-session) or write a custom policy.
   - Configure max sessions per user = 2.
   - Open 3 browser profiles and log in — verify the oldest session is terminated.

### Lab Exercises
- Test the "Remember Me" option and observe extended session lifetime.
- Implement "Log me out of all sessions" from the Account Console.
- Simulate session hijacking (copy a session cookie to a new browser) and configure IP-pinned session validation.
- Configure per-client session idle timeouts (different from realm defaults).

### Learning Outcomes
- How session lifetime configuration balances security (short sessions) and UX (long sessions).
- Why concurrent session limiting is a compliance control for shared-account prevention.
- The difference between a Keycloak SSO session and an application-level session.

### Blog Post Angle
*"Keycloak Session Security: Timeouts, Concurrent Limits, and Forced Logout"* — a security operations guide to session hardening.

---

## Lab 29 — Securing a React SPA with Keycloak.js

### Goal
Integrate Keycloak authentication into a React single-page application using the official `keycloak-js` adapter. Implement login, logout, token refresh, and route protection.

### Tools Required
- Node.js + npm
- `create-react-app` or Vite
- `keycloak-js` npm package
- Visual Studio Code (optional)

### Key IAM Concepts
- **SPA Authentication**: Browser-based apps cannot store secrets — must use public clients with PKCE.
- **keycloak-js Adapter**: Keycloak's official JavaScript library handling the OIDC flow, token storage, and silent refresh.
- **Silent Refresh**: Using a hidden iframe to refresh tokens without interrupting the user experience.
- **Protected Route**: A React component that redirects unauthenticated users to login.
- **Token Storage**: Where to store tokens in a browser (memory vs. localStorage vs. cookie) and the tradeoffs.

### Step-by-Step Outline
1. Bootstrap a React app: `npm create vite@latest my-spa -- --template react`.
2. Install: `npm install keycloak-js`.
3. Initialize Keycloak:
   ```javascript
   const keycloak = new Keycloak({
     url: 'http://localhost:8080',
     realm: 'lab-realm',
     clientId: 'my-web-app',
   });
   await keycloak.init({ onLoad: 'check-sso', pkceMethod: 'S256' });
   ```
4. Build a `ProtectedRoute` component that redirects to Keycloak if not authenticated.
5. Display user name and token claims from `keycloak.tokenParsed`.
6. Implement logout button: `keycloak.logout()`.
7. Set up silent token refresh:
   ```javascript
   setInterval(() => {
     keycloak.updateToken(60).catch(() => keycloak.login());
   }, 30000);
   ```
8. Call a backend API from Lab 7 using the access token.

### Lab Exercises
- Implement role-based UI rendering: show admin menu only if `realm_access.roles` includes `admin`.
- Store nothing in localStorage — use memory-only token storage.
- Implement the silent check-sso iframe and observe it in DevTools.
- Add a token expiry countdown to the UI.

### Learning Outcomes
- How browser-based apps implement OAuth without exposing secrets.
- The XSS/CSRF tradeoffs of different token storage strategies.
- How silent refresh enables seamless UX with short-lived access tokens.

### Blog Post Angle
*"Securing a React App with Keycloak: PKCE, Silent Refresh, and Route Protection"* — a full-stack tutorial connecting a React SPA to a Keycloak-protected Express API.

---

## Lab 30 — Securing a Node.js / Express API

### Goal
Build a production-grade JWT middleware for an Express API that validates Keycloak-issued tokens, enforces roles, handles token expiry, and returns standardized error responses.

### Tools Required
- Node.js (`express`, `jwks-rsa`, `express-jwt` or `jsonwebtoken`)
- Postman
- Keycloak (from any previous lab)

### Key IAM Concepts
- **JWT Validation**: Verifying token signature, expiry, issuer, and audience — stateless, no IdP call required.
- **JWKS Caching**: Fetching and caching the IdP's public keys for efficient token validation.
- **Bearer Token**: The HTTP Authorization header pattern: `Authorization: Bearer <token>`.
- **WWW-Authenticate Header**: The standard HTTP response header for 401 errors indicating the required authentication scheme.
- **Token Claims Extraction**: Reading `realm_access.roles`, `sub`, `preferred_username`, and custom claims from the validated payload.

### Step-by-Step Outline
1. Install dependencies:
   ```bash
   npm install express jsonwebtoken jwks-rsa express-jwt
   ```
2. Build JWT middleware using `jwks-rsa` to auto-fetch and cache Keycloak's public keys:
   ```javascript
   const { expressjwt: jwt } = require('express-jwt');
   const jwksRsa = require('jwks-rsa');
   
   const verifyJwt = jwt({
     secret: jwksRsa.expressJwtSecret({
       cache: true,
       rateLimit: true,
       jwksUri: 'http://localhost:8080/realms/lab-realm/protocol/openid-connect/certs',
     }),
     algorithms: ['RS256'],
     issuer: 'http://localhost:8080/realms/lab-realm',
     audience: 'account',
   });
   ```
3. Build role-checking middleware:
   ```javascript
   const requireRole = (role) => (req, res, next) => {
     const roles = req.auth?.realm_access?.roles || [];
     if (!roles.includes(role)) return res.status(403).json({ error: 'Forbidden' });
     next();
   };
   ```
4. Apply middleware to routes:
   ```javascript
   app.get('/api/data', verifyJwt, requireRole('viewer'), handler);
   app.delete('/api/data/:id', verifyJwt, requireRole('admin'), handler);
   ```
5. Return proper `WWW-Authenticate: Bearer` header on 401.
6. Test with valid, expired, wrong-audience, and tampered tokens.

### Lab Exercises
- Handle clock skew (add `clockTolerance: 5` seconds to handle slight time differences).
- Build middleware for client role validation (`resource_access.<clientId>.roles`).
- Add request logging that extracts `sub` and `preferred_username` for audit.
- Validate the `azp` (authorized party) claim to ensure the token was issued for the expected client.
- Write unit tests using mock JWTs signed with a test key pair.

### Learning Outcomes
- How stateless JWT validation works without calling the IdP per request.
- Why JWKS caching is critical for API performance.
- The complete set of claims that should be validated on every token.
- Common JWT security pitfalls: algorithm confusion (`alg: none`), missing audience check, no expiry check.

### Blog Post Angle
*"JWT Validation in Node.js: Building Production-Grade Keycloak Middleware"* — a security-focused tutorial with a hardened middleware implementation and common pitfall walkthrough.

---

## Summary: IAM Concepts Coverage Map

| IAM Domain | Labs |
|---|---|
| Authentication Basics | 1, 2 |
| Multi-Factor Authentication | 3 |
| OAuth 2.0 | 4, 5, 23, 24 |
| OpenID Connect (OIDC) | 6 |
| Token Engineering | 21, 22 |
| RBAC | 7 |
| ABAC | 8 |
| Fine-Grained Authorization / PBAC | 19 |
| Token Exchange & Delegation | 20 |
| Single Sign-On (SSO) | 9 |
| Single Logout (SLO) | 10 |
| Social Login / Identity Brokering | 11, 12, 14 |
| SAML 2.0 | 13 |
| Directory Services / LDAP | 15 |
| SCIM 2.0 / Provisioning | 16 |
| Custom Auth Flows | 17 |
| Passwordless | 18 |
| Session Management | 28 |
| Self-Service UX | 25 |
| Admin API / Automation | 26 |
| Audit & Observability | 27 |
| App Integration (SPA) | 29 |
| App Integration (API) | 30 |

---

## Recommended Learning Path

**Week 1 — Foundations:** Labs 1 → 2 → 4 → 5 → 6

**Week 2 — Authorization:** Labs 7 → 8 → 21 → 22 → 23

**Week 3 — SSO & Federation:** Labs 9 → 10 → 11 → 12 → 13

**Week 4 — Enterprise IAM:** Labs 14 → 15 → 16 → 27

**Week 5 — Advanced Flows:** Labs 3 → 17 → 18 → 19 → 20

**Week 6 — Integration & Production:** Labs 24 → 25 → 26 → 28 → 29 → 30

---

*All labs are designed for Keycloak 25+ running in development mode. For production deployments, additional hardening (TLS, database backend, clustering) is required.*
