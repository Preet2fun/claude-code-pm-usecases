# FRD 1.1: Basic Tenant Provisioning

**FRD ID:** FRD_1.1_BASIC_TENANT_PROVISIONING
**Version:** v1.0
**Date:** 2025-10-26
**Based on PRD:** PRD_TENANT_ONBOARDING v1.0
**Priority:** P0 (Critical)
**Phase:** Phase 1 - MVP Core
**Sprint:** Sprint 1 (Weeks 1-2)

---

## Document Purpose

This Functional Requirements Document (FRD) defines the **functional behavior, user flows, and business logic** for the Basic Tenant Provisioning capability. This document is intended for:

- **UX Designers:** To create wireframes, Figma designs, and user experience flows
- **Software Architects:** To design technical contracts (APIs, data models, event schemas)
- **Developers:** To implement the functional requirements
- **QA Engineers:** To create test cases and validate acceptance criteria

**Note:** This document focuses on WHAT the system does, not HOW it's technically implemented. Technical specifications (API endpoints, database schemas, event schemas) will be defined separately by the software architect and development team.

---

## 1. Executive Summary

The Basic Tenant Provisioning capability enables **passwordless self-service tenant signup** and fully automated workspace provisioning. A user signs up with just their email, verifies their identity via magic link or OTP, and receives a fully functional workspace with a unique subdomain within 60 seconds—with automatic login. The system handles provisioning failures gracefully with automatic rollback and cleanup.

**Key Outcomes:**
- Passwordless signup (email-only, no password required) with magic link + OTP dual verification
- Self-service tenant creation with zero manual intervention
- Instant auto-login after email verification
- Unique subdomain and workspace for each tenant
- Automatic rollback on provisioning failure (no orphaned resources)
- Welcome email with workspace access details
- Lightweight pricing page integration (plan query parameter support)
- Enhanced progress screen with engaging content during provisioning

---

## 2. Scope

### In-Scope (Functional)
- **Passwordless signup flow (email-only registration with dual verification: Magic Link + OTP)**
- Email verification mechanism with dual options (click magic link OR enter OTP)
- Instant auto-login after email verification (magic link creates authenticated session)
- OTP validation screen for users who prefer entering code
- Subdomain generation and uniqueness validation
- Workspace provisioning automation
- Initial admin account creation (passwordless, email-based authentication)
- Default workspace configuration
- Provisioning failure detection and rollback
- Welcome email notification
- Basic anti-fraud checks (rate limiting, CAPTCHA)
- **Lightweight pricing page integration (plan query parameter support)**
- **Enhanced progress screen with engaging content (feature highlights, tips during provisioning)**

### Out-of-Scope (Deferred or Not Included)
- Password-based authentication (fully passwordless in MVP)
- Social login (Google, Microsoft) - Phase 2
- Custom domain support (e.g., tenant.customdomain.com) - Phase 2
- Multi-region deployment selection - Phase 2
- Assisted/manual provisioning workflow for enterprise - FRD 1.3
- Payment integration during signup - FRD 2.1
- Full pricing page UI (lightweight integration only via query parameter)
- Guided onboarding wizard (deferred to post-provisioning FRD 1.5 or Phase 2)
- Module template selection during signup (default template applied automatically)
- Advanced fraud detection (ML-based) - Future

---

## 3. User Roles & Actors

### Primary Actors

1. **Prospective Tenant (Anonymous User)**
   - **Description:** Person signing up to create a new tenant workspace
   - **Permissions:** Can access signup page, submit signup form (email only), verify email via magic link or OTP
   - **Goal:** Create a new workspace and become the tenant admin with instant access (no password needed)

2. **Tenant Admin (Post-Signup)**
   - **Description:** The first user who signed up for the tenant
   - **Permissions:** Full administrative access to their tenant workspace
   - **Goal:** Access workspace (auto-logged-in after verification), configure settings, invite users
   - **Authentication:** Passwordless (future logins via magic link or SSO per FRD 1.2)

3. **System (Automated Actor)**
   - **Description:** Backend automation that provisions workspace
   - **Permissions:** Create databases, allocate resources, send emails, generate magic links and OTPs
   - **Goal:** Provision workspace successfully or rollback on failure

### Secondary Actors

4. **Platform Admin (Indirect)**
   - **Description:** Platform operator who monitors provisioning health
   - **Permissions:** View provisioning status, troubleshoot failures (via FRD 1.3)
   - **Interaction:** Receives alerts on provisioning failures, can view logs

---

## 4. Functional Flows

### 4.1 Main Flow: Successful Self-Service Passwordless Signup & Provisioning

**Preconditions:**
- User has valid email address
- User has internet browser access to signup page
- Backend provisioning infrastructure is operational

**Flow:**

#### Step 1: Access Signup Page (with Optional Plan Parameter)
1. User navigates to public signup URL (e.g., `https://motadata.com/signup` or `https://motadata.com/signup?plan=free`)
2. System displays simplified passwordless signup form
3. If `plan` query parameter present, system displays plan badge (e.g., "Free Plan" or "Professional Trial")

**UI Elements (Functional):**
- Page title: "Create Your Workspace"
- Subtitle: "Get started with Motadata in seconds—no password required"
- **Plan badge (if plan parameter present):** "Free Plan" or "[Plan Name] - 14-day Trial" (read-only, informational)
- Form fields (see section 5.1 for field specifications):
  - Organization Name (required)
  - Email Address (required)
  - Desired Subdomain (optional - auto-generated if blank)
  - Terms of Service checkbox (required)
- CAPTCHA challenge (if rate limit exceeded)
- **No password field** (passwordless signup)
- "Create Workspace" button (primary CTA)
- Footer links: "Already have an account? Sign in"

**Plan Parameter Handling:**
- If `plan=free`: Display "Free Plan" badge, provision with freemium tier defaults
- If `plan=professional`: Display "Professional Plan - 14-day Trial" badge, provision with professional tier defaults
- If `plan=enterprise`: Display "Enterprise Plan - 14-day Trial" badge, provision with enterprise tier defaults
- If no plan parameter: Default to freemium tier

---

#### Step 2: User Enters Signup Information (Email Only, No Password)
1. User fills in required fields:
   - Organization Name
   - Email Address
   - Desired Subdomain (optional - auto-generated if blank)
2. User completes CAPTCHA (if presented)
3. User checks "I agree to Terms of Service" checkbox
4. User clicks "Create Workspace" button

**System Actions:**
- Validate form inputs (see section 5.2 for validation rules)
- Check subdomain uniqueness (if provided)
- Auto-generate subdomain if not provided (see section 5.3)
- Check if email already exists in system
- Perform rate limit check (max 5 signups per IP per hour)
- Capture plan parameter (if present) for workspace provisioning

**Validation Results:**
- **Success:** Proceed to Step 3
- **Failure:** Display inline error messages, keep form data populated, allow retry

---

#### Step 3: System Sends Verification Email with Magic Link + OTP
1. System creates tenant record in "Pending" state
2. System generates:
   - **Magic Link Token:** Secure token (UUID v4, valid for 24 hours, single-use)
   - **OTP Code:** 6-digit numeric code (valid for 15 minutes, single-use)
3. System sends verification email containing BOTH magic link AND OTP
4. System displays "Check Your Email" confirmation screen with OTP entry option

**UI Elements (Functional):**
- Success message: "We've sent a verification email to [email]"
- Instructions:
  - "**Option 1:** Click the verification link in the email to instantly access your workspace"
  - "**Option 2:** Enter the 6-digit code below to verify"
- **OTP Entry Section:**
  - Label: "Enter 6-digit code from email"
  - OTP input field (6 digits, auto-focus, auto-submit on 6th digit)
  - "Verify Code" button
  - Character countdown: "Code expires in 14:32"
- Secondary action: "Didn't receive email? Resend" button (available after 60 seconds)
- Email client shortcuts (Gmail, Outlook icons - open webmail)

**System Actions:**
- Store tenant record with state = "Pending"
- Store magic link token and OTP with expiration timestamps (24 hours for magic link, 15 minutes for OTP)
- Queue verification email for delivery (within 60 seconds)
- Log signup event in audit trail
- Store plan parameter (if present) in tenant metadata

---

#### Step 4a: User Verifies Email via Magic Link (Option 1 - Recommended)
1. User opens email client
2. User clicks magic link in email (e.g., `https://motadata.com/verify?token=xxxxx`)
3. System validates magic link token
4. System creates authenticated session for user (auto-login)
5. System transitions tenant state from "Pending" to "Provisioning"
6. System triggers provisioning workflow (async)
7. System displays "Setting Up Your Workspace" progress screen (user already logged in)

**UI Elements (Functional):**
- Automatic authentication (user logged in, no password needed)
- Progress screen displayed while provisioning in background
- User can see email address and organization name in header (authenticated state)

**System Actions:**
- Validate magic link token (not expired, not already used, matches tenant in Pending state)
- Create authenticated session (JWT token or session cookie)
- Update tenant state to "Provisioning"
- Trigger async provisioning job
- Log verification event in audit trail (method: magic_link)

---

