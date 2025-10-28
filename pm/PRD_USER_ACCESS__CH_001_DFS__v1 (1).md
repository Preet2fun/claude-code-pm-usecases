---
dfs_id: CH_001_PASSWORDLESS_AUTH_DFS_v1
title: Passwordless Authentication
parent_prd: PRD_USER_ACCESS
parent_chapter: CH_001
version: 1
status: draft
generated_by: claude-chapter-create
generated_at: 2025-10-03T00:00:00Z
platform_context: loaded
---

# Detailed Feature Specification (DFS)

## Overview

```
Parent PRD ID: PRD_USER_ACCESS
Parent Chapter ID: CH_001
DFS ID: CH_001_PASSWORDLESS_AUTH_DFS_v1
Title: Passwordless Authentication
Summary: Secure, frictionless authentication using email-based OTP/magic links with optional TOTP MFA for privileged users
```

---

## 1. Identification

- **DFS ID**: CH_001_PASSWORDLESS_AUTH_DFS_v1
- **Title**: Passwordless Authentication
- **Parent PRD**: PRD_USER_ACCESS
- **Parent Chapter**: CH_001
- **Version**: 1
- **Status**: draft
- **Related DFS**:
  - CH_002_DFS (User Lifecycle Management - enabled by this feature)
  - CH_003_DFS (Session Management - enabled by this feature)
- **Authors & Reviewers**: Product Team

---

## 2. User Story

```
As an end user, I want to authenticate without passwords using email-based verification and optional authenticator apps, so that I experience frictionless, secure access while the platform enforces security policies appropriate to my role.
```

---

## 3. Acceptance Criteria — Functional Behavior

### 3.1 UI Behavior

**Login Screen (Public - Unauthenticated Users)**

**Location:** `/login` (public route)

**Navigation:** Direct URL access, redirected from protected routes, or via email invitation links

**UI Elements:**
- **Adaptive Canvas:** Full-page login form (no context to preserve - first entry point)
- **Primary Input:** Email field (autofocus)
- **Primary CTA:** "Continue" button
- **Secondary Options:**
  - "Send magic link instead" (text link, toggles delivery method)
  - "Trouble signing in?" (link to help documentation)
- **Branding Area:** Tenant logo + customized welcome message (tenant-specific)

**Interaction Flow:**
1. User enters email address
2. Platform validates email format in real-time (inline error for invalid format)
3. User clicks "Continue"
4. System displays "Check your email" success screen with:
   - Email address confirmation (masked: j***@example.com)
   - Instruction: "Enter the 6-digit code sent to your email"
   - OTP input field (6 digits, auto-focus, numeric keypad on mobile)
   - Countdown timer: "Code expires in 5:00"
   - Resend link: "Didn't receive code? Resend" (disabled for 30s)
   - Alternative method link: "Send magic link instead"

**Chat-First Alternative:**
- **Not applicable** for login (security - must verify identity before chat access)

**AI Augmentation:**
- Smart email domain detection → If corporate domain detected, display banner: "Your organization uses SSO. Sign in with [SSO Provider]" (future enhancement placeholder)

**Dynamic Navigation:**
- Post-authentication redirect to:
  - Original requested URL (if redirected from protected route)
  - Default dashboard for user role
  - TOTP enrollment screen (if privileged user on first login)

**Context Preservation:**
- Pre-login state preserved via `redirect_url` query parameter
- TOTP enrollment required before accessing any other interface (blocking modal for privileged users)

---

**TOTP Enrollment Screen (Authenticated Privileged Users - First Login)**

**Location:** `/auth/setup-mfa` (protected route, post-authentication checkpoint)

**Trigger:** Automatic redirect for privileged users (System Admin, Security Officer) on first successful login

**UI Pattern:** Blocking modal (full-screen overlay, cannot dismiss - security enforcement)

**UI Elements:**
- **Header:** "Secure Your Account - Required"
- **Instruction:** "As a privileged user, you must enable two-factor authentication"
- **QR Code Display:** Dynamic QR code for authenticator app enrollment
- **Manual Entry Option:** Collapsible section with TOTP secret key (for manual entry)
- **Verification Input:** 6-digit TOTP code field
- **Supported Apps List:** "Google Authenticator, Microsoft Authenticator, Authy, 1Password"
- **Primary CTA:** "Verify and Continue"
- **No Skip Option:** Enrollment is mandatory (no "Skip" or "Remind me later")

**Interaction Flow:**
1. User scans QR code with authenticator app
2. User enters 6-digit TOTP code from app
3. Platform validates code
4. Success → Redirect to originally requested destination
5. Failure → Inline error "Invalid code. Please try again" (preserves QR code, allows re-entry)

**Self-Service TOTP Management (All Authenticated Users - Profile Settings)**

**Location:** `/profile/security` (adaptive canvas - right panel preserves current workspace)

**Access:** Any authenticated user (self-service)

**UI Pattern:** Adaptive canvas (right panel slide-in)

**UI Elements:**
- **Section Header:** "Two-Factor Authentication (Recommended)"
- **Status Indicator:**
  - Enabled: Green badge "Active" + enrolled device name + "Enrolled on [date]"
  - Disabled: Yellow badge "Not Configured" + "Add extra security to your account"
- **Actions:**
  - If not enrolled: "Enable Two-Factor Authentication" button
  - If enrolled: "Remove MFA" button (requires current TOTP verification) + "View Backup Codes"
- **Enrollment Flow:** Same as first-login TOTP enrollment (non-blocking, user-initiated)

---

**Progressive Lockout Warning States**

**Trigger:** Failed authentication attempts

**UI Behavior by Failure Count:**

**1-3 Failures:**
- Instant retry allowed
- Inline error: "Invalid code. Please check your email and try again."
- No additional friction

**4-5 Failures:**
- 30-second delay imposed
- Warning banner (yellow): "Too many failed attempts. You can try again in [countdown timer]"
- "Need help?" link to support documentation
- Submit button disabled during countdown

