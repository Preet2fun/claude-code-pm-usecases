---
dfs_id: CH_002_DFS_v1
title: User Lifecycle Management (Invite to Deactivate)
parent_prd: PRD_USER_ACCESS
parent_chapter: CH_002
version: 1.0
status: draft
generated_by: dfs-generator-agent
generated_at: 2025-10-07T00:00:00Z
platform_context: applied
template_version: 4.0-minimal
---

# DFS: User Lifecycle Management (Invite to Deactivate)

## 1. OVERVIEW

### 1.1 What & Why

**Purpose:** Enable tenant administrators to manage the complete user lifecycle from invitation through activation, suspension, and deactivation with automated workflows. Users transition through distinct states with clear business rules, audit trails, and compliance controls to support enterprise onboarding, offboarding, and security requirements.

**Scope:**
- Individual and bulk user invitation workflows (CSV import)
- User activation flow with profile setup and passwordless authentication enrollment
- User suspension with session revocation and role retention
- User deactivation with data retention and right-to-erasure compliance
- Automated lifecycle state transitions with notifications
- Tenant-scoped user limits and resource ownership validation

**Business Value:**
- **Rapid Onboarding:** Admins can bulk invite 100 users via CSV and complete invitations within 5 minutes, reducing manual provisioning overhead.
- **Security & Compliance:** Suspended users cannot access systems but retain audit trails; deactivated users have access revoked with GDPR-compliant data deletion after 90 days.
- **Operational Efficiency:** Activation success rate >95% with <1 hour invitation-to-activation time (95th percentile), enabling seamless enterprise user onboarding.

**Success Metrics:**
- Invitation-to-activation time <1 hour (95th percentile)
- User activation success rate >95%
- Lifecycle state change latency <5 seconds

### 1.2 PRD Traceability

| PRD Req ID | Requirement Summary | DFS Coverage | Status |
|------------|---------------------|--------------|--------|
| FR-UCM-001 | Individual user invitation with 7-day activation link | Section 4.1 | ✅ |
| FR-UCM-002 | Bulk user invitation via CSV import | Section 4.2 | ✅ |
| FR-UCM-003 | User activation with profile setup | Section 4.3 | ✅ |
| FR-UCM-004 | Passwordless authentication enrollment during activation | Section 4.3 | ✅ |
| FR-UCM-005 | User suspension with session revocation | Section 4.4 | ✅ |
| FR-UCM-006 | User deactivation with resource ownership check | Section 4.5 | ✅ |
| FR-UCM-007 | 90-day data retention after deactivation | Section 10 | ✅ |
| FR-UCM-008 | Automated lifecycle state logging | Section 12 | ✅ |
| FR-UCM-009 | Tenant user limit enforcement | Section 5 | ✅ |
| FR-UCM-010 | Invitation link expiry validation | Section 5 | ✅ |
| SEC-UCM-001 | Active users must have valid authentication method | Section 6 | ✅ |
| COMP-UCM-001 | GDPR right-to-erasure within 30 days | Section 11 | ✅ |
| COMP-UCM-002 | Immutable audit logging of lifecycle transitions | Section 11 | ✅ |
| NFR-UCM-001 | Activation success rate >95%, latency <1 hour | Section 11 | ✅ |

**Coverage:** 14 of 14 requirements complete, 0 deferred

---

## 2. USERS & CONTEXT

| User Type | What They Do | What They Need | Pain Points Solved |
|-----------|--------------|----------------|-------------------|
| **Tenant Admin** | Invites, suspends, deactivates users; manages bulk onboarding | Ability to invite 100s of users at once, clear user state visibility, error handling for resource conflicts | Manual, error-prone CSV processing; inability to track invitation status; blocked deactivations due to unknown resource ownership |
| **System Administrator** | Provisions tenants, manages global user limits, monitors compliance | Tenant-level user limits, GDPR compliance enforcement, audit trail visibility | Lack of centralized lifecycle visibility across tenants; manual enforcement of compliance policies |
| **End User (Invitee)** | Receives invitation email, completes profile setup, enrolls authentication method | Clear, simple activation flow; guidance on authentication setup | Confusing activation flows; expired links without resend option; unclear error messages |
| **Manager/Security Officer** | Reviews suspended users, approves reactivations, audits deactivations | Dashboard of suspended/deactivated users, justification fields, compliance reports | No visibility into why users were suspended; inability to audit lifecycle decisions |

---

## 3. STATE MODEL

### States

| State | User Sees/Experiences | Can User Act? | Transitions To |
|-------|----------------------|---------------|----------------|
| **Invited** | User receives email with activation link (valid 7 days); cannot login until activation complete | No (pending activation) | Active, Invitation Expired |
| **Invitation Expired** | Activation link shows "Link expired" error; user sees message to contact admin for resend | No | Invited (if admin resends) |
| **Active** | User has full access per assigned roles; can login, perform actions, access resources | Yes | Suspended, Deactivated |
| **Suspended** | Login blocked with message "Account suspended. Contact your administrator."; active sessions immediately terminated | No | Active (if admin reactivates), Deactivated |
| **Deactivated** | Login blocked with message "Account deactivated."; all access revoked; data scheduled for deletion after 90 days | No | Active (if reactivated within 90-day window) |
| **Deleted** | User record permanently removed after 90-day retention period or GDPR erasure request | No | N/A (terminal state) |

### Transitions

| From → To | Trigger | Conditions | Notifications Sent |
|-----------|---------|------------|-------------------|
| Invited → Active | User completes activation workflow | Activation link valid (<7 days); user completes profile setup; user enrolls ≥1 auth method | User: "Welcome to Motadata OneOps! Your account is now active." |
| Invited → Invitation Expired | 7 days elapse since invitation sent | No user action taken | User: "Your invitation link has expired. Contact your administrator to resend." |
| Active → Suspended | Admin clicks "Suspend User" OR automated policy trigger (e.g., failed compliance review) | User does not own critical resources OR admin overrides | User: "Your account has been suspended. Contact your administrator." Admin: "User [Name] has been suspended." |
| Suspended → Active | Admin clicks "Reactivate User" | Admin provides justification (optional) | User: "Your account has been reactivated. You can now login." Admin: "User [Name] has been reactivated." |
| Active/Suspended → Deactivated | Admin clicks "Deactivate User" | User does not own critical resources (tickets, workflows, assets) OR admin reassigns resources first | User: "Your account has been deactivated. Contact your administrator if you believe this is an error." Admin: "User [Name] has been deactivated. Data will be retained for 90 days." |
| Deactivated → Active | Admin clicks "Reactivate User" within 90-day window | Data still retained (within 90 days); admin provides justification | User: "Your account has been reactivated. You can now login." Admin: "User [Name] has been reactivated." |
| Deactivated → Deleted | 90 days elapse OR admin initiates GDPR erasure request | Retention period expired OR GDPR request approved | Admin: "User [Name] data has been permanently deleted." (User not notified if deleted) |

---

## 4. FLOWS

### Flow 1: Individual User Invitation

#### Entry & Pre-conditions
- **Trigger:** Tenant Admin clicks "Invite User" button in Admin Portal → Users section
- **Requires:**
  - Admin has `users.invite` permission
  - Tenant has not exceeded user limit (configurable per tenant)
  - User is authenticated and session active

#### Steps

1. **Admin Enters User Details**
   - **User sees:** Modal dialog with title "Invite New User"
     - Fields:
       - "Email Address*" (text input, required)
       - "First Name" (text input, optional)
       - "Last Name" (text input, optional)
       - "Role" (dropdown, optional, defaults to tenant default role)
       - "Send invitation email" (checkbox, checked by default)
     - Buttons: "Send Invitation" (primary), "Cancel" (secondary)
   - **User does:** Enters invitee email, optionally fills name/role, clicks "Send Invitation"
   - **Validation:**
     - Email required, max 255 chars, RFC 5322 format, case-insensitive, stored lowercase
     - Email uniqueness within tenant (not across tenants)
     - Tenant user limit not exceeded
   - **Errors:**
     - Email blank → "Email address is required."
     - Invalid email format → "Please enter a valid email address (e.g., user@example.com)."
     - Duplicate email → "A user with this email address already exists in your organization."
     - Tenant limit exceeded → "Your organization has reached the maximum user limit ({{limit}}). Contact support to increase your limit."
   - **System responds:** Validates input; if valid, generates 7-day activation link with cryptographically secure token (256-bit entropy)
   - **Success:** Modal closes; toast notification: "Invitation sent to {{email}}."
   - **Loading:** Button shows spinner, text changes to "Sending...", modal disabled during send (<3 seconds)

2. **System Sends Invitation Email**
   - **System does:** Queues invitation email via Email/Notification Service
   - **User sees:** Toast notification: "Invitation email sent successfully."
   - **Validation:** Email service available; if unavailable, retries 3 times with exponential backoff
   - **Errors:**
     - Email service unavailable after retries → "Unable to send invitation email. Please try again later or contact support."
   - **System responds:** Logs invitation event (admin ID, invitee email, timestamp, activation link expiry)