#### Step 4b: User Verifies Email via OTP (Option 2 - Alternative)
1. User remains on "Check Your Email" page with OTP entry field
2. User opens email, copies 6-digit OTP code
3. User enters OTP code in input field
4. User clicks "Verify Code" button (or code auto-submits after 6th digit)
5. System validates OTP code
6. System creates authenticated session for user (auto-login)
7. System transitions tenant state from "Pending" to "Provisioning"
8. System triggers provisioning workflow (async)
9. System displays "Setting Up Your Workspace" progress screen

**UI Elements (Functional):**
- OTP input field: 6-digit numeric input (can be auto-split into 6 boxes for better UX)
- Real-time validation (highlight invalid characters, show green checkmark when 6 digits entered)
- Error display below input: "Invalid code. Please check and try again." (if validation fails)
- Countdown timer: "Code expires in 14:32" (real-time countdown)

**System Actions:**
- Validate OTP code (matches stored OTP, not expired, not already used)
- Create authenticated session (JWT token or session cookie)
- Update tenant state to "Provisioning"
- Trigger async provisioning job
- Log verification event in audit trail (method: otp)

**Validation Results:**
- **Success:** User authenticated, proceed to provisioning progress screen
- **Failure (invalid OTP):** Display error "Invalid code. Please try again." (3 attempts allowed before requiring new email)
- **Failure (expired OTP):** Display error "Code expired. [Request new code]" button

---

#### Step 5: System Provisions Workspace (Automated, User Sees Enhanced Progress Screen)

**Provisioning Steps (Sequential - Backend):**

1. **Allocate Infrastructure Resources**
   - Reserve compute resources based on plan tier (if applicable)
   - Allocate storage space (default quota per plan: Free=10GB, Professional=50GB, Enterprise=100GB)
   - Provision networking (if needed)

2. **Create Tenant Database/Schema**
   - Create dedicated database or schema (depending on architecture)
   - Apply database migrations
   - Set up database user and permissions

3. **Initialize Workspace Configuration**
   - Apply configuration template based on plan parameter (free/professional/enterprise default templates)
   - Create initial admin user account (email-based, passwordless)
   - Generate default roles and permissions
   - Set timezone to UTC (or detected from signup location)
   - Set default locale to English (or detected from browser)

4. **Finalize Subdomain and DNS**
   - Confirm subdomain availability
   - Register subdomain in DNS (if external DNS)
   - Configure routing/load balancer

5. **Transition Tenant to Active State**
   - Update tenant state from "Provisioning" to "Active"
   - Record provisioning completion timestamp
   - Log successful provisioning event

**Enhanced Progress Screen (User Experience - Frontend):**

While provisioning runs in background, user sees engaging content:

- **Animated Progress Indicator:** Pulsing logo or spinner with progress substeps
- **Primary Status Message:** "Setting Up Your Workspace... This usually takes 30-60 seconds"
- **Progress Substeps (Real-time Updates if available):**
  - ✓ Creating your database
  - ⏳ Configuring your workspace (current step, animated)
  - ⏸ Finalizing setup (pending)

- **Engaging Content Carousel (Auto-rotates every 5 seconds):**
  - **Slide 1: Feature Highlight**
    - Title: "Powerful IT Service Management"
    - Description: "Manage incidents, requests, and changes with built-in ITSM workflows"
    - Icon: Service desk illustration

  - **Slide 2: Quick Start Tip**
    - Title: "Get Started Quickly"
    - Description: "Once your workspace is ready, you can invite team members and customize settings"
    - Icon: Team collaboration illustration

  - **Slide 3: Integration Preview**
    - Title: "Connect Your Tools"
    - Description: "Integrate with Slack, Microsoft Teams, and 50+ other tools"
    - Icon: Integration icons

  - **Slide 4: Support Resource**
    - Title: "We're Here to Help"
    - Description: "Access our knowledge base, tutorials, and 24/7 support anytime"
    - Icon: Support illustration

- **Did You Know? Tips (Optional, Displayed Below Carousel):**
  - "💡 Tip: You can customize your workspace URL later in Settings"
  - "💡 Tip: Free plan includes unlimited team members"
  - "💡 Tip: Your data is automatically backed up daily"

**Success Criteria (per step):**
- Each step completes within timeout (30 seconds per step, 120 seconds total)
- No errors encountered
- All resources successfully allocated

**On Success:** Proceed to Step 6
**On Failure:** Trigger rollback (see section 4.3)

---

#### Step 6: System Notifies User of Successful Provisioning

1. System sends welcome email with workspace details
2. System auto-redirects user to workspace dashboard (user already authenticated from magic link/OTP)