**5+ Failures:**
- 15-minute lockout
- Blocking error screen (replaces login form):
  - Icon: Lock symbol
  - Heading: "Account Temporarily Locked"
  - Message: "For your security, this account is locked due to multiple failed login attempts."
  - Countdown: "You can try again in [14:32]"
  - Alternative action: "Contact your administrator for immediate assistance"
  - Email notification sent to user: "Unusual login activity detected"

---

### 3.2 Field Definitions

**Authentication Request (User Input)**

```
email — string, required, max 255 chars, case-insensitive, regex: ^[^@]+@[^@]+\.[^@]+$, stored as lowercase, trimmed whitespace
authentication_method — enum, optional, values: ["otp_code", "magic_link"], default: "otp_code"
device_fingerprint — string, optional, max 512 chars, browser/device hash for anomaly detection
```

**OTP Generation (System-Generated)**

```
otp_code — string, system-generated, 6 numeric digits, cryptographically random
otp_token_id — UUID, unique identifier for OTP record
otp_expires_at — ISO 8601 UTC timestamp, 5 minutes from generation
otp_created_at — ISO 8601 UTC timestamp, generation time
otp_attempts_remaining — integer, default: 3, decrements on validation failure
```

**Magic Link Generation (System-Generated)**

```
magic_link_token — string, system-generated, 256-bit cryptographically random hex string, URL-safe
magic_link_url — string, full URL: https://{tenant_domain}/auth/verify?token={magic_link_token}
magic_link_expires_at — ISO 8601 UTC timestamp, 15 minutes from generation
magic_link_single_use — boolean, true (token invalidated after first successful use)
```

**TOTP Configuration (User Enrollment)**

```
totp_secret — string, system-generated, base32-encoded 160-bit secret, stored encrypted
totp_qr_code_data — string, URI format: otpauth://totp/{tenant}:{email}?secret={totp_secret}&issuer={tenant_name}
totp_enrolled_at — ISO 8601 UTC timestamp, enrollment completion time
totp_device_name — string, optional, max 50 chars, user-provided label (e.g., "iPhone 13"), default: "Authenticator App"
totp_backup_codes — array[string], 10 single-use recovery codes, 8 alphanumeric chars each, stored hashed
```

**Authentication Session (System State)**

```
session_id — UUID, unique session identifier
user_id — UUID, authenticated user identifier
tenant_id — UUID, user's tenant identifier (implicit scoping)
session_created_at — ISO 8601 UTC timestamp
session_expires_at — ISO 8601 UTC timestamp, configurable (default: 24 hours from last activity)
session_ip_address — string, IPv4/IPv6 address
session_user_agent — string, max 512 chars, browser/device info
mfa_verified — boolean, true if TOTP verified (for privileged users)
```

**Failed Authentication Tracking (System State - Rate Limiting)**

```
failed_attempts_count — integer, increments per failed validation, resets on success
failed_attempts_window_start — ISO 8601 UTC timestamp, 15-minute rolling window start
lockout_expires_at — ISO 8601 UTC timestamp, null if not locked, 15 minutes from 5th failure
```

---

### 3.3 Business Logic / Flows

**Flow 1: Email-Based OTP Authentication (Primary Path)**

**Trigger:** User submits email on login screen with authentication_method = "otp_code" (default)

**Steps:**