#### Business Logic

**Rule: Invitation Link Generation**
- **Inputs:** Invitee email, tenant ID, admin ID, timestamp
- **Logic:** Generate unique activation token using secure random generator (256-bit entropy); store token hash (bcrypt, 10 rounds) with expiry timestamp (current time + 7 days)
- **Output:** Activation URL: `https://{{tenant_subdomain}}.oneops.com/activate?token={{token}}`
- **Edge cases:**
  - If user already invited but not activated, regenerate new token and invalidate old token
  - If user previously deactivated and within 90-day retention window, reactivation flow takes precedence (not new invitation)

**Rule: Tenant User Limit Enforcement**
- **Inputs:** Tenant ID, current user count, tenant max user limit
- **Logic:** Count active + invited + suspended users (exclude deactivated); if count >= limit, block invitation
- **Output:** Allow (count < limit) or Block (count >= limit)
- **Edge cases:** System Admins can override limit for emergency provisioning (requires justification)

#### Post-conditions
- User record created in "Invited" state
- Activation link sent to invitee email (valid 7 days)
- Invitation event logged immutably (admin, timestamp, email, expiry)
- User appears in admin dashboard with "Invited" badge

---

### Flow 2: Bulk User Invitation via CSV

#### Entry & Pre-conditions
- **Trigger:** Tenant Admin clicks "Bulk Invite Users" button in Admin Portal → Users section
- **Requires:**
  - Admin has `users.invite.bulk` permission
  - Tenant has capacity for bulk users (limit - current users >= CSV row count)
  - CSV file follows required format (email, first_name, last_name, role)

#### Steps

1. **Admin Uploads CSV File**
   - **User sees:** Modal dialog with title "Bulk Invite Users"
     - File upload dropzone: "Drag and drop CSV file here or click to browse"
     - Link: "Download CSV template" (downloads sample CSV with headers)
     - CSV format requirements:
       - Headers: `email,first_name,last_name,role` (first_name, last_name, role optional)
       - Max 1,000 rows per file
       - UTF-8 encoding
     - Buttons: "Upload & Validate" (primary), "Cancel" (secondary)
   - **User does:** Uploads CSV file, clicks "Upload & Validate"
   - **Validation:**
     - File type: .csv only
     - File size: max 5MB
     - CSV parsing: valid UTF-8, proper delimiters
     - Row validation: email format, uniqueness within CSV and tenant, role validity
   - **Errors:**
     - Invalid file type → "Please upload a CSV file (.csv extension)."
     - File too large → "File size exceeds 5MB limit. Please reduce the number of users or split into multiple files."
     - CSV parsing error (row X) → "Error on row {{row}}: {{error_detail}} (e.g., 'Invalid email format: user@')."
     - Duplicate email within CSV → "Row {{row}}: Email {{email}} appears multiple times in the CSV."
     - Duplicate email in tenant → "Row {{row}}: User {{email}} already exists in your organization."
     - Tenant limit exceeded → "Cannot invite {{row_count}} users. Your organization has {{available}} slots available out of {{limit}}."
   - **System responds:** Parses CSV, validates all rows, displays validation results
   - **Success:** Validation panel appears showing:
     - "✅ {{valid_count}} users ready to invite"
     - "❌ {{error_count}} users with errors (see details below)"
     - Error details table (row, email, error message)
     - Buttons: "Invite Valid Users" (enabled if valid_count > 0), "Cancel"
   - **Loading:** Progress bar during validation (<5 seconds for 1,000 rows)

2. **Admin Confirms Bulk Invitation**
   - **User sees:** Validation results panel (from Step 1)
   - **User does:** Reviews validation results; clicks "Invite Valid Users" to proceed
   - **System responds:** Creates user records for valid rows, generates activation links, queues invitation emails
   - **Success:** Modal closes; toast notification: "Inviting {{valid_count}} users... Email notifications are being sent."
   - **Loading:** Progress bar: "Inviting users... {{progress}}%" (<5 minutes for 100 users)

3. **System Sends Bulk Invitation Emails**
   - **System does:** Queues all invitation emails asynchronously; tracks send status
   - **User sees:** Toast notification updates:
     - "Invitation emails queued. This may take a few minutes."
     - After completion: "{{success_count}} invitations sent successfully. {{failed_count}} failed."
   - **Admin Action:** Admin can view bulk invitation job status in Admin Portal → Jobs section
   - **Errors:**
     - Email send failures → Admin sees failed email list with "Retry" button

#### Business Logic

**Rule: CSV Row-by-Row Validation**
- **Inputs:** CSV file, tenant ID, tenant user limit
- **Logic:**
  - Parse CSV headers; validate required column `email` exists
  - For each row:
    - Validate email format (RFC 5322), uniqueness within CSV and tenant
    - Validate role (if provided) exists in tenant role catalog
    - Validate tenant user limit (current + pending rows <= limit)
  - Aggregate validation results: valid rows, error rows (row number, email, error message)
- **Output:** Validation report (valid_count, error_count, error_details)
- **Edge cases:**
  - Empty rows skipped (not counted as errors)
  - Duplicate emails within CSV: only first occurrence valid, subsequent marked as errors
  - Invalid roles: default to tenant default role (logged as warning)