**UI Elements (Functional):**
- Success screen: "Your Workspace is Ready!" (optional, or skip directly to dashboard)
- Workspace details (if shown):
  - Workspace URL: `https://[subdomain].motadata.com`
  - Admin email: [user's email]
- Call-to-action button: "Go to Dashboard" (or auto-redirect after 3 seconds)
- Additional resources:
  - "Getting Started Guide" link
  - "Invite Team Members" link
  - "Contact Support" link

**Welcome Email Contents:**
- Subject: "Welcome to Motadata - Your Workspace is Ready!"
- Body:
  - Greeting: "Hi [Name], welcome to Motadata!"
  - Workspace URL: `https://[subdomain].motadata.com`
  - Next steps: Setup checklist (customize profile, invite team, explore features)
  - **Future login instructions:** "To access your workspace in the future, simply enter your email and we'll send you a magic link"
  - Support contact information
  - Unsubscribe link (for non-transactional emails)

**System Actions:**
- Update tenant state to "Active" (if not already)
- Queue welcome email for delivery (within 2 minutes)
- Log workspace access event in audit trail
- Auto-redirect to workspace dashboard

---

#### Step 7: User Accesses Workspace (Already Authenticated)
1. User auto-redirected to workspace dashboard: `https://[subdomain].motadata.com/dashboard`
2. User already logged in (authenticated session from magic link/OTP verification)
3. User lands on workspace dashboard with onboarding checklist or welcome tour (post-provisioning onboarding, deferred to future FRD)

**No password login required** - User authenticated during email verification step

**End of Main Flow**

---

### 4.2 Alternate Flow: Email Verification Resend

**Trigger:** User clicks "Resend Verification Email" on confirmation screen

**Preconditions:**
- User is on "Check Your Email" screen
- At least 60 seconds have passed since last send
- Verification not yet completed

**Flow:**
1. User clicks "Resend Verification Email" button
2. System validates resend eligibility (rate limit: max 3 resends per tenant)
3. System generates:
   - New magic link token (invalidates previous)
   - New OTP code (invalidates previous)
4. System queues email for delivery
5. System displays confirmation: "Verification email resent. Please check your inbox."
6. OTP countdown timer resets to 15:00

**Rate Limit Behavior:**
- First resend: Available after 60 seconds
- Second resend: Available after 120 seconds
- Third resend: Available after 300 seconds
- After 3 resends: Display message "Please contact support if you continue to have issues"

**UI Feedback:**
- Button disabled during send operation (show spinner)
- Button disabled with countdown timer until next resend available
- Success message shown after resend
- OTP input field cleared and ready for new code

---

### 4.3 Alternate Flow: Provisioning Failure & Automatic Rollback

**Trigger:** Any step in provisioning workflow fails

**Failure Scenarios:**
- Database creation fails (insufficient resources, timeout)
- DNS/subdomain registration fails
- Configuration template application fails
- Network connectivity issue during provisioning

**Rollback Flow:**

#### Step 1: Detect Failure
1. System detects failure in provisioning step
2. System logs failure details (step failed, error message, timestamp)
3. System transitions tenant state to "Provisioning_Failed"
4. System triggers rollback workflow (async)

---

#### Step 2: Execute Rollback (Automated)
1. **Cleanup Resources (Reverse Order of Provisioning):**
   - Remove DNS/subdomain registration
   - Delete tenant database/schema
   - Deallocate infrastructure resources
   - Remove any partial configurations

2. **Verify Cleanup Completion:**
   - Confirm all resources removed (no orphans)
   - Verify subdomain available again
   - Confirm database/schema deleted

3. **Update Tenant State:**
   - Transition tenant from "Provisioning_Failed" to "Failed" (permanent)
   - Record rollback completion timestamp
   - Log rollback event in audit trail

**Rollback Timeout:** Maximum 5 minutes for complete cleanup

---

#### Step 3: Notify User of Failure
1. System displays error message on progress screen (if user still on page)
2. System sends failure notification email

**UI Elements (Functional):**
- Error screen: "We encountered an issue setting up your workspace"
- Error message: "We're sorry, but we couldn't complete your workspace setup due to a technical issue. Our team has been notified."
- Reference ID: "Reference: [TENANT_ID or ERROR_ID]"
- Call-to-action: "Try Again" button (allows user to retry signup)
- Support contact: "If the issue persists, please contact support@motadata.com with reference ID above"

**Failure Email Contents:**
- Subject: "Action Required: Workspace Setup Issue"
- Body:
  - Apology for inconvenience
  - Explanation: Technical issue during setup
  - Next steps: Try signing up again, or contact support
  - Support contact information
  - Reference ID for support (tenant ID or transaction ID)

**System Actions:**
- Queue failure email for delivery (within 5 minutes)
- Alert platform admin via monitoring system (if failure rate exceeds threshold)
- Retry provisioning automatically after 5 minutes (max 2 auto-retries)

---

#### Step 4: User Retry or Contact Support
1. **Option A: User clicks "Try Again"**
   - System redirects to signup page
   - Form pre-populated with previous signup data (email, name, subdomain)
   - User submits again (no email verification needed if already verified, but new magic link/OTP sent for security)
   - System attempts provisioning again

2. **Option B: User contacts support**
   - User provides reference ID from email
   - Support team investigates and manually provisions if needed (FRD 1.3)

**End of Alternate Flow**

---

### 4.4 Alternate Flow: OTP Validation Failure

**Trigger:** User enters incorrect OTP code

**Flow:**
1. User enters 6-digit OTP code in input field
2. User clicks "Verify Code" button
3. System validates OTP code
4. System detects code mismatch or expired
5. System displays inline error message

**UI Elements (Functional):**
- Error message below OTP input: "Invalid code. Please check and try again."
- Remaining attempts counter: "2 attempts remaining" (if implementing attempt limit)
- Input field highlighted in red
- Option to request new code: "Didn't receive code or expired? [Resend Email]"

**Validation Logic:**
- **Attempt 1-2:** Display error, allow retry
- **Attempt 3 (final):** Display error + "Maximum attempts reached. [Request new code]" button
- After 3 failed attempts: Require new email resend (generates new OTP)

**Error Types:**
- **Invalid OTP:** "Invalid code. Please check and try again."
- **Expired OTP (>15 minutes):** "Code expired. [Request new code]"
- **Already Used OTP:** "This code has already been used. [Request new code]"

---

### 4.5 Alternate Flow: Magic Link Token Expired

**Trigger:** User clicks magic link after 24 hours

**Flow:**
1. User clicks magic link in email
2. System validates token and checks expiration
3. System detects token expired (>24 hours old)
4. System displays error message

**UI Elements (Functional):**
- Error screen: "Verification Link Expired"
- Message: "This verification link has expired. Verification links are valid for 24 hours."
- Call-to-action: "Request New Verification Email" button
- Alternative: "Start over with a new signup" link

**On "Request New Verification Email":**
1. System generates new magic link token and OTP
2. System sends new verification email
3. System displays "Check Your Email" screen with OTP entry option

---

### 4.6 Alternate Flow: Subdomain Already Taken

**Trigger:** User enters subdomain that's already in use

**Flow:**
1. User enters desired subdomain in signup form
2. User clicks "Create Workspace" button
3. System validates subdomain uniqueness (checks database)
4. System detects subdomain already exists
5. System displays inline error message below subdomain field

**UI Elements (Functional):**
- Error message: "This subdomain is already taken. Please try another."
- Suggested alternatives (optional):
  - "[original]-1"
  - "[original]-2"
  - "[original]-[random]"
- User can modify subdomain and resubmit

**Validation Timing:**
- Real-time validation: Check on blur (when user leaves field)
- Final validation: Check on form submit

---

### 4.7 Alternate Flow: Email Already Registered

**Trigger:** User enters email that already exists in system

**Flow:**
1. User enters email that's already registered
2. User clicks "Create Workspace" button
3. System detects email exists in database
4. System displays inline error message below email field

**UI Elements (Functional):**
- Error message: "An account with this email already exists."
- Call-to-action link: "Sign in instead" (redirects to login page where user can request magic link)
- Additional info: "We'll send you a sign-in link if you've forgotten how to access your workspace"

**Security Note:** To prevent email enumeration attacks, consider alternative approach:
- Display generic message: "If this email is registered, we'll send a verification link"
- Send email to existing account holder: "Someone tried to sign up with your email"
- Do not reveal whether email exists or not

**Recommended Approach for MVP:** Use direct error message for better UX (configurable based on security policy)

---

## 5. Business Logic & Validation Rules

### 5.1 Signup Form Fields (Passwordless)

| Field Name | Type | Required | Validation Rules | Notes |
|------------|------|----------|------------------|-------|
| **Organization Name** | Text | Yes | - Min 2 characters<br>- Max 100 characters<br>- Letters, spaces, hyphens, apostrophes only<br>- No leading/trailing spaces | Used for personalization and subdomain auto-generation |
| **Email Address** | Email | Yes | - Valid email format (RFC 5322)<br>- Max 255 characters<br>- Must not exist in system<br>- Domain not in blocklist | Primary identifier for tenant admin and passwordless authentication |
| **Desired Subdomain** | Text | No | - Min 3 characters<br>- Max 63 characters<br>- Lowercase alphanumeric and hyphens only<br>- Must start with letter<br>- Must end with alphanumeric<br>- No consecutive hyphens<br>- Must be unique (globally)<br>- Not in reserved words list | Auto-generated if blank: sanitized org name + random suffix |
| **Plan Parameter (Hidden)** | Hidden | No | - Values: free, professional, enterprise<br>- Captured from query parameter | Used to apply plan-specific defaults during provisioning |
| **Terms of Service Agreement** | Checkbox | Yes | - Must be checked | Link opens ToS in new tab |
| **CAPTCHA** | CAPTCHA | Conditional | - Valid CAPTCHA response | Only shown if rate limit threshold exceeded (>5 signups per IP per hour) |

**Note:** **No password field** in this FRD (fully passwordless signup)

---

### 5.2 Validation Rules Details

#### Email Validation
- **Format Check:** Use standard email regex or validation library
- **Domain Validation:** Check MX records exist (optional, for better UX)
- **Disposable Email Detection:** Block known disposable email providers (e.g., mailinator, temp-mail)
- **Typo Detection:** Suggest corrections for common typos (e.g., "gmial.com" → "Did you mean gmail.com?")

**Error Messages:**
- "Please enter a valid email address"
- "Disposable email addresses are not allowed"
- "This email address is already registered. [Sign in instead]"

---

#### OTP Validation
- **Format Check:** Exactly 6 numeric digits
- **Code Match:** Must match stored OTP for tenant
- **Expiration Check:** Must be used within 15 minutes of generation
- **Single-Use:** OTP invalidated after successful verification
- **Attempt Limit:** Max 3 attempts per OTP (requires new email after 3 failures)

**Error Messages:**
- "Please enter a 6-digit code"
- "Invalid code. Please check and try again. [X attempts remaining]"
- "Code expired. [Request new code]"
- "Maximum attempts reached. [Request new code]"

---

#### Magic Link Token Validation
- **Format Check:** UUID v4 format
- **Token Match:** Must match stored token in database
- **Expiration Check:** Must be used within 24 hours of generation
- **Single-Use:** Token invalidated after successful verification
- **State Check:** Tenant must be in "Pending" state

**Error Messages:**
- "Invalid verification link"
- "Verification link expired. [Request new link]"
- "This link has already been used. [Request new link]"

---

#### Subdomain Validation
- **Real-Time Check:** Validate on blur and on form submit
- **Character Restrictions:** Lowercase alphanumeric and hyphens only (automatically convert to lowercase)
- **Reserved Words:** Block subdomains: `www`, `api`, `admin`, `app`, `mail`, `ftp`, `localhost`, `test`, `staging`, `dev`, `prod`, `help`, `support`, `status`, `blog`, `docs`, `cdn`, `static`, `assets`
- **Profanity Filter:** Block offensive words

**Error Messages:**
- "Subdomain must be at least 3 characters long"
- "Subdomain can only contain lowercase letters, numbers, and hyphens"
- "This subdomain is already taken. Try [suggestion-1], [suggestion-2]"
- "This subdomain is reserved. Please choose another."

---

### 5.3 Subdomain Auto-Generation Logic

If user leaves subdomain field blank:

1. **Sanitize Organization Name:**
   - Convert to lowercase
   - Remove special characters (keep only alphanumeric)
   - Replace spaces with hyphens
   - Trim to max 30 characters
   - Example: "Acme Corporation!" → "acme-corporation"

2. **Check Uniqueness:**
   - Query database for subdomain
   - If available: Use sanitized name
   - If taken: Append random 4-digit number
   - Example: "acme-corporation-8472"

3. **Retry Logic:**
   - Max 3 attempts with different random suffixes
   - If all fail: Generate fully random subdomain (e.g., "workspace-a8f3b2")

**Display to User:**
- Show auto-generated subdomain on confirmation screen: "Your workspace URL: https://[subdomain].motadata.com"

---

### 5.4 Magic Link & OTP Generation Logic

#### Magic Link Token Generation
1. **Format:** UUID v4 (e.g., "550e8400-e29b-41d4-a716-446655440000")
2. **Storage:** Store hashed token in database using SHA-256 (not plain text)
3. **Expiration:** 24 hours from generation timestamp
4. **Single-Use:** Mark as "used" in database after successful verification
5. **URL Format:** `https://motadata.com/verify?token=[TOKEN]`

**Security:**
- Token long enough to prevent brute force (128-bit UUID = 2^128 combinations)
- HTTPS required for token transmission
- Token invalidated after use or expiration

---

#### OTP Code Generation
1. **Format:** 6-digit numeric code (e.g., "123456")
2. **Generation:** Cryptographically secure random number generator
3. **Storage:** Store hashed OTP in database using bcrypt or SHA-256
4. **Expiration:** 15 minutes from generation timestamp
5. **Single-Use:** Mark as "used" in database after successful verification

**Security:**
- 6 digits = 1,000,000 combinations (sufficient with rate limiting)
- Rate limit: Max 3 attempts per OTP
- Short expiration (15 minutes) reduces attack window

---

#### Dual Verification Email Template (Magic Link + OTP)

**Subject:** Verify your email to activate your Motadata workspace

**From:** Motadata <noreply@motadata.com>

**Body (Plain Text):**
```
Hi [Name],

Thanks for signing up for Motadata!

You can verify your email in two ways:

OPTION 1: Click the link below to instantly access your workspace
[Magic Link Button]

OPTION 2: Enter this 6-digit code on the verification page
Your verification code: 123456
(Code expires in 15 minutes)

This verification link will expire in 24 hours.

If you didn't sign up for Motadata, you can safely ignore this email.

Thanks,
The Motadata Team

---
[Company Address]
```

**Body (HTML):**
- Branded header with logo
- Personalized greeting
- **Prominent call-to-action button:** "Verify Email & Access Workspace" (magic link)
- **Secondary option:** Large, bold 6-digit OTP code displayed in box
- Clear instructions for both verification methods
- Expiration notices (24 hours for link, 15 minutes for OTP)
- Fallback link if button doesn't work
- Footer with company info

---

### 5.5 Rate Limiting & Anti-Fraud

#### Rate Limits

| Action | Limit | Window | Scope | Behavior When Exceeded |
|--------|-------|--------|-------|------------------------|
| Signup attempts | 5 | 1 hour | Per IP address | Show CAPTCHA on 6th attempt |
| Signup attempts | 10 | 1 hour | Per IP address | Block further attempts, show error message |
| Email verification resend | 3 | Per tenant | Per tenant | Disable resend button, suggest contacting support |
| OTP validation attempts | 3 | Per OTP | Per OTP | Require new verification email after 3 failures |
| Magic link clicks | 10 | 1 hour | Per token | Lock token, require new verification email |

#### Anti-Fraud Checks

1. **CAPTCHA (Google reCAPTCHA v3 or hCaptcha):**
   - Show if rate limit threshold exceeded
   - Show if suspicious activity detected (VPN, known bot IP)
   - Verify CAPTCHA response before processing signup

2. **Email Domain Blocklist:**
   - Block disposable email providers
   - Block known spam domains
   - Allow override by platform admin (for testing)

3. **IP Reputation Check:**
   - Check IP against known spam/bot databases (optional)
   - Flag suspicious signups for manual review

4. **Honeypot Field:**
   - Include hidden field in form (not visible to users)
   - If filled, reject signup silently (likely bot)

---

### 5.6 Plan Parameter Handling

**Query Parameter Format:**
```
/signup?plan=free
/signup?plan=professional
/signup?plan=enterprise
```

**Plan-Specific Defaults:**

| Plan | Storage Quota | User Limit | Features Enabled | Trial Duration |
|------|---------------|------------|------------------|----------------|
| free | 10 GB | Unlimited | Basic ITSM | N/A (Freemium) |
| professional | 50 GB | Unlimited | ITSM + ITOM | 14 days |
| enterprise | 100 GB | Unlimited | Full Suite | 14 days |

**Provisioning Logic:**
- If `plan` parameter present and valid → Apply plan-specific template during workspace provisioning (Step 5.3)
- If `plan` parameter absent or invalid → Default to "free" tier
- Store plan in tenant metadata for future reference and upgrade paths

---

## 6. State Machine: Tenant Provisioning States

### State Definitions

| State | Description | User-Visible? | Next States |
|-------|-------------|---------------|-------------|
| **Pending** | Signup submitted, email verification pending (magic link or OTP) | No | Provisioning, Failed |
| **Provisioning** | Email verified (magic link or OTP), workspace provisioning in progress, user authenticated | Yes (enhanced progress screen) | Active, Provisioning_Failed |
| **Active** | Workspace successfully provisioned and ready, user auto-logged-in | Yes (workspace dashboard) | Suspended, Deactivated |
| **Provisioning_Failed** | Provisioning failed, rollback in progress | Yes (error screen) | Failed, Provisioning (retry) |
| **Failed** | Provisioning failed permanently, rollback complete | Yes (error screen) | (Terminal state) |

---

### State Transition Diagram

```
┌──────────┐
│  START   │
└────┬─────┘
     │ User submits signup (email only)
     ▼
┌──────────┐
│ Pending  │◄──────────────────┐
└────┬─────┘                   │
     │ Email verified          │ Token/OTP expired
     │ (Magic Link or OTP)     │ User requests new
     ▼                         │
┌──────────────┐               │
│ Provisioning │───────────────┘
│ (User Auth)  │
└───┬──────┬───┘
    │      │ Provisioning fails
    │      ▼
    │ ┌─────────────────────┐
    │ │ Provisioning_Failed │
    │ └──┬──────────────┬───┘
    │    │ Rollback OK  │ Retry
    │    ▼              │
    │ ┌────────┐        │
    │ │ Failed │        │
    │ └────────┘        │
    │  (Terminal)       │
    │                   │
    │ Provisioning      │
    │ successful        │
    ▼                   ▼
┌────────┐      ┌──────────────┐
│ Active │      │ Provisioning │
│(User   │      │              │
│ In)    │      │              │
└────────┘      └──────────────┘
```

---

### State Transition Rules

| From State | To State | Trigger | Conditions | Actor |
|------------|----------|---------|------------|-------|
| Pending | Provisioning | Magic link clicked OR OTP verified | Token/OTP valid, not expired, user authenticated | System |
| Pending | Failed | Token/OTP expired + max resends | 24+ hours passed (magic link) or 15+ minutes (OTP), 3 resends exhausted | System |
| Provisioning | Active | Provisioning complete | All provisioning steps successful, user remains authenticated | System |
| Provisioning | Provisioning_Failed | Provisioning step fails | Any step times out or errors | System |
| Provisioning_Failed | Failed | Rollback complete | All resources cleaned up | System |
| Provisioning_Failed | Provisioning | Retry triggered | Auto-retry or manual retry | System or Admin |

---

## 7. UI/UX Functional Requirements

### 7.1 Signup Page (Passwordless)

**Page URL:** `https://motadata.com/signup` or `https://motadata.com/signup?plan=free`

**Layout (Functional Description):**
- Single-column centered form layout
- Company logo at top
- Page title: "Create Your Workspace"
- Subtitle: "Get started with Motadata in seconds—no password required"
- **Plan badge (if plan parameter present):** Display plan name above form (e.g., "Free Plan" or "Professional Plan - 14-day Trial")
- Simplified form fields (NO password field)
- Submit button: "Create Workspace" (primary CTA)
- Footer links: "Already have an account? Sign in"

**Form Fields:**
1. Organization Name (text input)
2. Email Address (email input)
3. Desired Subdomain (text input, optional)
4. Terms of Service checkbox
5. CAPTCHA (conditional)

**Responsive Behavior:**
- Mobile: Single column, full width fields
- Desktop: Centered card (max width 500px), standard field widths

**Accessibility:**
- All form fields have labels (not just placeholders)
- Error messages announced by screen readers
- Keyboard navigation supported (tab order logical)
- Focus indicators visible

**Loading States:**
- Submit button shows spinner during processing
- Submit button disabled to prevent double-submit
- Form fields remain visible (not hidden during submission)

---

### 7.2 Email Verification Confirmation Page with OTP Entry

**Page URL:** `https://motadata.com/verify/confirm`

**Layout (Functional Description):**
- Centered card with success icon (check mark)
- Title: "Check Your Email"
- Body text: "We've sent a verification email to [email]. You have two options to verify:"
- **Option 1 Instructions:** "**Click the verification link** in the email to instantly access your workspace"
- **Option 2 Instructions:** "**Or enter the 6-digit code** below"
- **OTP Entry Section (Prominent):**
  - Label: "Enter 6-digit code from email"
  - OTP input field (6 boxes or single field, numeric only)
  - "Verify Code" button (enabled when 6 digits entered)
  - Countdown timer: "Code expires in 14:32" (real-time countdown)
  - Error message area (below input, shown on validation failure)
- Instructions: "Please check your spam folder if you don't see the email within a few minutes."
- Secondary action: "Didn't receive the email?" → Expandable section with "Resend Verification Email" button
- Email client shortcuts (Gmail, Outlook icons)

**OTP Input Field Behavior:**
- Auto-focus on page load
- Numeric input only (reject non-numeric characters)
- Auto-submit when 6th digit entered (optional, or require button click)
- Visual feedback: Green border when valid, red border on error
- Clear button to reset input

**Resend Email Section:**
- Initially visible (not hidden like in old version)
- Button: "Resend Verification Email"
- Countdown timer if resend recently triggered: "Resend available in 00:45"
- Success message after resend: "Email sent! Please check your inbox."

**Accessibility:**
- Success icon has alt text
- Dynamic content (OTP timer, resend button state) announced to screen readers
- OTP input field ARIA labels for accessibility

---

### 7.3 Workspace Provisioning Progress Page (Enhanced with Engaging Content)

**Page URL:** `https://motadata.com/setup/progress` or auto-shown after email verification

**Layout (Functional Description):**
- Centered card with animated loading indicator (spinner or pulsing logo)
- Title: "Setting Up Your Workspace"
- Subtitle: "This usually takes 30-60 seconds. Please don't close this window."
- User info displayed (since user is authenticated): "Setting up for [email]"
- **Progress substeps (optional, if detailed status available):**
  1. ✓ Verifying your email (completed)
  2. ✓ Creating your database (completed)
  3. ⏳ Configuring your workspace (current step, animated)
  4. ⏸ Finalizing setup (pending)

**Engaging Content Section (NEW - Below progress indicator):**

**Content Carousel (Auto-rotates every 5 seconds):**
- Slide indicator dots (4 dots)
- Left/right navigation arrows (optional)

**Slide 1: Feature Highlight - ITSM**
- Icon: Service desk illustration
- Title: "Powerful IT Service Management"
- Description: "Manage incidents, requests, and changes with built-in ITSM workflows designed for modern IT teams"

**Slide 2: Quick Start Tip**
- Icon: Rocket illustration
- Title: "Get Started in Minutes"
- Description: "Once your workspace is ready, you can invite team members, customize settings, and integrate with your favorite tools"

**Slide 3: Integration Preview**
- Icon: Integration nodes illustration
- Title: "Connect Your Ecosystem"
- Description: "Seamlessly integrate with Slack, Microsoft Teams, Jira, and 50+ other tools to centralize your operations"

**Slide 4: Support Resource**
- Icon: Support headset illustration
- Title: "We're Here to Help"
- Description: "Access our comprehensive knowledge base, video tutorials, and 24/7 support whenever you need assistance"

**Did You Know? Tips (Below carousel, rotates with slides):**
- "💡 Tip: You can customize your workspace URL later in Settings"
- "💡 Tip: Free plan includes unlimited team members and basic ITSM"
- "💡 Tip: Your data is automatically backed up daily and encrypted"
- "💡 Tip: Invite your team with a single click after setup completes"

**Behavior:**
- Auto-refresh or WebSocket connection to check provisioning status
- Poll every 2 seconds for status updates
- Carousel auto-advances every 5 seconds (pause on hover)
- On success: Auto-redirect to workspace dashboard (or success page)
- On failure: Show error message (see section 7.4)

**Edge Cases:**
- User closes window: On return (via email link), check tenant state and show appropriate page
- Timeout (5+ minutes): Show warning, offer to "Check Status" or "Contact Support"

---

### 7.4 Provisioning Error Page

**Page URL:** `https://motadata.com/setup/error`

**Layout (Functional Description):**
- Centered card with error icon (red X or warning symbol)
- Title: "We Encountered an Issue"
- Body text: "We're sorry, but we couldn't complete your workspace setup due to a technical issue. Our team has been notified and is investigating."
- Reference ID: "Reference: [TENANT_ID or ERROR_ID]" (copyable)
- Primary CTA: "Try Again" button (retries provisioning)
- Secondary CTA: "Contact Support" link

**Behavior:**
- "Try Again" button: Re-initiates provisioning workflow, shows progress page
- "Contact Support" link: Opens support contact form with reference ID pre-filled
- User can copy reference ID for support ticket

---

### 7.5 Workspace Ready / Success Page (Optional - May Skip to Dashboard)

**Page URL:** `https://motadata.com/setup/complete`

**Layout (Functional Description):**
- Centered card with success icon (green check mark or celebration animation)
- Title: "Your Workspace is Ready!"
- Workspace details card:
  - Label: "Workspace URL"
  - Value: `https://[subdomain].motadata.com` (clickable link)
  - Copy button (copies URL to clipboard)
- Admin email confirmation: "Signed in as: [user's email]" (user already authenticated)
- Primary CTA: "Go to Dashboard" button (large, prominent) or auto-redirect after 3 seconds
- Secondary resources:
  - "Getting Started Guide" link
  - "Invite Team Members" link
  - "Contact Support" link

**Behavior:**
- "Go to Dashboard" button: Navigates to workspace dashboard (user already logged in)
- Auto-redirect after 3 seconds (countdown shown: "Redirecting in 3... 2... 1...")
- Copy button: Shows "Copied!" tooltip on click
- Links open in new tabs

**Accessibility:**
- Success icon has alt text
- Workspace URL announced by screen readers

---

### 7.6 Email Templates (Functional Content)

#### Email 1: Dual Verification Email (Magic Link + OTP)

**Subject:** Verify your email to activate your Motadata workspace

**From:** Motadata <noreply@motadata.com>

**Body (Plain Text):**
```
Hi [Name],

Thanks for signing up for Motadata!

You can verify your email in two ways:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OPTION 1: INSTANT ACCESS (RECOMMENDED)
Click this link to verify and access your workspace:
[Verify Email & Access Workspace]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OPTION 2: ENTER 6-DIGIT CODE
Your verification code: 123456
(Code expires in 15 minutes)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

This verification link will expire in 24 hours.

If you didn't sign up for Motadata, you can safely ignore this email.

Thanks,
The Motadata Team

---
Motadata Inc.
[Company Address]
```

**Body (HTML):**
- Branded header with logo
- Personalized greeting: "Hi [Name],"
- **Primary CTA (Magic Link):**
  - Large button: "Verify Email & Access Workspace"
  - Subtext: "Click to verify and get instant access"
- **Divider:** "OR"
- **Secondary Option (OTP):**
  - Large, bold 6-digit code in a box: `1 2 3 4 5 6`
  - Subtext: "Enter this code on the verification page"
  - Expiration notice: "Code expires in 15 minutes"
- **Expiration Notices:**
  - "Verification link expires in 24 hours"
  - "Verification code expires in 15 minutes"
- Fallback link (if button doesn't work): "Or copy and paste this URL: https://motadata.com/verify?token=xxxxx"
- Footer with company info
- Note: "If you didn't sign up, you can safely ignore this email"

---

#### Email 2: Welcome Email (Post-Provisioning)

**Subject:** Welcome to Motadata - Your Workspace is Ready!

**From:** Motadata <welcome@motadata.com>

**Body (Plain Text):**
```
Hi [Name],

Welcome to Motadata! Your workspace is ready and you're all set.

Workspace URL: https://[subdomain].motadata.com
You're signed in as: [email]

Here's what to do next:
1. Explore your workspace dashboard
2. Customize your profile and settings
3. Invite your team members
4. Connect your tools and integrations

FUTURE LOGINS (PASSWORDLESS):
To access your workspace in the future, simply visit:
https://[subdomain].motadata.com
Enter your email, and we'll send you a magic link to sign in instantly.

Need help? Check out our Getting Started Guide or contact support.

Let's get started!
The Motadata Team

---
[Company Address]
[Unsubscribe Link]
```

**Body (HTML):**
- Branded header with logo
- Personalized greeting
- Workspace details card:
  - Workspace URL (large, clickable)
  - Email used (confirmation)
  - Plan tier (if applicable): "Free Plan" or "Professional Trial"
- Next steps checklist (numbered list with icons)
- **Future Login Instructions (Highlighted Box):**
  - Title: "How to Sign In Next Time"
  - Instructions: "We use passwordless authentication for your security and convenience. Simply enter your email and we'll send you a magic link."
  - CTA button: "Access Your Workspace"
- Resource links (Getting Started, Support, Documentation)
- Footer with company info and unsubscribe

---

#### Email 3: Provisioning Failure Notification

**Subject:** Action Required: Workspace Setup Issue

**From:** Motadata Support <support@motadata.com>

**Body (Plain Text):**
```
Hi [Name],

We're sorry, but we encountered an issue while setting up your Motadata workspace.

Reference ID: [TENANT_ID]

What happened?
We experienced a technical issue during the setup process. Our team has been notified and is investigating.

What to do next?
1. Try signing up again: https://motadata.com/signup
2. Contact our support team: support@motadata.com (include Reference ID above)

We apologize for the inconvenience and appreciate your patience.

The Motadata Team

---
[Company Address]
```

**Body (HTML):**
- Branded header with logo
- Apology and explanation
- Reference ID (bold, large font, easy to copy)
- Clear next steps with CTA buttons
- Support contact information
- Footer

---

## 8. Error Handling & Edge Cases

### 8.1 Error Categories

| Error Category | User Impact | System Response | User Feedback |
|----------------|-------------|-----------------|---------------|
| **Validation Error** | Cannot submit form | Display inline error message, keep form data | "Please correct the highlighted fields" |
| **Duplicate Email** | Cannot create account | Display error, suggest sign in | "Email already registered. [Sign in]" |
| **Invalid OTP** | Cannot verify email | Display inline error, allow retry (3 attempts max) | "Invalid code. Please try again. [X attempts remaining]" |
| **Expired OTP** | Cannot verify email | Display error with resend option | "Code expired. [Request new code]" |
| **Expired Magic Link** | Cannot verify email | Display error with resend option | "Link expired. [Request new link]" |
| **Network Error** | Temporary failure | Retry automatically (3 attempts), then show error | "Connection issue. Please try again." |
| **Provisioning Failure** | Workspace not created | Automatic rollback, send notification | "Setup failed. We've been notified. [Try Again]" |
| **Rate Limit Exceeded** | Temporarily blocked | Show CAPTCHA or block with cooldown | "Too many attempts. Please try again in [X] minutes." |

---

### 8.2 Edge Cases & Handling

#### Edge Case 1: User Uses Both Magic Link and OTP

**Scenario:** User clicks magic link in email, then also tries to enter OTP on verification page

**Handling:**
- First successful verification method (magic link or OTP) marks email as verified and starts provisioning
- If user already verified via magic link, OTP entry page shows: "Email already verified! Redirecting to your workspace..."
- If user already verified via OTP, magic link click shows: "Email already verified! Redirecting to your workspace..."
- Both methods transition tenant to same "Provisioning" state

**User Impact:** Seamless (no error, user redirected to progress screen)

---

#### Edge Case 2: User Closes Browser During OTP Entry

**Scenario:** User enters partial OTP digits, then closes browser before submitting

**Handling:**
- OTP remains valid for 15 minutes (expiration countdown continues server-side)
- User can return to verification page later (browser bookmark or re-click email magic link)
- If OTP still valid: User can continue entering remaining digits
- If OTP expired: User sees "Code expired. [Request new code]" and can resend email

**User Impact:** Minimal (can resume or request new code)

---

#### Edge Case 3: Email Delivery Failure

**Scenario:** Email service is down or email bounces

**Handling:**
- Verification Email:
  - Queue email for retry (up to 3 retries with exponential backoff: 2s, 4s, 8s)
  - If all retries fail: Log error, alert platform admin
  - User can request resend (generates new magic link + OTP, retries delivery)

- Welcome Email:
  - Queue for retry (up to 5 retries over 24 hours)
  - Non-critical: User already in workspace (auto-logged-in), can access without email
  - Log delivery failure for platform admin review

**User Impact:** Minimal (user can request resend, or use welcome email as reference later)

---

#### Edge Case 4: User Verifies Email Multiple Times (Magic Link)

**Scenario:** User clicks magic link multiple times (e.g., from different devices or browsers)

**Handling:**
- First click: Initiate provisioning, create authenticated session
- Subsequent clicks: Check tenant state
  - If provisioning: Show "Already in progress" message with link to progress page
  - If active: Show "Already activated" message with link to workspace
  - If failed: Show error page with retry option
- Magic link token marked as "used" after first successful click (single-use security)

**UI Message:**
- "Your workspace is already being set up! [Check Progress]"
- "Your workspace is already active! [Access Workspace]"

---

#### Edge Case 5: OTP Max Attempts Exceeded

**Scenario:** User enters wrong OTP 3 times consecutively

**Handling:**
1. After 3rd failed attempt, lock OTP for that tenant
2. Display error: "Maximum attempts exceeded. [Request new verification email]"
3. Require user to request new email (generates new magic link + new OTP)
4. Log security event (potential brute force attempt)

**Security Benefit:** Prevents OTP brute force attacks (1,000,000 combinations / 3 attempts = low success rate)

---

#### Edge Case 6: Provisioning Hangs (Exceeds 5-Minute Timeout)

**Scenario:** Provisioning process stalls (e.g., database creation hangs)

**Handling:**
- System timeout: 5 minutes for entire provisioning workflow
- On timeout:
  - Trigger rollback workflow
  - Log timeout event (critical alert to platform admin)
  - Mark tenant as "Provisioning_Failed"
  - Show error page to user with retry option
  - Auto-retry once after 10 minutes (in case of transient issue)

**Alert to Platform Admin:** Immediate notification (P1 alert)

---

#### Edge Case 7: Subdomain Becomes Unavailable Between Validation and Provisioning

**Scenario:** Subdomain is validated as available during signup, but taken by another signup before provisioning (race condition)

**Handling:**
- During provisioning, re-check subdomain availability
- If taken: Append random suffix (e.g., `-2`, `-a8f3`)
- Update tenant record with final subdomain
- Show final subdomain on success page and welcome email
- Log event for monitoring (indicates race condition, may need locking mechanism)

**User Impact:** Minimal (subdomain slightly different than expected, but user is notified)

---

#### Edge Case 8: User Tries to Sign Up Again While Email Verification Pending

**Scenario:** User submits signup form, receives "Check Your Email" page, then tries to sign up again with same email

**Handling:**
- Check if email has pending verification (tenant in "Pending" state)
- If pending: Display message "You already have a pending signup. Please check your email or [Resend verification email]"
- Do not create duplicate tenant record
- Optionally: Allow resend from signup page directly

**User Impact:** Prevented duplicate signups, guided to complete existing signup

---

## 9. Integration Touchpoints (Functional)

### 9.1 Email Service Integration

**Purpose:** Send transactional emails (dual verification with magic link + OTP, welcome, failure notifications)

**Functional Requirements:**
- Send email to specified recipient
- Support HTML and plain text formats
- Track delivery status (sent, delivered, bounced, failed)
- Support email templates with variable substitution (e.g., `{{name}}`, `{{magic_link}}`, `{{otp_code}}`, `{{workspace_url}}`)
- Queue emails for asynchronous delivery (non-blocking)

**Failure Handling:**
- Retry failed sends (up to 3 times with exponential backoff: 2s, 4s, 8s)
- Alert platform admin if email service unavailable
- Fallback to secondary email provider if primary fails (see FRD 2.3 for multi-provider redundancy)

**SLA:** Email sent within 60 seconds (verification) or 2 minutes (welcome)

---

### 9.2 Authentication Session Management

**Purpose:** Create authenticated session for user after magic link or OTP verification (passwordless auto-login)

**Functional Requirements:**
- Generate secure session token (JWT or session ID) after successful verification
- Store session with expiration (e.g., 30 days for "remember me", 4 hours for standard session)
- Set httpOnly, secure cookies for session token
- Support session refresh (if using JWT with refresh tokens)
- Validate session on workspace access

**Failure Handling:**
- If session creation fails: Log error, show generic error message, allow user to retry verification
- Session expiration: Redirect to login page (future: send magic link for re-authentication per FRD 1.2)

**SLA:** Session created within 200ms of verification

---

### 9.3 DNS Management Integration

**Purpose:** Register subdomain for tenant workspace

**Functional Requirements:**
- Check subdomain availability (query DNS records)
- Register new subdomain (create DNS A/CNAME record)
- Point subdomain to appropriate load balancer or application server
- Support wildcard SSL certificate for `*.motadata.com`

**Failure Handling:**
- If DNS registration fails: Rollback provisioning
- Retry DNS registration (up to 2 times)
- Log DNS errors for platform admin review

**SLA:** DNS record propagated within 60 seconds (for internal DNS)

---

### 9.4 Database Provisioning Integration

**Purpose:** Create dedicated database or schema for tenant

**Functional Requirements:**
- Create new database or schema with tenant-specific name (e.g., `tenant_[tenant_id]`)
- Apply database migrations (create tables, indexes, constraints)
- Apply plan-specific schema templates (if plan parameter present):
  - Free plan: Basic ITSM tables
  - Professional plan: ITSM + ITOM tables
  - Enterprise plan: Full suite tables
- Create database user with appropriate permissions (tenant-scoped)
- Validate database connectivity and schema integrity

**Failure Handling:**
- If database creation fails: Rollback (delete database/schema)
- Retry database creation (up to 2 times)
- Log database errors for platform admin review

**SLA:** Database ready within 30 seconds

---

### 9.5 Monitoring & Alerting Integration

**Purpose:** Track provisioning events, detect failures, alert admins

**Functional Requirements:**
- Log all provisioning events (start, steps, completion, failure)
- Track provisioning duration and success rate
- Track verification method usage (magic link vs OTP percentage)
- Send alerts when provisioning failure rate exceeds threshold (>5% over 15 minutes)
- Provide dashboard for platform admin to monitor provisioning health

**Alert Triggers:**
- Provisioning failure rate >5% (over 15-minute window)
- Single provisioning timeout (>5 minutes)
- Email delivery failure rate >5%
- Database provisioning failure
- OTP brute force attempt (3 failures in <1 minute)

**Alert Recipients:** Platform admin, on-call engineer

---

## 10. Acceptance Criteria

### 10.1 Functional Acceptance Criteria

**Must-Have (Required for Release):**

1. **Passwordless Signup & Verification:**
   - ✅ User can submit signup form with email only (no password field)
   - ✅ System validates all form fields (see section 5.1)
   - ✅ System sends dual verification email (magic link + OTP) within 60 seconds
   - ✅ User can verify email via magic link (instant auto-login)
   - ✅ User can verify email via OTP (6-digit code entry, instant auto-login)
   - ✅ User can resend verification email (max 3 times, generates new magic link + OTP)
   - ✅ Expired magic link (>24 hours) shows error with option to request new
   - ✅ Expired OTP (>15 minutes) shows error with option to request new
   - ✅ Invalid OTP (wrong code) shows error with retry option (max 3 attempts)

2. **Instant Auto-Login After Verification:**
   - ✅ Magic link click creates authenticated session immediately
   - ✅ OTP verification creates authenticated session immediately
   - ✅ User redirected to provisioning progress screen while already logged in
   - ✅ User auto-redirected to workspace dashboard after provisioning completes (no manual login required)

3. **Subdomain:**
   - ✅ User can specify desired subdomain
   - ✅ System validates subdomain uniqueness and format
   - ✅ System auto-generates subdomain if blank
   - ✅ Reserved subdomains are blocked

4. **Provisioning:**
   - ✅ Email verification triggers automatic provisioning
   - ✅ User sees enhanced progress screen with engaging content during provisioning
   - ✅ Provisioning completes within 60 seconds (p95)
   - ✅ Provisioned workspace is accessible via subdomain URL
   - ✅ User already logged in (authenticated session from verification)

5. **Lightweight Pricing Page Integration:**
   - ✅ System accepts `plan` query parameter (e.g., `/signup?plan=free`)
   - ✅ System displays plan badge if parameter present
   - ✅ System applies plan-specific defaults during provisioning (storage quota, features, templates)
   - ✅ System defaults to freemium tier if no plan parameter

6. **Enhanced Progress Screen:**
   - ✅ Progress screen displays engaging content (feature highlights, tips, integration preview)
   - ✅ Content carousel auto-rotates every 5 seconds (4 slides)
   - ✅ Tips section displays helpful information during provisioning
   - ✅ User sees authenticated state (email displayed) during provisioning

7. **Welcome Notification:**
   - ✅ User receives welcome email within 2 minutes of provisioning completion
   - ✅ Welcome email contains workspace URL and passwordless login instructions
   - ✅ User can access workspace (already logged in from verification step)

8. **Error Handling & Rollback:**
   - ✅ Provisioning failure triggers automatic rollback
   - ✅ Rollback completes within 5 minutes
   - ✅ No orphaned resources after rollback (verified by test suite)
   - ✅ User receives failure notification email
   - ✅ User can retry signup after failure

9. **Rate Limiting:**
   - ✅ System enforces rate limits (5 signups per IP per hour)
   - ✅ CAPTCHA shown after rate limit exceeded
   - ✅ Hard block after 10 signups per IP per hour
   - ✅ OTP validation limited to 3 attempts per code

10. **Audit Logging:**
    - ✅ All provisioning events logged (signup, verification method used, provisioning, completion, failure)
    - ✅ Audit logs include timestamp, actor, tenant ID, action, verification method (magic_link or otp), result

---

### 10.2 Non-Functional Acceptance Criteria

1. **Performance:**
   - ✅ Signup form submission response <2 seconds
   - ✅ Dual verification email (magic link + OTP) sent within 60 seconds (p95)
   - ✅ Magic link click to authenticated session creation <1 second
   - ✅ OTP validation and session creation <500ms
   - ✅ Provisioning completes within 60 seconds (p95)
   - ✅ Workspace accessible within 5 seconds of provisioning completion

2. **Reliability:**
   - ✅ Provisioning success rate >98% (excluding known infrastructure issues)
   - ✅ Email delivery rate >99% (verification and welcome emails)
   - ✅ Rollback success rate 100% (all failed provisionings cleaned up)
   - ✅ Magic link token single-use enforcement 100%
   - ✅ OTP single-use enforcement 100%

3. **Security:**
   - ✅ Magic link tokens hashed in database (SHA-256 or stronger)
   - ✅ OTP codes hashed in database (bcrypt or SHA-256)
   - ✅ All validation rules enforced on backend (not just frontend)
   - ✅ Rate limiting prevents brute force (3 OTP attempts max, 10 magic link clicks max per hour)
   - ✅ Session tokens httpOnly, secure cookies
   - ✅ No cross-tenant data leakage (validated by test suite)
   - ✅ HTTPS enforced for all verification URLs

4. **Accessibility:**
   - ✅ All forms keyboard-navigable
   - ✅ All form fields have labels (not just placeholders)
   - ✅ OTP input field accessible (ARIA labels, screen reader support)
   - ✅ Error messages announced by screen readers
   - ✅ Countdown timers accessible (ARIA live regions)
   - ✅ Color contrast meets WCAG AA standards

5. **Usability:**
   - ✅ Inline error messages shown immediately (real-time validation)
   - ✅ Form data retained on validation errors (no need to re-enter)
   - ✅ Clear success and error messages (no jargon)
   - ✅ Enhanced progress screen with engaging content (reduces perceived wait time)
   - ✅ Dual verification options (magic link + OTP) accommodate user preferences
   - ✅ Countdown timers for OTP expiration (transparent, no surprise expirations)

---

### 10.3 Test Cases (Sample)

**Test Case 1: Happy Path - Passwordless Signup via Magic Link**
1. User navigates to signup page (`/signup?plan=professional`)
2. User sees "Professional Plan - 14-day Trial" badge
3. User enters organization name and email (no password field visible)
4. User checks Terms of Service checkbox
5. User clicks "Create Workspace"
6. System displays "Check Your Email" screen with OTP entry option
7. User opens email and clicks magic link ("Verify Email & Access Workspace" button)
8. System creates authenticated session automatically (user logged in)
9. System displays "Setting Up Workspace" enhanced progress screen with feature carousel
10. After 30-45 seconds, provisioning completes
11. User auto-redirected to workspace dashboard (already logged in)
12. User sees workspace with professional plan features enabled

**Expected:** All steps complete successfully, passwordless auto-login works, plan applied correctly

---

**Test Case 2: Happy Path - Passwordless Signup via OTP**
1. User navigates to signup page (`/signup`)
2. User enters organization name and email
3. User clicks "Create Workspace"
4. System displays "Check Your Email" screen with OTP entry field
5. User opens email, copies 6-digit OTP code (e.g., "123456")
6. User enters OTP in input field
7. System validates OTP and creates authenticated session
8. System displays provisioning progress screen (user logged in)
9. After 30-45 seconds, provisioning completes
10. User auto-redirected to workspace dashboard (already logged in)

**Expected:** OTP verification works, instant auto-login, workspace accessible

---

**Test Case 3: OTP Validation Failure and Retry**
1. User on "Check Your Email" screen with OTP entry
2. User enters wrong OTP code (e.g., "000000")
3. System displays error: "Invalid code. Please try again. 2 attempts remaining"
4. User enters wrong OTP again
5. System displays error: "Invalid code. Please try again. 1 attempt remaining"
6. User enters correct OTP (from email)
7. System validates successfully and creates authenticated session
8. User proceeds to provisioning

**Expected:** OTP attempt limit enforced, user can retry, successful on 3rd attempt

---

**Test Case 4: OTP Expiration**
1. User signs up, receives verification email
2. User waits 16 minutes (OTP expires after 15 minutes)
3. User enters valid OTP code from email
4. System displays error: "Code expired. [Request new code]"
5. User clicks "Request new code" (resend email)
6. System sends new email with new OTP
7. User enters new OTP within 15 minutes
8. System validates successfully

**Expected:** Expired OTP rejected, user can request new code, new OTP works

---

**Test Case 5: Magic Link and OTP Both Used (Edge Case)**
1. User signs up, receives verification email with magic link and OTP
2. User clicks magic link in email first
3. System creates authenticated session, starts provisioning
4. User (in another browser tab) tries to enter OTP on verification page
5. System detects email already verified
6. System displays: "Email already verified! Redirecting to your workspace..."
7. User redirected to progress screen

**Expected:** Both verification methods handled gracefully, no errors, single provisioning initiated

---

**Test Case 6: Enhanced Progress Screen Content**
1. User verifies email (magic link or OTP)
2. Provisioning starts, user sees progress screen
3. User sees animated loading indicator
4. User sees feature highlight carousel (4 slides)
5. Carousel auto-advances every 5 seconds
6. User sees "Did You Know?" tip below carousel
7. After provisioning completes, user auto-redirected to workspace

**Expected:** Engaging content displayed, carousel works, perceived wait time reduced

---

**Test Case 7: Lightweight Pricing Page Integration**
1. User navigates to `/signup?plan=enterprise`
2. System displays "Enterprise Plan - 14-day Trial" badge on signup page
3. User completes signup and verification
4. System provisions workspace with enterprise-tier defaults (100GB storage, full suite features)
5. User lands in workspace with enterprise plan enabled

**Expected:** Plan parameter captured, plan-specific defaults applied during provisioning

---

**Test Case 8: Provisioning Failure & Rollback**
1. User signs up and verifies email
2. System starts provisioning
3. Database creation fails (simulated error)
4. System triggers rollback
5. System cleans up all resources
6. User sees error screen with "Try Again" button
7. User receives failure notification email

**Expected:** Rollback completes, no orphaned resources, user notified

---

**Test Case 9: Rate Limit Exceeded**
1. User attempts 6 signups from same IP within 1 hour
2. On 6th attempt, system shows CAPTCHA
3. User completes CAPTCHA
4. System allows signup to proceed
5. User attempts 5 more signups (total 11)
6. On 11th attempt, system blocks with error message

**Expected:** CAPTCHA shown after 5 attempts, hard block after 10

---

**Test Case 10: Subdomain Already Taken**
1. User enters subdomain "acme" (already exists)
2. User submits form
3. System displays error: "Subdomain already taken"
4. System suggests alternatives: "acme-1", "acme-2"
5. User changes to "acme-corp"
6. User submits form
7. System proceeds successfully

**Expected:** Inline error shown, alternatives suggested, user can retry

---

## 11. Out of Scope for FRD 1.1

The following features are explicitly excluded from this FRD and will be covered in future FRDs:

1. **Password-Based Authentication** - Fully passwordless in FRD 1.1; password support may be added later as optional per FRD 1.2
2. **Social Login (Google, Microsoft, etc.)** - FRD 1.2 or Phase 2
3. **Custom Domain Support** (e.g., tenant.customdomain.com) - Phase 2
4. **Multi-Region Deployment Selection** - Phase 2
5. **Full Pricing Page UI** - FRD 2.1 (lightweight integration via query parameter only in FRD 1.1)
6. **Payment Integration** (credit card entry during signup) - FRD 2.1
7. **Assisted/Manual Provisioning Workflow** (for enterprise sales) - FRD 1.3
8. **Admin Approval Gate** (for enterprise signups) - FRD 1.3
9. **Guided Onboarding Wizard** (post-provisioning setup steps) - FRD 1.5 or Phase 2
10. **Module Template Selection During Signup** - Phase 2 (default template applied automatically in MVP)
11. **Tenant Customization During Signup** (logo, colors, etc.) - Phase 2
12. **Advanced Fraud Detection** (ML-based, risk scoring) - Future
13. **SMS-Based OTP** (email OTP only in MVP) - Phase 2 enhancement

---

## 12. Dependencies

### 12.1 Technical Dependencies (For Architect/Developer)

**Note:** This section lists functional dependencies. Technical implementation details (specific libraries, frameworks, services) will be defined by software architect.

1. **Email Service:**
   - Functional capability to send transactional emails with dual content (magic link + OTP)
   - Support for HTML and plain text
   - Delivery tracking
   - Required by: Dual verification email, welcome email, failure notifications

2. **Passwordless Authentication System:**
   - Magic link token generation and validation (UUID v4, secure hashing)
   - OTP code generation and validation (6-digit, cryptographically secure)
   - Session management (JWT or secure session cookies, httpOnly, secure flags)
   - Token/OTP single-use enforcement
   - Required by: Passwordless signup, instant auto-login

3. **DNS Management:**
   - Ability to register subdomains programmatically
   - DNS record creation/validation
   - SSL certificate support (wildcard for `*.motadata.com`)
   - Required by: Subdomain provisioning

4. **Database System:**
   - Multi-tenant database architecture (database-per-tenant or schema-per-tenant)
   - Database provisioning automation with plan-specific templates
   - Migration management
   - Required by: Workspace database creation

5. **Background Job Processing:**
   - Asynchronous task execution (for provisioning workflow)
   - Job retry and failure handling
   - Job status tracking (for real-time progress updates)
   - Required by: Provisioning automation, email delivery

6. **Monitoring & Logging:**
   - Event logging and audit trail (including verification method used: magic_link vs otp)
   - Metrics collection (provisioning duration, success rate, verification method usage percentage)
   - Alerting (failure rate thresholds, OTP brute force detection)
   - Required by: Operational visibility, debugging, security monitoring

---

### 12.2 External Dependencies (Must be Resolved Before Development)

| Dependency | Status | Owner | Blocker? | Notes |
|------------|--------|-------|----------|-------|
| Email Service (SendGrid/SES) | OPEN | Infrastructure Team | Yes | Domain verification, DNS records, dual content template support (magic link + OTP) |
| Passwordless Auth Library/Service | OPEN | Backend Team | Yes | Magic link + OTP generation, validation, session management |
| DNS Management API | OPEN | Cloud Operations | Yes | Access to DNS provider API or internal DNS service |
| Database Infrastructure | OPEN | Cloud Operations | Yes | Database cluster provisioned, plan-specific template provisioning scripts ready |
| SSL Certificate (Wildcard) | OPEN | DevOps | Yes | `*.motadata.com` certificate required |
| Monitoring Infrastructure | OPEN | DevOps | No | Can use basic logging initially, full monitoring later |

---

## 13. Success Metrics (Post-Launch)

### 13.1 Primary Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| **Provisioning Success Rate** | >98% | (Successful provisionings / Total attempts) × 100 |
| **Provisioning Duration (p95)** | <60 seconds | Time from email verification to Active state |
| **Email Verification Rate** | >85% | (Verified emails / Total signups) × 100 (expected higher with passwordless) |
| **Signup Completion Rate** | >75% | (Completed signups / Form submissions) × 100 (expected higher without password field) |
| **Rollback Success Rate** | 100% | (Clean rollbacks / Failed provisionings) × 100 |
| **Magic Link vs OTP Usage** | Magic Link >70% | (Magic link verifications / Total verifications) × 100 (track user preference) |

---

### 13.2 Secondary Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Email Delivery Rate (Verification) | >99% | (Delivered / Sent) × 100 |
| Email Delivery Rate (Welcome) | >99% | (Delivered / Sent) × 100 |
| OTP Validation Success Rate (First Attempt) | >90% | (Successful first attempts / Total OTP validations) × 100 |
| Magic Link Click-Through Rate | >80% | (Magic link clicks / Verification emails sent) × 100 |
| Subdomain Auto-Generation Rate | >50% | (Auto-generated / Total signups) × 100 |
| Average Provisioning Duration | <45 seconds | Median time from verification to Active |
| Signup Form Validation Error Rate | <15% | (Form submissions with errors / Total submissions) × 100 (expected lower with fewer fields) |
| Enhanced Progress Screen Engagement | N/A (baseline) | Carousel slide views, time spent on progress screen (measure perceived wait improvement) |

---

### 13.3 Monitoring & Alerts

**Real-Time Monitoring:**
- Provisioning success rate (15-minute rolling window)
- Provisioning duration distribution (p50, p95, p99)
- Email delivery rate
- Signup form submission rate
- Verification rate (magic link vs OTP breakdown)
- OTP validation attempts and failure rate
- Magic link usage vs OTP usage (track user preferences)

**Alerts:**
- Provisioning failure rate >5% (15-minute window) → P1 alert
- Single provisioning timeout (>5 minutes) → P1 alert
- Email delivery failure rate >5% → P2 alert
- Database provisioning failure → P1 alert
- OTP brute force detected (3 failures in <1 minute per tenant) → P2 alert (security monitoring)
- Magic link token reuse attempt detected → P2 alert (security monitoring)

---

## 14. Glossary

| Term | Definition |
|------|------------|
| **Tenant** | A workspace instance with dedicated data and subdomain (e.g., customer organization) |
| **Subdomain** | Unique URL prefix for tenant workspace (e.g., `acme.motadata.com`) |
| **Provisioning** | Automated process of creating workspace infrastructure (database, resources, configuration) |
| **Rollback** | Process of cleaning up resources after provisioning failure |
| **Magic Link** | Secure, time-limited URL token sent via email to verify user's email and auto-login (passwordless authentication) |
| **OTP (One-Time Password)** | 6-digit numeric code sent via email to verify user's email (alternative to magic link) |
| **Passwordless Authentication** | Authentication method without passwords, using magic links or OTP codes instead |
| **Dual Verification** | Offering both magic link AND OTP in the same email, giving users choice of verification method |
| **Auto-Login** | Automatic session creation and authentication after email verification (no manual login required) |
| **State Machine** | Logical representation of tenant provisioning states and transitions |
| **Workspace** | The tenant's dedicated application environment (UI, database, configurations) |
| **Admin User** | The first user who signed up for the tenant (full administrative privileges, passwordless access) |
| **Rate Limiting** | Restriction on number of signups or validation attempts per IP/tenant to prevent abuse |
| **CAPTCHA** | Challenge-response test to distinguish humans from bots |
| **Plan Parameter** | Query parameter (`?plan=free/professional/enterprise`) used for lightweight pricing page integration |
| **Enhanced Progress Screen** | Provisioning progress page with engaging content (feature highlights, tips) to improve perceived wait time |

---

## 15. Document Review & Sign-Off

**Review Required By:**
- Product Lead (functional requirements, user flows, passwordless auth UX)
- UX Designer (UI/UX functional requirements, wireframes, OTP entry UX, enhanced progress screen)
- Software Architect (integration touchpoints, technical feasibility, passwordless auth implementation)
- Security Representative (validation rules, passwordless security, anti-fraud checks, OTP security)
- QA Lead (acceptance criteria, test cases, passwordless auth testing)

**Sign-Off:**

| Role | Name | Date | Status |
|------|------|------|--------|
| Product Lead | Rakesh Rathod | — | Pending |
| UX Designer | [Name TBD] | — | Pending |
| Software Architect | Amit Patel | — | Pending |
| Security Representative | Pratik Patel | — | Pending |
| QA Lead | [Name TBD] | — | Pending |

---

## 16. Change Log

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2025-10-24 | Rakesh Rathod | Initial FRD for Basic Tenant Provisioning |

---

## 18. Related Documents

- **PRD:** PRD_TENANT_ONBOARDING v1.0
- **Release Plan:** RELEASE_PLAN_2_MONTH.md
- **Technical Specifications:** (To be created by Software Architect)
  - API Contract: TBD (must include magic link + OTP generation endpoints, session management)
  - Data Model: TBD (must include magic link token, OTP code storage, plan metadata)
  - Event Schema: TBD (must include verification_method field: magic_link | otp)
- **UX Designs:** (To be created by UX Designer)
  - Wireframes: TBD (passwordless signup form, OTP entry screen, enhanced progress screen)
  - Figma Mockups: TBD
  - User Flow Diagram: TBD (passwordless verification flow with dual options)

---

**Document Status:** ✅ Ready for Review

**Next Steps:**
1. Product Lead review and approval (passwordless approach, plan integration, enhanced progress UX)
2. UX Designer creates wireframes for:
   - Simplified passwordless signup form (no password field)
   - OTP entry screen with countdown timer
   - Enhanced progress screen with content carousel
3. Software Architect defines:
   - Magic link token generation and validation (UUID v4, SHA-256 hashing)
   - OTP code generation and validation (6-digit, bcrypt hashing, rate limiting)
   - Session management for instant auto-login (JWT or secure session cookies)
   - Plan parameter handling and template provisioning logic
4. Security review of passwordless authentication approach (token security, OTP brute force prevention)
5. Development begins after all sign-offs obtained

---