1. **Email Validation**
   - Validate email format (RFC 5322 regex)
   - Normalize email (lowercase, trim whitespace)
   - Check if email exists in tenant (if not, treat as invalid - don't reveal user existence for security)

2. **Rate Limiting Check**
   - Query failed_attempts for email in last 15 minutes
   - If count ≥ 5 → return error "Account temporarily locked" (HTTP 429)
   - If count = 4 → impose 30-second delay before processing
   - If count ≤ 3 → proceed immediately

3. **OTP Generation**
   - Generate 6-digit random numeric code (cryptographically secure)
   - Create OTP record: {otp_code, email, otp_expires_at: now + 5min, otp_attempts_remaining: 3}
   - Invalidate any existing unused OTP for same email (one active OTP per user)

4. **Email Delivery (Asynchronous with Auto-Retry)**
   - Queue email: "Your OneOps Login Code: {otp_code}" (plain text + HTML template)
   - **Retry Logic (per Discovery Q3 Decision):**
     - Attempt 1: Immediate send
     - If failure: Wait 5 seconds → Attempt 2
     - If failure: Wait 15 seconds → Attempt 3
     - If all attempts fail: Log error, return success to user (don't reveal delivery failure)
   - Email includes: Code, expiry time, "Didn't request this? Ignore this email"

5. **User Response**
   - Display "Check your email" screen (UI section 3.1)
   - User enters OTP code
   - Submit triggers Flow 2 (OTP Validation)

**Permissions:** Public (unauthenticated)

**Audit Event:** `auth.otp_generated {email, timestamp, ip_address, device_fingerprint}`

---

**Flow 2: OTP Validation & Session Creation**

**Trigger:** User submits OTP code from email

**Steps:**

1. **OTP Lookup**
   - Find OTP record by email (most recent, unused)
   - If not found → error "Invalid or expired code"
   - If otp_expires_at < now → error "Code expired. Request a new code"

2. **Attempt Limit Check**
   - If otp_attempts_remaining = 0 → error "Too many failed attempts. Request a new code"

3. **Code Validation**
   - Compare submitted code to otp_code (constant-time comparison)
   - If mismatch:
     - Decrement otp_attempts_remaining
     - Increment failed_attempts_count for email
     - Return error "Invalid code. X attempts remaining"
   - If match: Proceed to step 4

4. **Session Creation**
   - Create session record: {session_id, user_id, tenant_id, session_created_at, session_expires_at, session_ip_address, session_user_agent}
   - Mark OTP as used (otp_used_at = now)
   - Reset failed_attempts_count to 0

5. **MFA Enforcement (Privileged Users - per Discovery Q2 Decision)**
   - Check if user role = "System Admin" OR "Security Officer"
   - If yes AND totp_enrolled_at IS NULL → set mfa_verified = false
   - Redirect to `/auth/setup-mfa` (blocking TOTP enrollment)
   - If TOTP already enrolled → require TOTP verification (Flow 4)
   - If not privileged user → proceed to step 6

6. **Session Response**
   - Return session token (HTTP-only cookie or Bearer token)
   - Redirect to target URL or default dashboard

**Permissions:** Public (unauthenticated)

**Audit Event:** `auth.session_created {user_id, email, timestamp, ip_address, device_fingerprint, mfa_verified}`

---

**Flow 3: Magic Link Authentication (Alternative Path)**

**Trigger:** User selects "Send magic link instead" option

**Steps:**

1. **Email Validation** (same as Flow 1, step 1)

2. **Rate Limiting Check** (same as Flow 1, step 2)

3. **Magic Link Generation**
   - Generate 256-bit random token (URL-safe base64)
   - Create magic link record: {magic_link_token, email, magic_link_expires_at: now + 15min, magic_link_single_use: true}
   - Invalidate any existing unused magic links for same email

4. **Email Delivery (Asynchronous with Auto-Retry)**
   - Queue email: "Your OneOps Login Link" (HTML template with button)
   - Link: `https://{tenant_domain}/auth/verify?token={magic_link_token}`
   - **Same retry logic as Flow 1, step 4**

5. **User Clicks Link**
   - Browser opens `/auth/verify?token={magic_link_token}` (may be different device/browser)
   - Validate token (Flow 3a)

**Permissions:** Public (unauthenticated)

**Audit Event:** `auth.magic_link_generated {email, timestamp, ip_address, device_fingerprint}`

---

**Flow 3a: Magic Link Validation & Session Creation**

**Trigger:** User clicks magic link (GET request to /auth/verify)

**Steps:**

1. **Token Lookup**
   - Extract token from query parameter
   - Find magic link record by token
   - If not found → error "Invalid or expired link"
   - If magic_link_expires_at < now → error "Link expired. Request a new one"
   - If already used (magic_link_used_at IS NOT NULL) → error "Link already used. Request a new one"

2. **Session Creation**
   - Same as Flow 2, steps 4-6
   - Mark magic link as used (magic_link_used_at = now)

**Permissions:** Public (unauthenticated)

**Audit Event:** `auth.magic_link_used {user_id, email, timestamp, ip_address, device_fingerprint}`

---

**Flow 4: TOTP Enrollment (Privileged Users - First Login)**

**Trigger:** Privileged user completes OTP/magic link authentication, totp_enrolled_at IS NULL

**Steps:**

1. **TOTP Secret Generation**
   - Generate 160-bit random secret (base32-encoded)
   - Create TOTP URI: `otpauth://totp/{tenant_name}:{email}?secret={totp_secret}&issuer={tenant_name}`
   - Generate QR code image from URI

2. **Display Enrollment Screen**
   - Show QR code (UI section 3.1)
   - Display manual entry secret (for users without camera)
   - Wait for user to submit TOTP code

3. **TOTP Verification**
   - User submits 6-digit code from authenticator app
   - Validate code against totp_secret using TOTP algorithm (RFC 6238, 30s time step, ±1 time window tolerance)
   - If invalid → error "Invalid code. Please try again" (allow unlimited retries)
   - If valid: Proceed to step 4

4. **Enrollment Completion**
   - Store totp_secret (encrypted at rest)
   - Set totp_enrolled_at = now
   - Generate 10 backup codes (8-char alphanumeric, store hashed)
   - Display backup codes to user (one-time view, must save)
   - Update session: mfa_verified = true

5. **Redirect**
   - Redirect to originally requested URL or default dashboard

**Permissions:** Authenticated session required (post-OTP/magic link validation)

**Audit Event:** `auth.totp_enrolled {user_id, email, timestamp, device_name}`

---

**Flow 5: TOTP Verification (Privileged Users - Subsequent Logins)**

**Trigger:** Privileged user completes OTP/magic link authentication, totp_enrolled_at IS NOT NULL

**Steps:**

1. **Display TOTP Prompt**
   - After OTP/magic link validation, show TOTP verification screen (adaptive canvas - inline, no full redirect)
   - Input: 6-digit code
   - Option: "Use backup code instead" (reveals different input field)

2. **TOTP Validation**
   - Validate code against stored totp_secret (RFC 6238, ±1 time window)
   - If invalid → error "Invalid code. Please try again"
   - If valid: Proceed to step 3

3. **Session Update**
   - Set mfa_verified = true
   - Allow access to platform

**Permissions:** Authenticated session required

**Audit Event:** `auth.totp_verified {user_id, email, timestamp, ip_address}`

---

**Flow 6: Self-Service TOTP Management (All Users - Optional)**

**Trigger:** Any authenticated user navigates to `/profile/security` and clicks "Enable Two-Factor Authentication"

**Steps:**

1. **Enrollment Flow** (same as Flow 4, steps 1-4)
   - Non-blocking (user-initiated, not mandatory for non-privileged users)
   - Can cancel/exit at any time

2. **Removal Flow**
   - User clicks "Remove MFA"
   - Require current TOTP verification (prevents unauthorized removal)
   - Delete totp_secret, invalidate backup codes
   - Set totp_enrolled_at = NULL

**Permissions:** Authenticated session required

**Audit Event:**
- `auth.totp_self_enrolled {user_id, timestamp}`
- `auth.totp_removed {user_id, timestamp}`

---

**Cross-Cutting Functional Constraints:**

**Multi-Tenancy:**
- All authentication data (OTP, magic links, sessions) scoped to user's tenant
- Email uniqueness enforced per tenant (not globally)
- Tenant-specific branding applied to login screen and emails

**Permissions:**
- Public routes: `/login`, `/auth/verify` (no authentication required)
- Protected routes: `/auth/setup-mfa`, `/profile/security` (authenticated session required)
- TOTP enrollment mandatory for privileged users, optional for others

**Audit Logging (What to Log):**
- Event: `auth.otp_generated` → Fields: {email, timestamp, ip_address, device_fingerprint}
- Event: `auth.session_created` → Fields: {user_id, email, timestamp, ip_address, device_fingerprint, mfa_verified}
- Event: `auth.magic_link_generated` → Fields: {email, timestamp, ip_address}
- Event: `auth.magic_link_used` → Fields: {user_id, email, timestamp, ip_address}
- Event: `auth.totp_enrolled` → Fields: {user_id, email, timestamp, device_name}
- Event: `auth.totp_verified` → Fields: {user_id, email, timestamp, ip_address}
- Event: `auth.authentication_failed` → Fields: {email, reason, timestamp, ip_address, attempts_remaining}
- Event: `auth.account_locked` → Fields: {email, timestamp, lockout_duration}

**AI Touchpoints:**

- **Predictive:** Email domain detection for SSO suggestion (future enhancement)
- **Anomaly Detection:** Unusual login patterns (new device, new location) trigger additional verification or admin notification
- **Risk Scoring:** Device fingerprint analysis to detect suspicious authentication attempts

---

### 3.4 Error & Validation Rules

**Email Validation Errors**

```
Email field empty → 400 BAD_REQUEST "FIELD_REQUIRED" "Email address is required"
Email format invalid (fails regex) → 400 BAD_REQUEST "INVALID_EMAIL_FORMAT" "Please enter a valid email address"
Email does not exist in tenant → 401 UNAUTHORIZED "INVALID_CREDENTIALS" "Invalid email or authentication failed" (generic message - don't reveal user existence)
```

**OTP Validation Errors**

```
OTP code field empty → 400 BAD_REQUEST "FIELD_REQUIRED" "Verification code is required"
OTP code format invalid (not 6 digits) → 400 BAD_REQUEST "INVALID_CODE_FORMAT" "Code must be 6 digits"
OTP code not found or expired → 401 UNAUTHORIZED "CODE_EXPIRED" "Code expired. Click 'Resend' to get a new code"
OTP code mismatch (incorrect code) → 401 UNAUTHORIZED "INVALID_CODE" "Invalid code. {X} attempts remaining"
OTP attempts exhausted → 401 UNAUTHORIZED "TOO_MANY_ATTEMPTS" "Too many failed attempts. Request a new code"
```

**Magic Link Validation Errors**

```
Token parameter missing → 400 BAD_REQUEST "INVALID_REQUEST" "Missing authentication token"
Token not found or expired → 401 UNAUTHORIZED "LINK_EXPIRED" "This link has expired. Request a new login link"
Token already used → 401 UNAUTHORIZED "LINK_ALREADY_USED" "This link has already been used. Request a new login link"
```

**TOTP Validation Errors**

```
TOTP code field empty → 400 BAD_REQUEST "FIELD_REQUIRED" "Verification code is required"
TOTP code format invalid (not 6 digits) → 400 BAD_REQUEST "INVALID_CODE_FORMAT" "Code must be 6 digits"
TOTP code mismatch → 401 UNAUTHORIZED "INVALID_TOTP_CODE" "Invalid code. Please check your authenticator app and try again"
```

**Rate Limiting Errors**

```
OTP resend requested <30 seconds → 429 TOO_MANY_REQUESTS "RATE_LIMIT_RESEND" "Please wait {X} seconds before requesting a new code"
4th failed attempt (30s delay imposed) → 429 TOO_MANY_REQUESTS "RATE_LIMIT_DELAY" "Too many failed attempts. Please wait 30 seconds"
5+ failed attempts (15min lockout) → 429 TOO_MANY_REQUESTS "ACCOUNT_LOCKED" "Account temporarily locked due to multiple failed attempts. Try again in {X} minutes"
```

**Session Errors**

```
Session token missing → 401 UNAUTHORIZED "AUTHENTICATION_REQUIRED" "Please sign in to continue"
Session token invalid or expired → 401 UNAUTHORIZED "SESSION_EXPIRED" "Your session has expired. Please sign in again"
TOTP required but not verified → 403 FORBIDDEN "MFA_REQUIRED" "Two-factor authentication required. Please complete MFA setup"
```

**Email Delivery Errors (Internal - Not Surfaced to User per Discovery Q3)**

```
Email service unavailable after retries → Log error with ID, return success to user (silent failure)
```

---

### 3.5 Cross-Entity Interactions

**Email/Notification Service**

- **Interaction Type:** Asynchronous (event-driven)
- **Data Exchanged:**
  - OTP email: {recipient_email, otp_code, tenant_branding, expires_at}
  - Magic link email: {recipient_email, magic_link_url, tenant_branding, expires_at}
  - Account lockout notification: {recipient_email, lockout_expires_at, support_contact}
- **Timing:** Triggered immediately after OTP/magic link generation
- **Failure Handling:**
  - Automatic retry: 3 attempts with exponential backoff (5s, 15s)
  - Final failure: Log error with correlation ID, do not inform user (per Discovery Q3 Decision)
- **SLA:** >99.5% delivery success within 2 minutes (including retries)

**User Profile Service**

- **Interaction Type:** Synchronous (API call)
- **Data Exchanged:**
  - Lookup: {email, tenant_id} → {user_id, role, totp_enrolled_at}
  - TOTP update: {user_id, totp_secret, totp_enrolled_at, backup_codes}
- **Timing:** During authentication validation and TOTP enrollment
- **Failure Handling:**
  - Timeout: 5 seconds → Return 503 "Service temporarily unavailable"
  - Retry: 1 immediate retry, then fail
- **SLA:** <100ms response time (95th percentile)

**Session Management Service**

- **Interaction Type:** Synchronous (API call)
- **Data Exchanged:**
  - Create: {user_id, tenant_id, session_expires_at, ip_address, user_agent, mfa_verified}
  - Validate: {session_token} → {user_id, tenant_id, mfa_verified}
- **Timing:** After successful authentication
- **Failure Handling:**
  - Creation failure → Return 500 "Unable to create session. Please try again"
  - Circuit breaker: After 5 consecutive failures, return 503 for 30 seconds
- **SLA:** <50ms response time

**Audit Logging Service**

- **Interaction Type:** Asynchronous (fire-and-forget)
- **Data Exchanged:** Audit events (see section 3.3 for event schemas)
- **Timing:** Immediately after each authentication event
- **Failure Handling:**
  - Queue events locally if service unavailable
  - Retry with exponential backoff (max 3 retries)
  - Critical events (account_locked) escalate to admin notification if logging fails
- **SLA:** Best-effort (authentication flow not blocked by logging failures)

**AI Anomaly Detection Service (Optional - Future Enhancement)**

- **Interaction Type:** Asynchronous (non-blocking)
- **Data Exchanged:** {user_id, ip_address, device_fingerprint, location, timestamp} → {risk_score, anomaly_flags}
- **Timing:** During session creation (background analysis)
- **Failure Handling:** Graceful degradation (proceed with authentication if AI service unavailable)
- **SLA:** <500ms for risk score response (non-blocking)

---

### 3.6 Edge Cases & Exceptions

**Timing Issues**

1. **OTP Expires Mid-Entry**
   - User is typing code when 5-minute expiry passes
   - Behavior: Allow submission, validation fails with "Code expired" error
   - UI: Display "Request new code" button
   - Audit: Log expiry reason

2. **Session Expires During TOTP Enrollment**
   - User enrolling TOTP but session times out before completion
   - Behavior: Redirect to login, preserve partial enrollment state for 15 minutes
   - On re-authentication: Resume enrollment flow (show same QR code)
   - Audit: Log interrupted enrollment

3. **Concurrent OTP Requests**
   - User clicks "Resend" multiple times rapidly (race condition)
   - Behavior: Invalidate all previous OTPs, only latest is valid
   - Rate limit: Max 1 request per 30 seconds (enforced server-side)
   - UI: Disable "Resend" button for 30 seconds after click

4. **Clock Skew for TOTP**
   - User's device clock or server clock is inaccurate
   - Behavior: Accept codes from ±1 time window (90 seconds total: -30s to +60s)
   - If persistent failures: Display help message "Check your device time settings"

**Data Conflicts**

1. **Duplicate Session Creation**
   - User completes authentication in two browser tabs simultaneously
   - Behavior: Create separate sessions (both valid), track per session_id
   - Session limit: Max 5 concurrent sessions per user (configurable)
   - Oldest session invalidated when limit exceeded

2. **Email Address Changes Mid-Authentication**
   - Admin updates user email while OTP is pending
   - Behavior: OTP remains valid for old email (issued to that address)
   - New email requires new authentication flow
   - Audit: Log email change event

3. **TOTP Secret Regeneration**
   - User attempts to enroll TOTP twice (edge case: browser back button)
   - Behavior: Invalidate old secret, generate new QR code
   - Backup codes regenerated on successful enrollment
   - Warning: "Previous authenticator setup canceled. Use this new QR code"

**State Inconsistencies**

1. **Partial TOTP Enrollment (User Abandons Flow)**
   - User views QR code but never completes verification
   - Behavior: TOTP secret remains in "pending" state for 15 minutes
   - After 15 minutes: Secret invalidated, user must restart enrollment
   - totp_enrolled_at remains NULL (enrollment incomplete)

2. **Session Created but TOTP Enrollment Required**
   - Privileged user has valid session but totp_enrolled_at IS NULL
   - Behavior: All routes except `/auth/setup-mfa` return 403 "MFA setup required"
   - Session valid but access restricted (blocking checkpoint)

3. **TOTP Removed While Session Active**
   - User removes TOTP from one device while logged in on another
   - Behavior: Existing sessions remain valid (mfa_verified = true at session creation time)
   - Next login requires TOTP re-enrollment for privileged users

**Boundary Conditions**

1. **Zero Failed Attempts (Fresh Start)**
   - User has never attempted authentication
   - Behavior: No delay, no lockout, full 3 OTP attempts available

2. **Exactly 5 Failed Attempts**
   - User fails 5th attempt (boundary for lockout)
   - Behavior: Immediate 15-minute lockout, lockout_expires_at set
   - Email notification sent
   - Reset failed_attempts_count to 0 after lockout expires

3. **Max Session Limit Reached**
   - User has 5 active sessions (platform max), attempts 6th login
   - Behavior: Invalidate oldest session (by session_created_at), create new session
   - Notification: "Your oldest session was ended due to new login"

4. **Backup Code Exhaustion**
   - User uses all 10 backup codes
   - Behavior: Last code works, then TOTP removed (forces re-enrollment)
   - Warning after 8th code: "2 backup codes remaining. Re-enroll TOTP to generate new codes"

**Deletion/Archival Scenarios**

1. **User Deleted While OTP Pending**
   - Admin deletes user account while OTP email is in transit
   - Behavior: OTP validation fails with "Invalid credentials" (generic error)
   - Email delivered but code unusable
   - Audit: Log deletion event

2. **Tenant Suspended During Authentication**
   - Tenant account suspended mid-authentication flow
   - Behavior: Session creation fails with "Account access suspended. Contact support"
   - Existing sessions immediately invalidated
   - Redirect to tenant-suspended page

3. **Role Downgraded After TOTP Enrollment**
   - Privileged user (enrolled in TOTP) demoted to regular user
   - Behavior: TOTP remains enrolled (user keeps MFA)
   - TOTP verification optional (no longer mandatory)
   - Next login: TOTP prompt shown but "Skip" option available

---

## 4. Non-Functional Criteria (Story-Specific)

**Performance**

- **OTP Generation & Email Queuing:** <100ms API response time (95th percentile)
- **OTP Validation:** <50ms validation latency (excluding database query)
- **Email Delivery:** >99.5% delivered within 2 minutes (including retry attempts)
- **Magic Link Validation:** <200ms (token lookup + session creation)
- **TOTP Verification:** <30ms (cryptographic validation)
- **Concurrent Authentication Requests:** Support 1,000 simultaneous authentication flows per tenant

**Reliability**

- **OTP Delivery Success Rate:** >99.5% (per NFR requirement)
- **Authentication Availability:** 99.9% uptime (excluding planned maintenance)
- **Email Service Resilience:** 3 automatic retries with exponential backoff
- **Session Creation Success Rate:** >99.99% (given successful authentication)

**Security**

- **OTP Entropy:** 6 digits = ~19.9 bits entropy (acceptable for 5-minute expiry + 3-attempt limit)
- **Magic Link Token Entropy:** 256-bit cryptographically secure random (CSPRNG)
- **TOTP Secret Entropy:** 160-bit (per RFC 6238 recommendation)
- **TOTP Secret Storage:** Encrypted at rest (AES-256)
- **Session Token:** HTTP-only cookie (prevents XSS) OR Bearer token (for mobile apps)
- **Rate Limiting:** 5 failed attempts per 15-minute window → 15-minute lockout
- **OTP Resend Rate Limit:** Max 1 request per 30 seconds
- **Brute Force Protection:** Progressive delays (0s → 30s → 15min lockout)
- **Password Transmission:** N/A (passwordless - no plaintext secrets transmitted)

**Scalability**

- **Concurrent Users:** Support 10,000 concurrent authentication flows across all tenants
- **OTP Storage:** Automatic cleanup of expired OTPs (>5 minutes old) via background job
- **Session Storage:** Support 1 million active sessions (distributed session store)
- **Email Queue Throughput:** 10,000 emails/minute (authentication + notifications)

**ExperienceOps KPIs**

- **Clicks to Complete (Standard Login):**
  - Email entry → "Continue" → OTP entry → "Sign In" = **4 clicks**
  - Target: ≤4 clicks for 95% of logins

- **Time to Complete (Standard Login):**
  - Email entry to successful login: **<30 seconds** (median, including email delivery)
  - Target: 80% of logins complete in <45 seconds

- **Time to Complete (TOTP Enrollment):**
  - QR scan to enrollment complete: **<60 seconds** (median)
  - Target: 90% of enrollments complete in <90 seconds

- **Authentication Success Rate:**
  - Successful login on first OTP attempt: **>85%**
  - TOTP enrollment success (no abandonment): **>90%**

- **User Satisfaction:**
  - Post-login satisfaction survey (1-5 scale): **Average ≥4.2**
  - "Login was easy and fast" agreement: **>80%**

- **Error Recovery Rate:**
  - Users who successfully recover from OTP expiry (via resend): **>70%**
  - Users who complete TOTP enrollment after initial failure: **>60%**

**Accessibility**

- **WCAG 2.1 Level AA Compliance:** All UI elements (login form, TOTP enrollment, error messages)
- **Keyboard Navigation:** Full flow completable without mouse
- **Screen Reader Support:** ARIA labels for all form inputs, error announcements
- **Color Contrast:** Error messages, lockout warnings meet 4.5:1 contrast ratio

---

## 5. Use Cases / Scenarios

### Scenario 1: Happy Path - Standard User OTP Login

**Context:**
- User: jane.doe@example.com (regular user, no TOTP enrolled)
- Email exists in tenant "Acme Corp"
- No failed attempts in last 15 minutes
- Email service operational

**Action:**
1. User navigates to `/login`
2. User enters "jane.doe@example.com"
3. User clicks "Continue"
4. User waits 10 seconds, receives email with OTP "482901"
5. User enters "482901" in verification field
6. User clicks "Sign In"

**Expected Outcome:**
1. **Immediate Result:** User authenticated, session created
2. **Side Effects:**
   - Session record created: {session_id, user_id: jane-uuid, tenant_id: acme-uuid, session_expires_at: now+24h, mfa_verified: false}
   - OTP marked as used (otp_used_at = now)
   - failed_attempts_count reset to 0
3. **Audit Events Logged:**
   - `auth.otp_generated {email: jane.doe@example.com, timestamp, ip_address, device_fingerprint}`
   - `auth.session_created {user_id: jane-uuid, email: jane.doe@example.com, timestamp, ip_address, mfa_verified: false}`
4. **UI Feedback:**
   - Redirect to `/dashboard` (default for regular user)
   - Welcome banner: "Welcome back, Jane!"

---

### Scenario 2: Happy Path - Privileged User First Login with TOTP Enrollment

**Context:**
- User: admin@example.com (System Admin role)
- Email exists in tenant
- totp_enrolled_at IS NULL (first login)
- User has Google Authenticator app installed

**Action:**
1. User completes OTP authentication (same as Scenario 1, steps 1-6)
2. System detects privileged role + no TOTP → redirects to `/auth/setup-mfa`
3. User scans QR code with Google Authenticator
4. User enters 6-digit code "752314" from app
5. User clicks "Verify and Continue"
6. System displays backup codes screen
7. User saves backup codes, clicks "I've saved my codes"

**Expected Outcome:**
1. **Immediate Result:** TOTP enrolled, session updated, user authenticated
2. **Side Effects:**
   - totp_secret stored (encrypted)
   - totp_enrolled_at = now
   - totp_device_name = "Authenticator App" (default)
   - 10 backup codes generated and stored (hashed)
   - Session updated: mfa_verified = true
3. **Audit Events Logged:**
   - `auth.session_created {user_id, email, timestamp, mfa_verified: false}` (initial)
   - `auth.totp_enrolled {user_id, email, timestamp, device_name: "Authenticator App"}`
   - Session updated (mfa_verified: true)
4. **UI Feedback:**
   - Redirect to `/admin/dashboard` (default for admin)
   - Success toast: "Two-factor authentication enabled"

---

### Scenario 3: Error - Invalid OTP Code (3 Attempts)

**Context:**
- User: john@example.com
- OTP generated: "123456"
- User misreads email, enters wrong codes

**Action:**
1. User completes email entry, receives OTP "123456"
2. User enters "654321" (incorrect)
3. User enters "234567" (incorrect)
4. User enters "123465" (incorrect)
5. User enters "123456" (correct, but attempts exhausted)

**Expected Outcome:**
1. **Attempt 1:** Error "Invalid code. 2 attempts remaining" (inline error, form preserved)
2. **Attempt 2:** Error "Invalid code. 1 attempt remaining" (yellow warning banner)
3. **Attempt 3:** Error "Too many failed attempts. Request a new code" (OTP invalidated, otp_attempts_remaining = 0)
4. **Attempt 4 (correct code):** Error "Code is no longer valid. Request a new code"
5. **Side Effects:**
   - OTP marked as invalid (otp_attempts_remaining = 0)
   - failed_attempts_count incremented to 3
   - Email entry field preserved (user doesn't re-enter email)
6. **Audit Events:**
   - `auth.authentication_failed {email, reason: "invalid_otp", timestamp, attempts_remaining: 2}` (x3)
7. **UI Feedback:**
   - "Request New Code" button enabled
   - Instruction: "Check your email carefully and try again"

---

### Scenario 4: Error - Account Lockout (Progressive Delay)

**Context:**
- User: attacker@example.com (potential brute force attempt)
- User has failed authentication 4 times in last 10 minutes
- Attempting 5th login

**Action:**
1. User enters email "attacker@example.com"
2. User clicks "Continue"
3. User enters incorrect OTP (5th failure)

**Expected Outcome:**
1. **After 3rd Failure (prior context):**
   - Inline error: "Invalid code. 2 attempts remaining"
2. **After 4th Failure (prior context):**
   - Yellow warning banner: "Too many failed attempts. You can try again in 30 seconds"
   - Submit button disabled for 30 seconds (countdown timer displayed)
3. **After 5th Failure (current action):**
   - Blocking error screen replaces login form
   - Heading: "Account Temporarily Locked"
   - Message: "For your security, this account is locked due to multiple failed login attempts."
   - Countdown: "You can try again in 14:32"
   - Alternative: "Contact your administrator for immediate assistance"
4. **Side Effects:**
   - lockout_expires_at = now + 15 minutes
   - failed_attempts_count reset to 0 (will increment again after lockout expires)
   - Email notification sent to user: "Unusual login activity detected on your account"
5. **Audit Events:**
   - `auth.account_locked {email: attacker@example.com, timestamp, lockout_duration: 900}`
6. **UI Feedback:**
   - Lock icon displayed
   - Live countdown timer updates every second
   - "Contact Admin" button links to support

---

### Scenario 5: Edge Case - OTP Expires During Entry

**Context:**
- User: slow-typer@example.com
- OTP generated at 10:00:00 AM (expires 10:05:00 AM)
- User starts typing at 10:04:45 AM
- User submits at 10:05:10 AM (10 seconds after expiry)

**Action:**
1. User receives OTP "987654" at 10:00 AM
2. User opens email at 10:04:45 AM
3. User types "987654" slowly
4. User clicks "Sign In" at 10:05:10 AM (code expired 10 seconds ago)

**Expected Outcome:**
1. **Immediate Result:** Validation fails, error displayed
2. **Error Message:** "Code expired. Click 'Resend' to get a new code"
3. **Side Effects:**
   - OTP remains marked as expired (not counted against otp_attempts_remaining)
   - failed_attempts_count NOT incremented (expiry is not a failed attempt)
   - Email entry preserved (user doesn't re-enter email)
4. **Audit Events:**
   - `auth.authentication_failed {email, reason: "otp_expired", timestamp}`
5. **UI Feedback:**
   - "Resend Code" button highlighted (green, pulsing animation)
   - Instruction: "This code has expired. Request a new one to continue"
   - Email address field preserved (autofilled with slow-typer@example.com)

---

### Scenario 6: Edge Case - Role Downgrade After TOTP Enrollment

**Context:**
- User: former-admin@example.com (was System Admin, now Regular User)
- User enrolled TOTP when they were admin (totp_enrolled_at = 2025-09-01)
- Admin demoted user role to "Regular User" on 2025-10-01
- User attempts login on 2025-10-02

**Action:**
1. User completes OTP authentication (receives session)
2. System checks role: Regular User (not privileged)
3. System checks totp_enrolled_at: NOT NULL (TOTP still enrolled)
4. User is prompted for TOTP verification (optional for regular users)
5. User sees "Skip for now" button (not shown to privileged users)
6. User clicks "Skip for now"

**Expected Outcome:**
1. **Immediate Result:** User authenticated without TOTP verification
2. **Side Effects:**
   - Session created with mfa_verified: false
   - totp_secret remains in database (not deleted)
   - User can access platform without TOTP
3. **Audit Events:**
   - `auth.session_created {user_id, email, timestamp, mfa_verified: false}`
   - `auth.totp_skipped {user_id, email, timestamp}` (for security monitoring)
4. **UI Feedback:**
   - Redirect to `/dashboard`
   - Info banner (dismissible): "You have two-factor authentication enabled. You can manage it in Profile Settings"

---

## 6. UX / Design Reference

**Figma Link:** [To be added by UX team]

**Primary Screens/States:**

1. **Login Screen** (`/login`)
   - Email input field (autofocus)
   - "Continue" button (primary CTA)
   - "Send magic link instead" (text link)
   - Tenant branding (logo + custom message)
   - Empty state: Clean, minimal design
   - Error state: Inline email format validation

2. **OTP Verification Screen** (`/login` - step 2)
   - Email confirmation (masked)
   - 6-digit OTP input (numeric keypad on mobile)
   - Countdown timer (5:00 → 0:00)
   - "Resend Code" link (disabled for 30s)
   - "Send magic link instead" option
   - Error states: Invalid code, expired code, attempts remaining

3. **Progressive Lockout States**
   - Warning banner (4th failure): Yellow, countdown timer, disabled submit
   - Lockout screen (5+ failures): Full-page blocking error, lock icon, countdown

4. **TOTP Enrollment Screen** (`/auth/setup-mfa`)
   - Blocking modal (full-screen overlay, no dismiss)
   - QR code display (dynamic, regenerates on reload)
   - Manual entry collapsible section
   - 6-digit TOTP input
   - Supported apps list (icons)
   - Error state: Invalid TOTP code

5. **Backup Codes Display** (one-time view after TOTP enrollment)
   - 10 codes displayed (4x3 grid)
   - "Download as .txt" button
   - "Copy to Clipboard" button
   - Warning: "Save these codes. You won't see them again"
   - "I've Saved My Codes" button (primary CTA)

6. **Self-Service TOTP Management** (`/profile/security`)
   - Adaptive canvas (right panel slide-in)
   - Status badge (Active/Not Configured)
   - Enrolled device info + date
   - "Enable/Remove MFA" buttons
   - "View Backup Codes" (requires TOTP verification)

**Key Flows:**

1. **Standard Login Flow:** Email → Continue → Check email → Enter OTP → Sign In → Dashboard
2. **First-Time Privileged User:** (Standard Login) → TOTP Enrollment → Scan QR → Enter Code → Save Backup Codes → Dashboard
3. **Subsequent Privileged User Login:** (Standard Login) → TOTP Verification → Dashboard
4. **Progressive Lockout:** Email → OTP (fail 3x) → 30s delay warning → OTP (fail 2x) → 15min lockout screen
5. **Self-Service TOTP Enrollment:** Profile → Security → Enable MFA → Scan QR → Enter Code → Save Backup Codes

**Design Notes (Dynamic Behavior):**

- **Countdown Timers:** Update live every second (OTP expiry, lockout duration)
- **Resend Button State:** Disabled (gray) for 30s after click, then enabled (green) with tooltip "Click to resend"
- **Email Masking:** Display as `j***@example.com` (show first char + domain)
- **QR Code Regeneration:** New QR code on page reload (invalidates previous secret)
- **Backup Codes Download:** Plain text file named `oneops-backup-codes-{timestamp}.txt`

**Platform Patterns Applied:**

- **Chat-First:** Not applicable for authentication (security constraint - chat access requires authentication)
- **Adaptive Canvas:** TOTP management uses right panel (preserves profile settings context)
- **AI Augmentation:** Email domain detection for SSO suggestion (future enhancement - badge/banner)
- **Dynamic Navigation:** Post-login redirect to context-appropriate dashboard (admin vs regular user)
- **Context Preservation:** Failed authentication preserves email field (no re-entry required)

---

## 7. Dependencies (Functional)

**Prerequisites:**

1. **Email/Notification Service Configuration**
   - Requirement: Transactional email service (SendGrid, AWS SES, or similar) configured and operational
   - Impact: Cannot send OTP/magic link without email capability
   - Test Criteria: Email service must achieve >99.5% delivery rate

2. **User Profile Service**
   - Requirement: User entity exists with email, role, tenant_id fields
   - Impact: Authentication requires user lookup by email
   - Dependency: User creation flow must precede authentication (user must exist in system)

3. **Session Management Service**
   - Requirement: Session storage (Redis, database, or distributed cache) configured
   - Impact: Cannot persist authenticated sessions without session store
   - Test Criteria: Session store must support 1M+ concurrent sessions

4. **Tenant Provisioning**
   - Requirement: Tenant entity exists with branding configuration
   - Impact: Login screen requires tenant-specific branding (logo, custom message)
   - Dependency: Tenant setup must be completed before users can authenticate

**Enables:**

1. **CH_002: User Lifecycle Management**
   - Capability Unlocked: Authenticated users can access profile management, role assignment, user deactivation
   - Workflow: User must authenticate before performing lifecycle operations

2. **CH_003: Session Management**
   - Capability Unlocked: Session refresh, logout, concurrent session management
   - Workflow: Session created by authentication is managed by session service

3. **CH_004: Role-Based Access Control (RBAC)**
   - Capability Unlocked: Authenticated session includes role/permissions for authorization checks
   - Workflow: Authentication establishes user identity, RBAC enforces resource access

4. **All Protected Platform Features**
   - Capability Unlocked: Access to ITSM, ITOM, CRM modules (all require authentication)
   - Workflow: Authentication is gateway to entire platform

---

## 8. References

- **Parent PRD:** `PRD_USER_ACCESS`, Section 3.1 (User Authentication), Lines 45-89
- **Parent Chapter Plan:** `.specify/prds/PRD_USER_ACCESS_chapter_plan.md`, Chapter `CH_001`
- **Platform Vision:** `.specify/memory/platform_overview.md` (AI-native core, ExperienceOps, multi-tenancy principles)
- **Figma Design:** [To be added by UX team]
- **API Contract:** [To be defined by dev team - RESTful API endpoints for OTP generation, validation, TOTP enrollment]
- **Data Model:** [To be defined by dev team - OTP, MagicLink, TOTPConfig, Session entities]
- **Event Schema:** [To be defined by dev team - Audit event payloads for auth.* events]
- **Discovery Log:** `.specify/dfs/PRD_USER_ACCESS/CH_001_discovery_log.md`

**Relevant RFCs & Standards:**

- **RFC 6238:** TOTP: Time-Based One-Time Password Algorithm (TOTP implementation)
- **RFC 5322:** Internet Message Format (email validation regex)
- **WCAG 2.1:** Web Content Accessibility Guidelines (UI accessibility compliance)
- **OWASP Authentication Cheat Sheet:** Security best practices (rate limiting, lockout policies)

---

## 9. Validation Checklist

- [x] Metadata complete (Section 1)
- [x] User story in standard format (Section 2)
- [x] Acceptance criteria are atomic and testable (Section 3)
- [x] All fields have complete definitions (Section 3.2)
- [x] Business logic is step-by-step and unambiguous (Section 3.3)
- [x] All error conditions documented (Section 3.4)
- [x] External interactions specified (Section 3.5)
- [x] Edge cases explicitly handled (Section 3.6)
- [x] Story-specific NFRs identified (Section 4)
- [x] Use cases cover happy path + errors + edge cases (Section 5)
- [x] UX references present with placeholders (Section 6)
- [x] Dependencies traced to parent chapter (Section 7)
- [x] All references complete (Section 8)
- [x] Platform vision applied (next-gen UX patterns - adaptive canvas, AI touchpoints)
- [x] ExperienceOps KPIs included (clicks to complete, time to complete, satisfaction metrics)
- [x] No orphan logic (all rules sourced from chapter or discovery decisions)
- [x] Multi-tenancy functional constraints included (data scoping)
- [x] Permissions documented (public vs authenticated routes)
- [x] Audit logging specified (event names + key fields)

---

## 10. Change Log

- **v1** — 2025-10-03 — Initial DFS generated from chapter CH_001 with interactive discovery (5 questions answered)
  - Discovery decisions: Progressive lockout (Q1-C), MFA enforcement at first login (Q2-A), silent retry for OTP delivery (Q3-B), default to OTP code over magic link (Q4-Custom), self-service TOTP enrollment (Q5-A)
  - Platform context: Full platform_overview.md loaded for AI-native UX patterns
  - Next-gen UX applied: Adaptive canvas for TOTP management, progressive lockout UX, ExperienceOps KPIs

---

*End of DFS - CH_001 Passwordless Authentication*