**Rule: Bulk Invitation Transaction**
- **Inputs:** Validated CSV rows, tenant ID, admin ID
- **Logic:**
  - Create user records in batch (transaction: all-or-nothing for valid rows)
  - Generate activation tokens for all users
  - Queue invitation emails asynchronously (failures tracked but don't block user creation)
- **Output:** User records created, invitation emails queued
- **Edge cases:**
  - If email service fails for some users, user records still created; admin notified of send failures with "Retry" option

#### Post-conditions
- Valid user records created in "Invited" state
- Activation links sent to all valid invitee emails
- Bulk invitation job logged (admin, timestamp, valid_count, error_count, CSV filename)
- Admin receives summary notification (email) with success/failure counts

---

### Flow 3: User Activation

#### Entry & Pre-conditions
- **Trigger:** User clicks activation link in invitation email
- **Requires:**
  - Activation token valid (not expired, not used)
  - User record in "Invited" state

#### Steps

1. **User Lands on Activation Page**
   - **User sees:** Activation page with:
     - Header: "Welcome to Motadata OneOps"
     - Subtext: "Complete your profile to get started"
     - Form fields:
       - "First Name*" (text input, pre-filled if provided during invitation)
       - "Last Name*" (text input, pre-filled if provided during invitation)
       - "Phone Number" (text input, optional, E.164 format)
       - "Timezone*" (dropdown, defaults to browser-detected timezone)
       - "Language" (dropdown, defaults to browser language or tenant default)
     - Button: "Continue" (primary)
   - **User does:** Reviews/edits profile fields, clicks "Continue"
   - **Validation:**
     - First Name required, max 100 chars, trim whitespace
     - Last Name required, max 100 chars, trim whitespace
     - Phone optional, E.164 format if provided
     - Timezone required (selected from IANA timezone database)
   - **Errors:**
     - First Name blank → "First name is required."
     - Last Name blank → "Last name is required."
     - Invalid phone → "Please enter a valid phone number (e.g., +1-555-123-4567)."
     - Activation link expired → "This activation link has expired. Please contact your administrator to resend the invitation."
     - Activation link already used → "This activation link has already been used. Please login to your account."
   - **System responds:** Validates profile data; if valid, proceeds to authentication enrollment
   - **Success:** Transitions to Step 2
   - **Loading:** Button shows spinner, text changes to "Saving..." (<2 seconds)

2. **User Enrolls Passwordless Authentication Method**
   - **User sees:** Authentication enrollment page with:
     - Header: "Set Up Your Login Method"
     - Subtext: "Choose how you want to access your account. You can add more methods later."
     - Options (tabs):
       - "Email OTP" (default selected): "Receive a one-time code via email each time you login"
       - "Magic Link": "Receive a login link via email (click to login)"
       - "Authenticator App (TOTP)": "Use an authenticator app like Google Authenticator or Authy"
     - Button: "Activate Account" (primary)
   - **User does:** Selects authentication method, clicks "Activate Account"
   - **Validation:**
     - At least one authentication method must be selected
   - **Errors:**
     - No method selected → "Please select at least one authentication method."
   - **System responds:** Enrolls selected authentication method(s); transitions user to "Active" state
   - **Success:** User redirected to dashboard with toast: "Your account is now active. Welcome!"
   - **Loading:** Button shows spinner, text changes to "Activating..." (<3 seconds)

   **Authentication Method Details:**
   - **Email OTP:** No additional setup required; user receives OTP at registered email on each login
   - **Magic Link:** No additional setup required; user receives single-use link at registered email on each login
   - **Authenticator App (TOTP):** User sees QR code + manual entry key; must scan QR code with authenticator app and enter 6-digit verification code to confirm enrollment

#### Business Logic

**Rule: Activation Token Validation**
- **Inputs:** Activation token (from URL), user record
- **Logic:**
  - Retrieve user by token hash
  - Check token expiry (current time <= created_at + 7 days)
  - Check token usage (used_at IS NULL)
  - Mark token as used (set used_at = current time)
- **Output:** Valid (proceed) or Invalid (show error)
- **Edge cases:**
  - If token expired, display "Link expired" error with "Resend Invitation" button (requires admin approval)
  - If token already used, display "Link already used" error with "Go to Login" button

**Rule: Profile Data Update**
- **Inputs:** First name, last name, phone, timezone, language, user ID
- **Logic:** Update user record with profile data; validate field constraints; apply XSS sanitization and whitespace trimming
- **Output:** User profile updated
- **Edge cases:** If phone provided, validate E.164 format; if invalid, show error and block activation

**Rule: Authentication Method Enrollment**
- **Inputs:** Selected authentication method(s), user ID
- **Logic:**
  - Email OTP/Magic Link: Mark as enabled (no additional setup)
  - TOTP: Generate TOTP secret, display QR code, require user to verify 6-digit code before enabling
- **Output:** Authentication method(s) enrolled; user transitioned to "Active" state
- **Edge cases:**
  - If TOTP verification fails (invalid code), allow 3 retry attempts before showing "Invalid code. Please ensure your device clock is synchronized."
  - If user closes page mid-activation, activation token remains valid until expiry (user can resume)

#### Post-conditions
- User state transitioned from "Invited" to "Active"
- User profile updated with first name, last name, phone, timezone, language
- At least one passwordless authentication method enrolled
- Activation event logged (user ID, timestamp, authentication method(s))
- Welcome notification sent to user
- Admin notified: "{{User Name}} has activated their account."

---

### Flow 4: User Suspension

#### Entry & Pre-conditions
- **Trigger:** Admin clicks "Suspend User" action in Admin Portal → Users section OR automated policy trigger (e.g., failed compliance review)
- **Requires:**
  - Admin has `users.suspend` permission
  - User is in "Active" state
  - Admin provides justification (optional but recommended for audit)

#### Steps

1. **Admin Initiates Suspension**
   - **User sees:** Confirmation modal:
     - Title: "Suspend User: {{User Name}}?"
     - Message: "This user will be immediately logged out and unable to access the system. Role assignments will be retained for audit purposes."
     - Field: "Reason for Suspension" (textarea, optional, max 500 chars)
     - Buttons: "Suspend User" (danger), "Cancel" (secondary)
   - **User does:** Optionally enters suspension reason, clicks "Suspend User"
   - **Validation:** No validation required (reason optional)
   - **System responds:** Transitions user to "Suspended" state, revokes active sessions, sends notifications
   - **Success:** Modal closes; toast notification: "{{User Name}} has been suspended."
   - **Loading:** Button shows spinner, text changes to "Suspending..." (<3 seconds)

2. **System Revokes Active Sessions**
   - **System does:** Terminates all active sessions for suspended user; user immediately logged out if currently active
   - **User (suspended) sees:** If logged in, session ends; redirected to login page with message: "Your account has been suspended. Contact your administrator."
   - **System responds:** Logs session revocation events

#### Business Logic

**Rule: Suspension Validation**
- **Inputs:** User ID, admin ID, reason (optional)
- **Logic:**
  - Verify user is in "Active" state (cannot suspend already suspended/deactivated users)
  - Verify admin has `users.suspend` permission
  - Suspend user: transition state to "Suspended", retain role assignments, revoke sessions
- **Output:** User suspended
- **Edge cases:**
  - Cannot suspend own account (admin cannot suspend themselves; blocked with error: "You cannot suspend your own account.")
  - Cannot suspend last active System Administrator in tenant (blocked with error: "Cannot suspend the last System Administrator. Assign this role to another user first.")

**Rule: Session Revocation**
- **Inputs:** User ID
- **Logic:** Query all active session tokens for user; invalidate all tokens (mark as revoked); log session revocation events
- **Output:** All sessions terminated
- **Edge cases:** If user has 0 active sessions, suspension still proceeds (no error)

#### Post-conditions
- User state transitioned to "Suspended"
- All active sessions revoked
- Role assignments retained (not removed)
- Suspension event logged (admin ID, user ID, timestamp, reason, previous state)
- Notifications sent:
  - User: "Your account has been suspended. Contact your administrator."
  - Admin: "User {{Name}} has been suspended by {{Admin Name}}."

---

### Flow 5: User Deactivation

#### Entry & Pre-conditions
- **Trigger:** Admin clicks "Deactivate User" action in Admin Portal → Users section
- **Requires:**
  - Admin has `users.deactivate` permission
  - User is in "Active" or "Suspended" state
  - User does not own critical resources (open tickets, active workflows, assigned assets) OR admin reassigns resources first
  - Admin provides justification (optional but recommended for audit)

#### Steps

1. **System Checks Resource Ownership**
   - **System does:** Query resources owned by user:
     - Open tickets (status: New, In Progress, Pending)
     - Active workflows (status: Running, Paused)
     - Assigned assets (endpoint devices, IT assets)
   - **User sees:** If resources found, modal displays:
     - Title: "Cannot Deactivate User: {{User Name}}"
     - Message: "This user owns critical resources that must be reassigned before deactivation:"
     - List:
       - "{{count}} open tickets (view details)"
       - "{{count}} active workflows (view details)"
       - "{{count}} assigned assets (view details)"
     - Buttons: "Reassign Resources" (primary), "Cancel" (secondary)
   - **User does:** Clicks "Reassign Resources" to view resource assignment UI OR clicks "Cancel"
   - **Validation:** Resource ownership must be resolved before deactivation proceeds
   - **Errors:** If resources exist, deactivation blocked with above modal

2. **Admin Confirms Deactivation**
   - **User sees:** (After resource ownership resolved OR user owns no resources) Confirmation modal:
     - Title: "Deactivate User: {{User Name}}?"
     - Message: "This user will be immediately logged out and unable to access the system. Data will be retained for 90 days, after which it will be permanently deleted."
     - Warning: "⚠️ This action cannot be undone without admin intervention."
     - Field: "Reason for Deactivation" (textarea, optional, max 500 chars)
     - Buttons: "Deactivate User" (danger), "Cancel" (secondary)
   - **User does:** Optionally enters deactivation reason, clicks "Deactivate User"
   - **Validation:** No validation required (reason optional)
   - **System responds:** Transitions user to "Deactivated" state, revokes sessions, schedules data deletion
   - **Success:** Modal closes; toast notification: "{{User Name}} has been deactivated. Data will be retained for 90 days."
   - **Loading:** Button shows spinner, text changes to "Deactivating..." (<5 seconds)

3. **System Schedules Data Deletion**
   - **System does:** Sets `scheduled_deletion_date = current_time + 90 days`; queues background job to delete user data after 90 days
   - **System responds:** Logs deactivation event with scheduled deletion timestamp

#### Business Logic

**Rule: Resource Ownership Validation**
- **Inputs:** User ID
- **Logic:**
  - Query open tickets (status: New, In Progress, Pending) where `assigned_to = user_id`
  - Query active workflows (status: Running, Paused) where `owner = user_id`
  - Query assigned assets where `assigned_to = user_id`
  - Aggregate counts
- **Output:** Allow (count = 0) or Block (count > 0 with resource details)
- **Edge cases:**
  - If admin has `users.deactivate.force` permission, allow deactivation with resource ownership (requires justification; logged as high-risk action)
  - Resources owned by deactivated user remain assigned (tracked as "Orphaned" until reassigned)

**Rule: Deactivation Transition**
- **Inputs:** User ID, admin ID, reason (optional)
- **Logic:**
  - Verify user is in "Active" or "Suspended" state
  - Verify admin has `users.deactivate` permission
  - Deactivate user: transition state to "Deactivated", revoke all access, revoke sessions, schedule data deletion (90 days)
- **Output:** User deactivated
- **Edge cases:**
  - Cannot deactivate own account (admin cannot deactivate themselves; blocked with error: "You cannot deactivate your own account.")
  - Cannot deactivate last active System Administrator in tenant (blocked with error: "Cannot deactivate the last System Administrator. Assign this role to another user first.")

**Rule: Data Retention Scheduling**
- **Inputs:** User ID, deactivation timestamp
- **Logic:** Set `scheduled_deletion_date = deactivation_timestamp + 90 days`; queue background job for automatic deletion
- **Output:** Scheduled deletion date recorded
- **Edge cases:**
  - Tenant can configure retention period (30 days to 7 years based on compliance requirements); 90 days is default
  - GDPR erasure request overrides retention period (immediate deletion upon admin approval)

#### Post-conditions
- User state transitioned to "Deactivated"
- All active sessions revoked
- Role assignments removed
- Data retention scheduled (90 days)
- Deactivation event logged (admin ID, user ID, timestamp, reason, scheduled_deletion_date)
- Notifications sent:
  - User: "Your account has been deactivated. Contact your administrator if you believe this is an error."
  - Admin: "User {{Name}} has been deactivated by {{Admin Name}}. Data will be retained until {{scheduled_deletion_date}}."

---

### Flow 6: User Reactivation (Within 90-Day Window)

#### Entry & Pre-conditions
- **Trigger:** Admin clicks "Reactivate User" action in Admin Portal → Users section (Deactivated Users filter)
- **Requires:**
  - Admin has `users.reactivate` permission
  - User is in "Deactivated" state
  - User data still retained (within 90-day window)
  - Admin provides justification

#### Steps

1. **Admin Confirms Reactivation**
   - **User sees:** Confirmation modal:
     - Title: "Reactivate User: {{User Name}}?"
     - Message: "This user will regain access to the system with their previous role assignments restored."
     - Field: "Reason for Reactivation*" (textarea, required, max 500 chars)
     - Buttons: "Reactivate User" (primary), "Cancel" (secondary)
   - **User does:** Enters reactivation reason, clicks "Reactivate User"
   - **Validation:** Reason required (min 10 chars)
   - **Errors:**
     - Reason blank → "Reason for reactivation is required."
     - Reason too short → "Please provide a detailed reason (minimum 10 characters)."
     - User data deleted (past 90 days) → "Cannot reactivate user. Data has been permanently deleted."
   - **System responds:** Transitions user to "Active" state, restores role assignments, cancels scheduled deletion
   - **Success:** Modal closes; toast notification: "{{User Name}} has been reactivated."
   - **Loading:** Button shows spinner, text changes to "Reactivating..." (<3 seconds)

#### Business Logic

**Rule: Reactivation Eligibility**
- **Inputs:** User ID, current timestamp
- **Logic:**
  - Verify user is in "Deactivated" state
  - Verify `scheduled_deletion_date > current_time` (data still retained)
  - Verify admin has `users.reactivate` permission
- **Output:** Allow (eligible) or Block (data deleted or user not deactivated)
- **Edge cases:**
  - If user data already deleted (past 90 days), reactivation blocked; admin must create new invitation instead
  - Role assignments restored from historical snapshot (roles assigned at time of deactivation)

**Rule: Role Restoration**
- **Inputs:** User ID
- **Logic:** Query user's role assignments at time of deactivation (stored in audit log); restore role assignments
- **Output:** Role assignments restored
- **Edge cases:**
  - If roles no longer exist in tenant (deleted after deactivation), admin notified: "Some roles could not be restored (deleted): {{role_names}}. Please reassign manually."

#### Post-conditions
- User state transitioned to "Active"
- Role assignments restored
- Scheduled data deletion cancelled
- Reactivation event logged (admin ID, user ID, timestamp, reason)
- Notifications sent:
  - User: "Your account has been reactivated. You can now login."
  - Admin: "User {{Name}} has been reactivated by {{Admin Name}}."

---

## 5. VALIDATION

| Field/Input | Rule | Timing | Error Message | Why |
|-------------|------|--------|---------------|-----|
| **Email Address** | Required, max 255 chars, RFC 5322 format, unique within tenant, case-insensitive, stored lowercase | On form submit (invite, bulk CSV) | "Email address is required." / "Please enter a valid email address (e.g., user@example.com)." / "A user with this email address already exists in your organization." | Prevent duplicate users; ensure email deliverability; comply with industry standards (RFC 5322) |
| **First Name** | Required during activation, max 100 chars, trim whitespace, XSS sanitization | On activation form submit | "First name is required." | Basic profile completeness for personalization and audit trails |
| **Last Name** | Required during activation, max 100 chars, trim whitespace, XSS sanitization | On activation form submit | "Last name is required." | Basic profile completeness for personalization and audit trails |
| **Phone Number** | Optional, E.164 format if provided, stored with country code | On activation form submit | "Please enter a valid phone number (e.g., +1-555-123-4567)." | Support international users; enable SMS/voice-based recovery (future) |
| **Timezone** | Required during activation, IANA timezone database value | On activation form submit | "Timezone is required." | Display timestamps in user's local timezone; improve UX |
| **Activation Token** | Valid token (not expired, not used), matches user record | On activation page load | "This activation link has expired. Please contact your administrator to resend the invitation." / "This activation link has already been used. Please login to your account." | Prevent replay attacks; enforce 7-day expiry for security |
| **CSV File (Bulk Invite)** | .csv extension, max 5MB, UTF-8 encoding, max 1,000 rows, valid headers (email required) | On CSV upload | "Please upload a CSV file (.csv extension)." / "File size exceeds 5MB limit. Please reduce the number of users or split into multiple files." / "Error on row {{row}}: {{error_detail}}." | Prevent system overload; ensure data quality; support large-scale onboarding |
| **Tenant User Limit** | Current users (active + invited + suspended) < tenant max limit | On invite (individual or bulk) | "Your organization has reached the maximum user limit ({{limit}}). Contact support to increase your limit." / "Cannot invite {{row_count}} users. Your organization has {{available}} slots available out of {{limit}}." | Enforce licensing constraints; prevent abuse; drive upsell |
| **Suspension Reason** | Optional, max 500 chars | On suspension confirm | N/A (optional field) | Provide audit context; support compliance reviews |
| **Deactivation Reason** | Optional, max 500 chars | On deactivation confirm | N/A (optional field) | Provide audit context; support compliance reviews |
| **Reactivation Reason** | Required, min 10 chars, max 500 chars | On reactivation confirm | "Reason for reactivation is required." / "Please provide a detailed reason (minimum 10 characters)." | Ensure high-risk action is justified; support compliance audits |
| **Resource Ownership (Deactivation)** | User owns 0 critical resources (open tickets, active workflows, assigned assets) | On deactivation initiation | "Cannot deactivate user. This user owns critical resources that must be reassigned before deactivation: {{count}} open tickets, {{count}} active workflows, {{count}} assigned assets." | Prevent operational disruption; ensure business continuity |

---

## 6. PERMISSIONS

| Action | Required Permission | Allowed Roles | Scope |
|--------|-------------------|---------------|-------|
| Invite individual user | `users.invite` | Tenant Admin, System Administrator | Tenant |
| Bulk invite users (CSV) | `users.invite.bulk` | Tenant Admin, System Administrator | Tenant |
| View user list | `users.read` | Tenant Admin, System Administrator, Security Officer | Tenant |
| View user details | `users.read` | Tenant Admin, System Administrator, Security Officer, Manager (for direct reports only) | Tenant |
| Suspend user | `users.suspend` | Tenant Admin, System Administrator, Security Officer | Tenant |
| Deactivate user | `users.deactivate` | Tenant Admin, System Administrator | Tenant |
| Deactivate user (force, with resource ownership) | `users.deactivate.force` | System Administrator only | Tenant |
| Reactivate user | `users.reactivate` | Tenant Admin, System Administrator | Tenant |
| Resend invitation | `users.invite` | Tenant Admin, System Administrator | Tenant |
| View lifecycle audit logs | `audit.read` | Security Officer, Auditor | Tenant |
| Configure tenant user limits | `tenants.configure` | System Administrator only | Global |
| Initiate GDPR erasure request | `users.gdpr.erase` | System Administrator, Privacy Officer | Tenant |

**Special Rules:**
- Admins cannot suspend/deactivate their own accounts
- Cannot suspend/deactivate last active System Administrator in tenant (blocked)
- Managers can view lifecycle states of direct reports but cannot modify

---

## 7. NOTIFICATIONS

### Notification 1: User Invitation Email

- **Trigger:** Admin invites individual user OR bulk invite job completes successfully
- **To:** Invitee email address
- **Channel:** Email
- **Subject:** "You're invited to join {{tenant_name}} on Motadata OneOps"
- **Body:**
```
Hi {{first_name}},

{{admin_name}} has invited you to join {{tenant_name}} on Motadata OneOps.

To activate your account and get started, click the link below:

[Activate Your Account] → {{activation_link}}

This link is valid for 7 days. If you don't activate your account within this time, you'll need to request a new invitation from your administrator.

Need help? Contact your administrator at {{admin_email}}.

Welcome aboard!
— The Motadata OneOps Team

---
Variables:
{{first_name}} = Invitee first name (if provided during invitation) or "there"
{{admin_name}} = Name of admin who sent invitation
{{tenant_name}} = Tenant display name
{{activation_link}} = Activation URL with secure token
{{admin_email}} = Tenant admin contact email
```

---

### Notification 2: Activation Link Expired

- **Trigger:** User clicks activation link after 7 days have elapsed
- **To:** User sees message on activation page (no email sent)
- **Channel:** In-app message
- **Message:**
```
This activation link has expired.

Activation links are valid for 7 days. Please contact your administrator to resend the invitation.

[Contact Administrator] → mailto:{{admin_email}}
```

---

### Notification 3: Account Activated (Welcome)

- **Trigger:** User completes activation workflow successfully
- **To:** User email + in-app toast
- **Channel:** Email + in-app
- **Subject:** "Welcome to Motadata OneOps!"
- **Body (Email):**
```
Hi {{first_name}},

Your account is now active! You can access Motadata OneOps anytime at:

{{tenant_url}}

To get started:
1. Explore your dashboard
2. Check out our quick-start guide
3. Connect with your team

Need help? Visit our Help Center or contact support.

Welcome to the team!
— The Motadata OneOps Team

---
Variables:
{{first_name}} = User first name
{{tenant_url}} = Tenant login URL (e.g., https://acme.oneops.com)
```

**In-app toast:** "Your account is now active. Welcome!"

---

### Notification 4: Admin Notification - User Activated

- **Trigger:** User completes activation workflow successfully
- **To:** Tenant Admins (who invited the user OR all admins with `users.read` permission)
- **Channel:** Email (digest: batched hourly if multiple activations)
- **Subject:** "User activation: {{user_name}}"
- **Body:**
```
{{user_name}} ({{email}}) has activated their account.

Activation completed at: {{timestamp}}
Authentication method enrolled: {{auth_method}}

View user details: {{user_profile_link}}

— Motadata OneOps

---
Variables:
{{user_name}} = User full name
{{email}} = User email
{{timestamp}} = Activation timestamp (user's timezone)
{{auth_method}} = "Email OTP", "Magic Link", "Authenticator App (TOTP)"
{{user_profile_link}} = Direct link to user profile in Admin Portal
```

---

### Notification 5: Account Suspended

- **Trigger:** Admin suspends user OR automated policy suspends user
- **To:** User email + active sessions terminated
- **Channel:** Email + in-app (session termination)
- **Subject:** "Your account has been suspended"
- **Body:**
```
Hi {{first_name}},

Your account on Motadata OneOps has been suspended.

Reason: {{reason}}

You will not be able to access the system until your account is reactivated by an administrator.

If you believe this is an error, please contact your administrator at {{admin_email}}.

— The Motadata OneOps Team

---
Variables:
{{first_name}} = User first name
{{reason}} = Suspension reason (if provided) or "Not specified"
{{admin_email}} = Tenant admin contact email
```

**In-app (on login attempt):** "Your account has been suspended. Contact your administrator."

---

### Notification 6: Admin Notification - User Suspended

- **Trigger:** Admin suspends user OR automated policy suspends user
- **To:** Tenant Admins with `users.read` permission
- **Channel:** Email
- **Subject:** "User suspended: {{user_name}}"
- **Body:**
```
{{user_name}} ({{email}}) has been suspended.

Suspended by: {{admin_name}}
Suspended at: {{timestamp}}
Reason: {{reason}}

View user details: {{user_profile_link}}

— Motadata OneOps

---
Variables:
{{user_name}} = User full name
{{email}} = User email
{{admin_name}} = Admin who suspended user or "System (automated policy)"
{{timestamp}} = Suspension timestamp (admin's timezone)
{{reason}} = Suspension reason (if provided) or "Not specified"
{{user_profile_link}} = Direct link to user profile in Admin Portal
```

---

### Notification 7: Account Reactivated

- **Trigger:** Admin reactivates suspended or deactivated user
- **To:** User email
- **Channel:** Email
- **Subject:** "Your account has been reactivated"
- **Body:**
```
Hi {{first_name}},

Good news! Your account on Motadata OneOps has been reactivated.

You can now login and access the system at:

{{tenant_url}}

If you have any questions, please contact your administrator at {{admin_email}}.

Welcome back!
— The Motadata OneOps Team

---
Variables:
{{first_name}} = User first name
{{tenant_url}} = Tenant login URL
{{admin_email}} = Tenant admin contact email
```

---

### Notification 8: Account Deactivated

- **Trigger:** Admin deactivates user
- **To:** User email
- **Channel:** Email
- **Subject:** "Your account has been deactivated"
- **Body:**
```
Hi {{first_name}},

Your account on Motadata OneOps has been deactivated.

Reason: {{reason}}

Your data will be retained for 90 days. After this period, your data will be permanently deleted.

If you believe this is an error, please contact your administrator at {{admin_email}} immediately.

— The Motadata OneOps Team

---
Variables:
{{first_name}} = User first name
{{reason}} = Deactivation reason (if provided) or "Not specified"
{{admin_email}} = Tenant admin contact email
```

---

### Notification 9: Admin Notification - User Deactivated

- **Trigger:** Admin deactivates user
- **To:** Tenant Admins with `users.read` permission
- **Channel:** Email
- **Subject:** "User deactivated: {{user_name}}"
- **Body:**
```
{{user_name}} ({{email}}) has been deactivated.

Deactivated by: {{admin_name}}
Deactivated at: {{timestamp}}
Reason: {{reason}}
Data retention: Until {{scheduled_deletion_date}}

View user details: {{user_profile_link}}

— Motadata OneOps

---
Variables:
{{user_name}} = User full name
{{email}} = User email
{{admin_name}} = Admin who deactivated user
{{timestamp}} = Deactivation timestamp (admin's timezone)
{{reason}} = Deactivation reason (if provided) or "Not specified"
{{scheduled_deletion_date}} = Date when user data will be permanently deleted (current + 90 days)
{{user_profile_link}} = Direct link to user profile in Admin Portal
```

---

### Notification 10: Bulk Invitation Summary (Admin)

- **Trigger:** Bulk invite job completes (all emails sent or failed)
- **To:** Admin who initiated bulk invite
- **Channel:** Email
- **Subject:** "Bulk invitation complete: {{success_count}} users invited"
- **Body:**
```
Your bulk user invitation has been processed.

Summary:
- Successfully invited: {{success_count}} users
- Failed: {{failed_count}} users

{{#if failed_count > 0}}
Failed invitations (email send errors):
{{#each failed_users}}
- {{email}}: {{error_message}}
{{/each}}

You can retry failed invitations from the Admin Portal: {{retry_link}}
{{/if}}

View all users: {{users_list_link}}

— Motadata OneOps

---
Variables:
{{success_count}} = Number of successfully invited users
{{failed_count}} = Number of failed invitations
{{failed_users}} = Array of {email, error_message} for failed sends
{{retry_link}} = Link to retry failed invitations
{{users_list_link}} = Link to Admin Portal Users list
```

---

## 8. ERROR SCENARIOS

| Scenario | System Behavior | User Sees | Recovery |
|----------|-----------------|-----------|----------|
| **Email service unavailable during invitation** | Retry 3 times with exponential backoff (1s, 2s, 4s); if all retries fail, log error and display to admin | Toast: "Unable to send invitation email. Please try again later or contact support." | Admin can retry invitation from user list (action: "Resend Invitation") |
| **User clicks expired activation link** | Display "Link expired" message on activation page | "This activation link has expired. Please contact your administrator to resend the invitation." | User contacts admin; admin resends invitation (generates new 7-day token) |
| **User clicks already-used activation link** | Display "Link already used" message on activation page | "This activation link has already been used. Please login to your account." [Go to Login button] | User navigates to login page |
| **Duplicate email during invitation** | Block invitation, display error | "A user with this email address already exists in your organization." | Admin verifies email address or manages existing user record |
| **Tenant user limit exceeded during invitation** | Block invitation, display error with current limit | "Your organization has reached the maximum user limit ({{limit}}). Contact support to increase your limit." | Admin contacts support to increase tenant user limit OR deactivates unused users to free slots |
| **Tenant user limit exceeded during bulk invite** | Validate CSV before processing; block entire job if limit exceeded | "Cannot invite {{row_count}} users. Your organization has {{available}} slots available out of {{limit}}." | Admin reduces CSV row count OR increases tenant limit before retrying |
| **CSV parsing error (invalid format, encoding)** | Display validation errors with row numbers | "Error on row {{row}}: Invalid email format (user@)" / "Error: CSV file must be UTF-8 encoded." | Admin corrects CSV file and re-uploads |
| **User attempts deactivation while owning resources** | Query resource ownership; block deactivation if resources found, display resource list | "Cannot deactivate user. This user owns critical resources that must be reassigned before deactivation: {{count}} open tickets, {{count}} active workflows, {{count}} assigned assets." [Reassign Resources button] | Admin reassigns resources via resource management UI, then retries deactivation |
| **Admin attempts to suspend own account** | Block action, display error | "You cannot suspend your own account." | Admin requests another admin to perform suspension |
| **Admin attempts to deactivate last System Administrator** | Block action, display error | "Cannot deactivate the last System Administrator. Assign this role to another user first." | Admin assigns System Administrator role to another user before deactivating |
| **Concurrent activation by two users with same email (race condition)** | First activation succeeds; second activation fails with duplicate email error | (Second user) "A user with this email address already exists in your organization." | Second user contacts admin; admin investigates and corrects invitation |
| **Concurrent suspension by two admins (race condition)** | First suspension succeeds; second suspension fails with "User already suspended" error | (Second admin) Toast: "User is already suspended." | Admin refreshes user list to see updated state |
| **Reactivation attempt after 90-day retention period** | Block reactivation, display error | "Cannot reactivate user. Data has been permanently deleted." | Admin creates new invitation instead |
| **Authentication method enrollment fails (TOTP verification)** | Allow 3 retry attempts; after 3 failures, display error | "Invalid verification code. Please ensure your device clock is synchronized and try again." (After 3 attempts) "Unable to verify authenticator app. Please contact support." | User verifies device clock is correct, retries; if persistent, user contacts support |
| **Network timeout during activation** | Retry request; if timeout persists, display error | "Activation request timed out. Please check your connection and try again." [Retry button] | User retries activation; activation token remains valid until expiry |

---

### Concurrency

| Concurrent Action | Detection | Resolution | User Experience |
|-------------------|-----------|------------|-----------------|
| **Two admins invite same email simultaneously** | Database unique constraint on (tenant_id, email) | First invitation succeeds; second fails with duplicate error | Second admin sees: "A user with this email address already exists in your organization." |
| **Two users activate same invitation token simultaneously** | Database unique constraint on token usage (`used_at` timestamp) | First activation succeeds; second fails with "token already used" | Second user sees: "This activation link has already been used. Please login to your account." |
| **Admin suspends user while user is completing activation** | Check user state before suspension; if activation in-progress, block suspension for 5 seconds, then retry | Activation completes first OR suspension proceeds after timeout | Admin sees brief "Processing..." state; suspension succeeds after activation |
| **Admin deactivates user while another admin reactivates** | Optimistic locking on user state transitions (version field) | First action succeeds; second fails with stale state error | Second admin sees: "User state has changed. Please refresh and try again." |
| **Bulk invite job processes CSV while admin invites same email manually** | Row-by-row validation checks email uniqueness at time of processing | Manual invitation succeeds; CSV row fails with duplicate error | Bulk invite summary shows failed row: "{{email}}: User already exists" |

---

## 9. INTEGRATIONS

| System | What We Need | Why | Failure Handling |
|--------|--------------|-----|------------------|
| **Email/Notification Service** | Send invitation emails, activation confirmation, suspension/deactivation notifications | All user lifecycle transitions require email notifications to users and admins | Retry 3 times with exponential backoff; if all fail, log error, display admin notification: "Email send failed. You can retry from the user list." User record still created (email can be resent). |
| **CH_001: Passwordless Authentication Service** | Enroll authentication methods during activation (Email OTP, Magic Link, TOTP) | Users must enroll at least one authentication method during activation to access system | If authentication enrollment fails, activation blocked; user sees error with support contact. Activation token remains valid for retry. |
| **Resource Ownership Service (ITSM, ITOM, Endpoint, CRM modules)** | Query open tickets, active workflows, assigned assets owned by user before deactivation | Prevent deactivation of users who own critical resources; ensure business continuity | If query fails (service unavailable), deactivation blocked with error: "Unable to verify resource ownership. Please try again later." Admin can use `users.deactivate.force` permission to override (logged as high-risk). |
| **Audit Logging Service** | Log all lifecycle state transitions (invite, activate, suspend, deactivate, reactivate) with actor, timestamp, reason | SOC 2, GDPR compliance require immutable audit trails for user lifecycle events | If audit logging fails, lifecycle action blocked (fail-closed); admin sees error: "Unable to complete action due to logging failure. Please contact support." Ensures no unlogged actions. |
| **Directory Sync Service (CH_004)** | Receive automated suspension events when user removed from directory | Enterprise directories (LDAP, SCIM) trigger user suspensions when employees leave organization | If directory sync unavailable, manual suspension still available; admins notified of sync failure for manual review. |

---

## 10. DATA RETENTION

**User Profile Data:**
- **Purpose:** Store user identity, profile attributes, state, and lifecycle metadata
- **What must persist:**
  - Core identity: email (unique within tenant, lowercase), first name, last name, phone, timezone, language
  - Lifecycle state: current state (Invited, Active, Suspended, Deactivated), state history (audit log)
  - Timestamps: created_at, activated_at, suspended_at, deactivated_at, scheduled_deletion_date
  - Authentication methods: enrolled methods (Email OTP, Magic Link, TOTP) linked to user record
  - Role assignments: current roles, historical roles (retained in audit log)
- **Uniqueness rules:**
  - Email unique within tenant (not across tenants)
  - User ID (UUID) globally unique
  - Activation token unique and single-use (marked as used after activation)
- **Required fields:** Email (mandatory); first name, last name, timezone (mandatory during activation)
- **Deletion rules:**
  - **Deactivation:** User state → "Deactivated"; all access revoked; data retained for 90 days (configurable per tenant: 30 days to 7 years)
  - **After 90 days:** User data permanently deleted (user record, profile, role assignments, authentication methods); audit logs retained (not deleted)
  - **GDPR erasure request:** Admin-initiated; immediate deletion upon approval (within 30 days of request); audit logs anonymized (user ID replaced with "DELETED_USER_{{uuid}}")

**Activation Tokens:**
- **Purpose:** Secure, time-limited tokens for user activation
- **What must persist:** Token hash (bcrypt, 10 rounds), expiry timestamp (created_at + 7 days), used_at timestamp (null until used)
- **Uniqueness rules:** Token globally unique (256-bit entropy)
- **Deletion rules:** Tokens deleted 30 days after expiry or use (no longer needed after activation)

**Lifecycle Audit Logs:**
- **Purpose:** Immutable audit trail for compliance (SOC 2, GDPR, HIPAA)
- **What must persist:**
  - Event type: invite_sent, activation_completed, user_suspended, user_deactivated, user_reactivated, user_deleted
  - Actor: admin ID or "system" (for automated events)
  - Target: user ID, email
  - Timestamp: event timestamp (UTC)
  - Metadata: reason (if provided), previous state, new state, resource ownership details (for deactivation)
- **Uniqueness rules:** Event ID (UUID) globally unique
- **Required fields:** All fields mandatory for audit completeness
- **Deletion rules:** Audit logs retained for 365 days minimum (configurable per tenant: 1-7 years); never deleted for compliance-regulated tenants (HIPAA, SOX)

**CSV Bulk Import Jobs:**
- **Purpose:** Track bulk invitation jobs for admin visibility and retry
- **What must persist:**
  - Job ID (UUID), admin ID, tenant ID, timestamp
  - CSV filename, row count (total, valid, error)
  - Job status: pending, processing, completed, failed
  - Error details: array of {row, email, error_message} for failed rows
- **Uniqueness rules:** Job ID globally unique
- **Deletion rules:** Job records retained for 90 days (admin visibility), then deleted; audit logs retained separately

---

## 11. NON-FUNCTIONAL

### Performance
- **Invitation send latency:** Individual invitation <3 seconds (95th percentile) from admin action to email queued
- **Bulk invitation processing:** 100 users invited within 5 minutes (CSV validation + email queuing)
- **Activation flow completion:** <1 hour (95th percentile) from invitation sent to activation completed (user-driven)
- **Lifecycle state change latency:** <5 seconds (95th percentile) for suspend/deactivate/reactivate actions
- **Session revocation latency:** <3 seconds for active session termination after suspend/deactivate
- **CSV validation:** <5 seconds for 1,000-row CSV parsing and validation

### Security
- **Authentication required:** All lifecycle management actions require authenticated admin session with valid permissions
- **Activation token security:** 256-bit entropy, bcrypt hashing (10 rounds), single-use, 7-day expiry
- **Session revocation:** All active sessions immediately terminated on suspend/deactivate (token invalidation)
- **PII protection:** User emails encrypted at rest (AES-256); activation tokens never logged in plaintext
- **RBAC enforcement:** Permission checks on all actions (users.invite, users.suspend, users.deactivate, users.reactivate)
- **Self-action prevention:** Admins blocked from suspending/deactivating own accounts
- **Last admin protection:** Cannot deactivate last System Administrator in tenant

### Compliance

**GDPR:**
- **Right-to-erasure:** User data deleted within 30 days of admin-initiated erasure request (overrides standard 90-day retention)
- **Data minimization:** Only necessary profile fields collected (email, name, phone, timezone)
- **Audit trails:** All lifecycle events logged immutably; logs anonymized after user deletion (user ID → "DELETED_USER_{{uuid}}")
- **Consent:** Users acknowledge data collection during activation (implicit consent via activation action)

**SOC 2 Type II:**
- **Immutable audit logging:** All lifecycle state transitions logged with actor, timestamp, reason, previous/new state
- **Access controls:** Role-based permissions enforced for all actions
- **Data retention:** Audit logs retained for 365 days minimum (configurable per tenant: 1-7 years)
- **Encryption:** User data encrypted in transit (TLS 1.3) and at rest (AES-256)

**HIPAA (for regulated tenants):**
- **Audit trail completeness:** 100% capture of lifecycle events (no missed events)
- **Access logging:** All admin actions logged with actor ID, IP address, timestamp
- **Data retention:** User data and audit logs retained for 7 years (HIPAA requirement for regulated tenants)

### Localization
- **Languages (MVP):** English (US), German, French, Spanish
- **Regional formats:**
  - Date/time: Displayed in user's timezone (from profile); ISO 8601 format in logs (UTC)
  - Phone numbers: E.164 format for storage; displayed with country code
- **Email templates:** Localized based on user's language preference (from profile or browser default)
- **Timezone handling:** All timestamps stored in UTC; displayed in user's timezone in UI

---

## 12. ACCEPTANCE CRITERIA

### Functional

| Criterion | Tests Req | Priority | Status |
|-----------|-----------|----------|--------|
| Admin can send individual user invitation with valid email; user receives activation email within 30 seconds | FR-UCM-001 | P0 | [ ] |
| User clicks activation link, completes profile setup, enrolls authentication method, and account transitions to Active state | FR-UCM-003, FR-UCM-004 | P0 | [ ] |
| Activation link expires after 7 days; expired link shows error message with admin contact | FR-UCM-010 | P0 | [ ] |
| Admin can bulk invite 100 users via CSV; all valid invitations sent within 5 minutes | FR-UCM-002 | P0 | [ ] |
| CSV validation detects duplicate emails, invalid formats, and tenant limit violations before processing | FR-UCM-002, FR-UCM-009 | P0 | [ ] |
| Admin can suspend active user; user immediately logged out and cannot login | FR-UCM-005 | P0 | [ ] |
| Suspended user sees "Account suspended" error on login attempt | FR-UCM-005 | P0 | [ ] |
| Admin can deactivate user with no resource ownership; user access revoked and data retention scheduled (90 days) | FR-UCM-006, FR-UCM-007 | P0 | [ ] |
| Admin cannot deactivate user who owns open tickets/workflows/assets; error message displays resource details | FR-UCM-006 | P0 | [ ] |
| Admin can reactivate deactivated user within 90-day window; user regains access with restored roles | FR-UCM-006 | P1 | [ ] |
| Tenant user limit enforced; invitation blocked when limit reached with clear error message | FR-UCM-009 | P0 | [ ] |
| Active user must have at least one valid authentication method enrolled (validated during activation) | SEC-UCM-001 | P0 | [ ] |
| Admin cannot suspend or deactivate own account; error message displayed | Security | P0 | [ ] |
| Cannot deactivate last System Administrator in tenant; error message displayed | Security | P0 | [ ] |
| GDPR erasure request deletes user data within 30 days; audit logs anonymized | COMP-UCM-001 | P0 | [ ] |
| All lifecycle state transitions logged immutably with actor, timestamp, reason, previous/new state | COMP-UCM-002 | P0 | [ ] |

### Non-Functional

| Criterion | Priority | Status |
|-----------|----------|--------|
| Invitation send latency <3 seconds (95th percentile) from admin action to email queued | P0 | [ ] |
| Bulk invitation processing completes within 5 minutes for 100 users | P0 | [ ] |
| Activation success rate >95% (users who click activation link successfully complete activation) | P0 | [ ] |
| Invitation-to-activation time <1 hour (95th percentile) from invitation sent to user completion | P0 | [ ] |
| Lifecycle state change latency <5 seconds (95th percentile) for suspend/deactivate/reactivate | P0 | [ ] |
| Session revocation completes within 3 seconds after suspend/deactivate | P0 | [ ] |
| CSV validation completes within 5 seconds for 1,000-row file | P1 | [ ] |
| Activation tokens use 256-bit entropy and bcrypt hashing (10 rounds) | P0 | [ ] |
| All user data encrypted in transit (TLS 1.3) and at rest (AES-256) | P0 | [ ] |
| Audit log capture completeness 100% (no missed lifecycle events) | P0 | [ ] |
| Audit logs retained for 365 days minimum (configurable per tenant) | P0 | [ ] |
| GDPR erasure request processed within 30 days | P0 | [ ] |

---

## 13. TEST SCENARIOS

### Happy Path

- **Scenario 1:** Admin sends individual invitation → User receives email within 30 seconds → User clicks activation link → User completes profile setup → User enrolls Email OTP → Account transitions to Active → Tests [FR-UCM-001, FR-UCM-003, FR-UCM-004] → [P0]
- **Scenario 2:** Admin uploads CSV with 100 valid users → CSV validation passes → Bulk invite job completes within 5 minutes → All 100 users receive invitation emails → Tests [FR-UCM-002] → [P0]
- **Scenario 3:** Admin suspends active user → User's active session immediately terminated → User sees "Account suspended" error on next login attempt → Tests [FR-UCM-005] → [P0]
- **Scenario 4:** Admin deactivates user with no resource ownership → User access revoked → Data retention scheduled for 90 days → Admin and user receive deactivation notifications → Tests [FR-UCM-006, FR-UCM-007] → [P0]
- **Scenario 5:** Admin reactivates deactivated user within 90-day window → User regains access → Role assignments restored → User can login successfully → Tests [FR-UCM-006] → [P1]

### Error Handling

- **Scenario 1:** Admin attempts to invite user with duplicate email → Invitation blocked → Error message: "A user with this email address already exists in your organization." → Tests [FR-UCM-001] → [P0]
- **Scenario 2:** User clicks activation link after 7 days → Activation page shows "Link expired" error → User contacts admin → Admin resends invitation → Tests [FR-UCM-010] → [P0]
- **Scenario 3:** Admin attempts to invite user when tenant limit reached → Invitation blocked → Error message: "Your organization has reached the maximum user limit ({{limit}}). Contact support to increase your limit." → Tests [FR-UCM-009] → [P0]
- **Scenario 4:** Admin uploads CSV with 10 invalid rows (bad email formats, duplicates) → CSV validation fails for 10 rows → Validation report shows error details → Admin corrects CSV and re-uploads → Tests [FR-UCM-002] → [P0]
- **Scenario 5:** Admin attempts to deactivate user who owns 5 open tickets → Deactivation blocked → Error message displays resource details → Admin reassigns tickets → Retries deactivation successfully → Tests [FR-UCM-006] → [P0]
- **Scenario 6:** Admin attempts to suspend own account → Action blocked → Error message: "You cannot suspend your own account." → Tests [Security] → [P0]
- **Scenario 7:** Admin attempts to deactivate last System Administrator → Action blocked → Error message: "Cannot deactivate the last System Administrator. Assign this role to another user first." → Tests [Security] → [P0]
- **Scenario 8:** Email service unavailable during invitation → System retries 3 times → All retries fail → Error message: "Unable to send invitation email. Please try again later or contact support." → Admin can retry from user list → Tests [Integration] → [P0]
- **Scenario 9:** Admin attempts to reactivate user after 90-day retention period (data deleted) → Reactivation blocked → Error message: "Cannot reactivate user. Data has been permanently deleted." → Tests [FR-UCM-007] → [P1]

### Edge Cases

- **Scenario 1:** User clicks activation link twice (concurrency) → First activation succeeds → Second activation fails with "Link already used" error → Tests [Concurrency] → [P0]
- **Scenario 2:** Two admins invite same email simultaneously (concurrency) → First invitation succeeds → Second fails with duplicate error → Tests [Concurrency] → [P0]
- **Scenario 3:** User receives invitation but already has active account (email collision, edge case) → Invitation blocked during send → Admin sees error → Tests [FR-UCM-001] → [P1]
- **Scenario 4:** Admin uploads CSV with 1,001 rows (exceeds max limit) → CSV validation fails → Error message: "CSV file exceeds maximum 1,000 rows. Please split into multiple files." → Tests [FR-UCM-002] → [P1]
- **Scenario 5:** User completes profile setup but closes browser before authentication enrollment → Activation token remains valid → User can resume activation later → Tests [FR-UCM-003] → [P1]
- **Scenario 6:** Admin deactivates user with `users.deactivate.force` permission (overrides resource ownership check) → Deactivation succeeds → Action logged as high-risk → Tests [FR-UCM-006, Security] → [P1]
- **Scenario 7:** GDPR erasure request submitted for deactivated user → User data deleted within 30 days → Audit logs anonymized (user ID → "DELETED_USER_{{uuid}}") → Tests [COMP-UCM-001] → [P0]
- **Scenario 8:** Directory sync service triggers automated suspension (user removed from LDAP) → User suspended → Admin notified → Tests [Integration with CH_004] → [P1]
- **Scenario 9:** User activates account with TOTP enrollment → TOTP verification fails 3 times → Error message: "Unable to verify authenticator app. Please contact support." → Tests [FR-UCM-004, Error Handling] → [P1]

### Security

- **Scenario 1:** Activation token uses 256-bit entropy and bcrypt hashing (10 rounds) → Token security validated → Tests [SEC-UCM-001, Security] → [P0]
- **Scenario 2:** Unauthorized user attempts to access admin invitation API → Request blocked with 403 Forbidden → Tests [Security] → [P0]
- **Scenario 3:** User without `users.invite` permission attempts to send invitation → Action blocked with 403 Forbidden → Tests [Security] → [P0]
- **Scenario 4:** Session revocation after suspend/deactivate completes within 3 seconds → User immediately logged out → Tests [Security] → [P0]
- **Scenario 5:** All lifecycle state transitions logged immutably with actor, timestamp, reason → Audit log completeness 100% → Tests [COMP-UCM-002] → [P0]
- **Scenario 6:** User data encrypted in transit (TLS 1.3) and at rest (AES-256) → Encryption validated → Tests [Security] → [P0]
- **Scenario 7:** Activation link contains plaintext token (not logged) → Token security validated → Tests [Security] → [P0]

---

## 14. ROLLOUT

**Standard release (no phased rollout required)**

- Feature released to all tenants simultaneously (GA)
- No feature flags or phased rollout needed (low-risk, core capability)
- Rollback criteria:
  - Critical bug (P0): Data loss, security breach, system outage → Immediate rollback
  - High-severity bug (P1): Widespread activation failures (>10% failure rate) → Rollback within 4 hours

---

## 15. DOCUMENTATION

### Help Articles Needed

1. **"How to Invite Users to Your Organization"** - Covers individual and bulk invitation workflows, CSV format, validation errors
2. **"User Activation Guide"** - Covers activation link expiry, profile setup, authentication method enrollment, troubleshooting expired links
3. **"Managing User States: Suspend vs. Deactivate"** - Explains differences between suspension and deactivation, use cases, data retention policies
4. **"Bulk User Onboarding with CSV Import"** - Step-by-step guide for CSV format, validation rules, error resolution, retry failed invitations
5. **"GDPR Compliance: User Data Deletion and Right-to-Erasure"** - Explains data retention policies, GDPR erasure request process, audit log anonymization

### Tooltips Needed

- **"Invite User" button (Admin Portal)** → "Send an invitation email to a new user with a 7-day activation link."
- **"Bulk Invite Users" button (Admin Portal)** → "Upload a CSV file to invite multiple users at once (max 1,000 users per file)."
- **"Suspend User" action** → "Temporarily block user access. Active sessions will be terminated immediately. Role assignments are retained for audit purposes."
- **"Deactivate User" action** → "Permanently remove user access. Data will be retained for 90 days, after which it will be permanently deleted."
- **"Reactivate User" action** → "Restore access for a suspended or deactivated user (within 90-day retention window)."
- **"Reason for Suspension" field** → "Optional. Provide context for audit trails and compliance reviews."
- **"Reason for Deactivation" field** → "Optional. Provide context for audit trails and compliance reviews."
- **"Reason for Reactivation" field** → "Required. Explain why this user is being reactivated (minimum 10 characters)."
- **Tenant user limit warning (invitation form)** → "Your organization is nearing the user limit ({{current}}/{{limit}}). Contact support to increase your limit."

### Support Readiness

- **Common issue 1:** User reports "Activation link expired"
  - **Resolution:** Admin resends invitation from Admin Portal → Users → [Select User] → Actions → "Resend Invitation"
  - **Escalate if:** User reports link expired within 7 days (indicates system issue; escalate to engineering)

- **Common issue 2:** Admin reports "Cannot deactivate user" error
  - **Resolution:** Check if user owns critical resources (open tickets, workflows, assets); guide admin to reassign resources before deactivating
  - **Escalate if:** Resource ownership check fails (service unavailable; escalate to engineering)

- **Common issue 3:** Bulk invite CSV validation fails with "duplicate email" errors
  - **Resolution:** Guide admin to review CSV for duplicate rows; explain email uniqueness within tenant (not across tenants)
  - **Escalate if:** Admin confirms no duplicates but validation still fails (indicates bug; escalate to engineering)

- **Common issue 4:** User reports "Account suspended" but admin did not suspend
  - **Resolution:** Check audit logs for suspension event (automated policy trigger or directory sync); explain automated suspension to admin
  - **Escalate if:** No audit log entry found (indicates logging failure; escalate to engineering)

- **Common issue 5:** GDPR erasure request not completed within 30 days
  - **Resolution:** Check job status in Admin Portal → Compliance → GDPR Requests; retry if failed
  - **Escalate if:** Job status shows "completed" but data not deleted (escalate to engineering + legal)

---

## 16. OUT OF SCOPE

### Deferred (Future Versions)

- **Self-service user invitation:** End users inviting external collaborators without admin approval (deferred to v2.0)
- **Invitation approval workflows:** Require manager approval before invitation sent (deferred to v2.0)
- **Custom activation flows:** Tenant-specific activation steps (e.g., NDA acceptance, training completion) (deferred to v2.0)
- **User import from external systems:** Automated bulk import from HRIS systems (Workday, BambooHR) (deferred to CH_004 Directory Sync)
- **SMS-based activation:** Send activation link via SMS instead of email (deferred to v2.0)
- **Temporary user accounts:** Time-bound user access (auto-deactivate after X days) (deferred to v2.0)
- **User transfer between tenants:** Move user from one tenant to another (deferred to v2.0)

### Excluded (Not Building)

- **Password-based authentication during activation:** Platform is passwordless-only (users must enroll OTP, magic link, or TOTP)
- **User self-registration:** Public signup without invitation (enterprise-only platform; invitation-only model)
- **User profile editing by end users:** Profile fields (name, email, phone) managed by admins only (self-service profile editing deferred to future chapter)
- **Custom user fields:** Tenant-defined custom attributes during invitation (deferred to v2.0)
- **Invitation templates:** Custom invitation email templates per tenant (default template only in v1.0)

---

## 17. OPEN QUESTIONS

*None at this time. All requirements specified with industry-standard defaults applied.*

---

## 18. ASSUMPTIONS

### Technical

- **Assumption:** Email/Notification Service is available and reliable (99.9% uptime SLA)
  - **Risk if false:** Invitation emails not delivered; users cannot activate accounts
  - **Mitigation:** Retry logic (3 attempts with exponential backoff); admin can resend invitations manually; email send failures tracked and displayed in Admin Portal

- **Assumption:** Passwordless Authentication Service (CH_001) is deployed and operational
  - **Risk if false:** Users cannot enroll authentication methods during activation; activation blocked
  - **Mitigation:** Activation flow validates authentication service availability before enrollment; clear error messages if service unavailable

- **Assumption:** Resource Ownership Service (ITSM, ITOM, Endpoint, CRM modules) provides accurate resource ownership data
  - **Risk if false:** Users with critical resources incorrectly deactivated; operational disruption
  - **Mitigation:** Admins can use `users.deactivate.force` permission to override (logged as high-risk); resource queries cached for 5 minutes (acceptable staleness)

### Business

- **Assumption:** Default tenant user limit is 100 users; System Admins can increase per tenant
  - **Confidence:** High (confirmed by product team)

- **Assumption:** Default data retention period is 90 days after deactivation; configurable per tenant (30 days to 7 years)
  - **Confidence:** High (SOC 2, GDPR, HIPAA requirements validated)

- **Assumption:** Activation link expiry is 7 days (industry standard for invitation links)
  - **Confidence:** High (aligns with security best practices)

- **Assumption:** Tenant Admins are responsible for managing user invitations (no delegation to end users in v1.0)
  - **Confidence:** Medium (self-service invitation deferred to v2.0)

---

## APPENDIX

### Glossary

| Term | Definition | Usage |
|------|------------|-------|
| **Activation Link** | Time-limited, single-use URL sent to invitee email for account activation (valid 7 days) | User clicks activation link to complete profile setup and enroll authentication method |
| **Invited State** | User state after invitation sent but before activation completed | User cannot login until activation workflow completed |
| **Active State** | User state after activation; full access per assigned roles | User can login and perform actions |
| **Suspended State** | User state with access temporarily blocked; role assignments retained | Admin suspends user for security or compliance reasons; user cannot login |
| **Deactivated State** | User state with access permanently removed; data scheduled for deletion after 90 days | Admin deactivates user (offboarding); user cannot login; data retained for compliance |
| **Tenant User Limit** | Maximum number of users (active + invited + suspended) allowed per tenant | Enforced during invitation to prevent exceeding licensing constraints |
| **Resource Ownership** | User owns critical resources (open tickets, active workflows, assigned assets) that must be reassigned before deactivation | Prevents deactivation of users with outstanding work; ensures business continuity |
| **GDPR Erasure Request** | Admin-initiated request to permanently delete user data (overrides standard 90-day retention) | Complies with GDPR right-to-erasure; completed within 30 days |
| **Bulk Invitation Job** | Background job that processes CSV file and sends invitations to multiple users | Tracks job status, validation errors, and email send failures for admin visibility |
| **Activation Token** | Cryptographically secure token (256-bit entropy, bcrypt hashed) embedded in activation link | Single-use, 7-day expiry; prevents replay attacks |

---

### References

- **PRD:** [PRD_USER_ACCESS.md](d:\Learning_Projects\Claude\spec-kit\.specify\prds\PRD_USER_ACCESS.md)
- **Chapter Plan:** [PRD_USER_ACCESS_chapter_plan.md](d:\Learning_Projects\Claude\spec-kit\.specify\prds\PRD_USER_ACCESS_chapter_plan.md) - CH_002
- **Designs:** [Figma designs for user lifecycle flows - To be created]
- **Related specs:**
  - CH_001: Passwordless Authentication (OTP, Magic Links, TOTP) - Prerequisite for activation flow
  - CH_004: Directory Sync (LDAP/SCIM) - Enables automated suspension via directory integration
  - CH_006: RBAC (Role-Based Access Control) - Role assignments managed during lifecycle
  - CH_012: Audit Logging & Compliance Reporting - Lifecycle events logged for compliance

---

**End of DFS: User Lifecycle Management (Invite to Deactivate)**
