# PRD: Tenant Onboarding System
**PRD ID:** PRD_TENANT_ONBOARDING
**Version:** v1.0 — 2025-10-24
**Owner:** Product Lead
**Status:** Draft

---

## 1 — Executive Summary

The Tenant Onboarding System delivers a self-service, scalable onboarding experience that accelerates time-to-first-value, maximizes trial-to-paid conversion, and eliminates manual operations across the tenant lifecycle. Primary outcome: achieve p50 TTFV under 10 minutes (vs SaaS industry average of ~1.5 days), drive 25-30% trial conversion, and enable 85%+ self-serve onboardings with minimal support burden. This capability strengthens Motadata OneOps' Multi-Tenant by Design pillar and Composable Ecosystem readiness.

---

## 2 — Problem Statement

Current tenant onboarding is manual, slow, and error-prone, resulting in poor conversion rates, high support costs, and delayed time-to-value. Industry benchmarks show SaaS average TTFV at ~1.5 days and onboarding completion rates at ~19%, far below user expectations in the self-service era. Without automated, intelligent provisioning workflows supporting both Enterprise and MSP use cases, the platform cannot achieve GTM velocity, competitive differentiation, or sustainable growth. Solving this now is critical for platform adoption, MSP expansion, and establishing ExperienceOps leadership.

---

## 3 — Goals & Success Metrics

**Goal 1: Accelerate Time-to-First-Value**
- **Metric:** TTFV (p50 and p90)
- **Target:** p50 < 10 minutes, p90 < 30 minutes (vs SaaS avg ~1.5 days)
- **Measurement:** Analytics funnel tracking time from signup completion to first meaningful action (e.g., first device discovered, first ticket created); baseline TBD; 6-month target

**Goal 2: Maximize Trial-to-Paid Conversion**
- **Metric:** Trial-to-Paid Conversion Rate
- **Target:** 25-30%
- **Measurement:** (Trials upgraded to paid) / (Total trials created) × 100; baseline TBD; measured over trial duration; 6-month target

**Goal 3: Increase Onboarding Completion Rate**
- **Metric:** Onboarding Completion Rate
- **Target:** ≥40% (vs SaaS avg ~19%)
- **Measurement:** (Tenants completing all onboarding steps) / (Tenants starting onboarding) × 100; baseline TBD; 6-month target

**Goal 4: Achieve High Self-Service Adoption**
- **Metric:** Self-Serve Onboarding Rate
- **Target:** ≥85%
- **Measurement:** (Tenants onboarded with zero manual intervention) / (Total tenants onboarded) × 100; includes MSP flows; baseline TBD; 6-month target

**Goal 5: Minimize Support Burden**
- **Metric:** Support Tickets per 100 Tenants (first 30 days)
- **Target:** ≤5 tickets per 100 tenants
- **Measurement:** (Onboarding-related support tickets in first 30 days) / (Tenants onboarded) × 100; baseline TBD; 6-month target

**Baseline Note:** This is a new system with no prior onboarding metrics. Baselines will be established during pre-MVP pilot phase and documented before GA launch. Industry benchmarks referenced for target setting.

---

## 4 — Scope (In / Out)

**In-scope (capability level)**
- Provisioning & Setup — Automated and manual tenant provisioning workflows with single-region support (MVP); multi-region expansion planned for Phase 2
- License & Subscription Management — Plan enforcement, trial management, MSP self-service license allocation (MSP provider buys in bulk and assigns to customers via dedicated interface, removing SaaS admin overhead)
- Communication — Lifecycle notifications, transactional messaging, delivery reliability
- Monitoring & Analytics — Onboarding performance visibility, conversion tracking, health dashboards
- Administration — Tenant discovery, support access, lifecycle management, bulk operations
- Usage-Based Billing & Metering (high-level) — AI feature usage (workflow executions, AI agent credits/tokens), feature flag management, count-based limits (technicians, assets)
- Post-Onboarding Lifecycle Management — Renewal tracking, expansion monitoring, usage analytics
- Security & Compliance — Audit trails, data residency, encryption, multi-tenant isolation

**Out-of-scope**
- AI-assisted onboarding (intelligent provisioning, predictive recommendations, autonomous troubleshooting)
- Predictive analytics or machine learning models for conversion optimization and churn prediction
- Cross-tenant analytics or insights aggregation
- CRM integration for lead-to-tenant conversion tracking (deferred to Phase 2)
- Data migration tooling (handled post-onboarding within tenant workspace via dedicated migration interface and tools)
- Detailed billing/metering implementation (high-level capability in-scope; detailed implementation deferred to module-specific PRDs)
---

## 5 — Personas & Primary Stakeholders

**Primary Personas:**
- **Tenant Admin** — Signs up, configures workspace, manages users and integrations
- **Platform Admin** — Oversees tenant provisioning, monitors system health, troubleshoots issues
- **Support Engineer** — Assists tenants with setup, resolves onboarding blockers

**Primary Stakeholders:**
- Product Lead (PRD owner, business outcome accountability)
- IT Operations (infrastructure provisioning, multi-region readiness)
- Customer Success (onboarding experience, conversion optimization)
- Compliance Officer (data residency, audit requirements, regulatory adherence)
- Security Representative (multi-tenant isolation, encryption, access controls)

---

## 6 — Functional Requirements — Capability-Level

### Capability: `Provisioning & Setup`

**Purpose:** Enable automated self-service and manual enterprise tenant provisioning with multi-region support, rollback on failure, and lifecycle state tracking.

**Actors:** Tenant Admin, Platform Admin, Support Engineer

**Behavior (capability-level):**
- Validate signup credentials (email verification, domain validation, anti-fraud checks)
- Allocate infrastructure resources (compute, storage, networking) based on plan selection
- Apply configuration templates matching use case (ITSM-only, ITOM-focused, Full Suite, MSP)
- Enable modules and features per subscription plan (ITSM, ITOM, Endpoint, Security, CRM)
- Provision tenant workspace with unique URL and initial admin credentials
- Support assisted provisioning with approval checkpoints for enterprise workflows
- Provide multi-region deployment selection with visibility into data residency constraints
- Track tenant lifecycle through defined states: Pending → Provisioning → Active → Suspended → Deactivated
- Rollback failed provisioning attempts with cleanup of all tenant artifacts (no orphaned resources)

**Business Rules / Constraints:**
- Tenant subdomain must be globally unique and comply with DNS naming rules (lowercase, alphanumeric, hyphens only)
- Email verification required before infrastructure provisioning begins
- Trial tenants default to shared infrastructure; paid tenants eligible for dedicated resources (plan-dependent)
- Multi-region selection locked after provisioning (migration requires separate workflow)
- Rollback must complete within 5 minutes and restore system to pre-provisioning state
- State transitions logged with audit trail (actor, timestamp, reason)

**Success Criteria (capability):**
- Self-service tenant provisioned from signup to Active state in <60 seconds (p95)
- Assisted provisioning workflow completes approval gates within SLA (e.g., 24-hour enterprise response)
- Zero orphaned resources after rollback (verified via infrastructure scan)
- 100% of tenants reach Active or Deactivated state (no stuck Provisioning states >5 minutes)

---

### Capability: `License & Subscription Management`

**Purpose:** Enforce subscription plans, manage trial access, control feature availability, and support MSP self-service license allocation with 2-level hierarchy (MSP Provider → Customer).

**Actors:** Tenant Admin, Platform Admin

**Behavior (capability-level):**
- Define subscription plans with associated capabilities (module access, user limits, storage quotas, API rate limits)
- Manage trial lifecycle (start date, duration, expiration, grace period, conversion)
- Control feature and module availability based on plan tier (Basic, Professional, Enterprise, MSP)
- Support MSP 2-level model: MSP Provider purchases licenses in bulk and self-manages allocation to customers via dedicated interface
- MSP Provider assigns licenses from their own pool to individual customers (offloading SaaS admin overhead)
- MSP customers access platform via unique URL path under MSP subdomain (e.g., msp.domain.com/customer1, msp.domain.com/customer2)
- Enforce usage limits and quotas (user count, storage, API calls, integrations)
- Provide upgrade pathways when limits approached (proactive notifications, self-serve upgrade CTAs)
- Block feature access when plan downgraded or trial expired (graceful degradation, data retention)

**Business Rules / Constraints:**
- Trial duration defaults to 14 days; custom durations require approval
- MSP hierarchy strictly limited to 2 levels: MSP Provider → Customer only (no nested MSPs allowed)
- License allocation must not exceed MSP provider's purchased pool capacity (real-time validation)
- Direct customers and MSP providers treated identically during provisioning; differentiation only in license management interface
- MSP provider receives subdomain; each customer accessed via unique URL path under MSP subdomain
- Grace period of 7 days after trial expiration before data archival
- Downgrades apply immediately to feature access; data remains accessible in read-only mode for 30 days

**Success Criteria (capability):**
- Trial-to-paid conversion workflow completes in <2 minutes (payment processing + plan upgrade)
- MSP license allocation updates reflect in customer tenant within 10 seconds
- Usage limit enforcement triggers within 1 minute of quota breach (e.g., user count exceeded)
- Zero unauthorized feature access post-downgrade or trial expiration (verified via access control audit)
---

### Capability: `Communication`

**Purpose:** Deliver reliable, personalized lifecycle notifications and transactional messages across onboarding journey with multi-provider redundancy.

**Actors:** Tenant Admin, Platform Admin, System (automated triggers)

**Behavior (capability-level):**
- Send account verification emails (signup confirmation, email validation link)
- Trigger onboarding guidance messages (welcome email, setup checklist, feature highlights)
- Deliver lifecycle notifications (trial expiration reminders, upgrade prompts, renewal alerts)
- Support transactional system messages (password reset, user invitations, access grants)
- Ensure reliable delivery with multi-provider redundancy (primary: SendGrid/SES; fallback: secondary provider)
- Personalize messaging based on context (tenant name, plan tier, onboarding progress, user role)
- Track delivery status and engagement (open rates, click-through rates, bounce rates)

**Business Rules / Constraints:**
- Email verification must be sent within 60 seconds of signup
- Trial expiration reminders sent at 7 days, 3 days, and 1 day before expiration
- Transactional emails (password reset, invitations) must be delivered within 2 minutes
- Fallback provider activates automatically if primary fails 3 consecutive sends
- Personalization tokens (tenant name, user name, URLs) must be sanitized to prevent injection attacks
- Unsubscribe link required for marketing emails; transactional emails exempt
- Email delivery logs retained for 90 days for compliance and troubleshooting

**Success Criteria (capability):**
- 99.9% email delivery rate (primary + fallback combined)
- Trial expiration reminders sent within 1-hour window of scheduled time
- Transactional email delivery <2 minutes (p95) from trigger event
- Zero email injection vulnerabilities (validated via security audit)

---

### Capability: `Monitoring & Analytics`

**Purpose:** Provide real-time visibility into onboarding performance, conversion progression, operational health, and customer success signals.

**Actors:** Platform Admin, Customer Success, Product Lead

**Behavior (capability-level):**
- Display onboarding funnel metrics (signups → verification → provisioning → first value → trial conversion)
- Show operational health dashboards (provisioning success rate, failure reasons, p95 latency)
- Identify customer success signals (onboarding completion, feature adoption, engagement scores)
- Track conversion progression (trial start → product usage → upgrade intent → paid conversion)
- Alert on significant problems (provisioning failure spike, degraded email delivery, dependency outages)
- Provide drill-down capability into individual tenant onboarding journeys
- Expose metrics via API for external analytics tools and dashboards

**Business Rules / Constraints:**
- Metrics updated in near real-time (≤5-minute lag from event occurrence)
- Alerts triggered when provisioning failure rate exceeds 5% over 15-minute window
- Customer success signals scored based on weighted criteria (completion %, feature usage, time-to-first-value)
- PII (tenant email, names) redacted in shared analytics views (raw data restricted to authorized roles)
- Historical metrics retained for 12 months for trend analysis
- Dashboards must load in <3 seconds (p95) for standard date ranges (7/30/90 days)

**Success Criteria (capability):**
- 100% of provisioning events captured in metrics (no data loss)
- Alerts delivered within 2 minutes of threshold breach
- Dashboard load time <3 seconds (p95) for standard queries
- API endpoint response time <500ms (p95) for metrics retrieval

---

### Capability: `Administration`

**Purpose:** Enable platform admins to discover tenants, access workspaces for support, manage lifecycle states, configure templates, and perform bulk operations.

**Actors:** Platform Admin, Support Engineer

**Behavior (capability-level):**
- Provide tenant discovery with search and filtering (name, domain, plan, state, region, creation date)
- Grant controlled support access to tenant workspaces with accountability (session logging, time-bound access, approval workflow)
- Allow manual lifecycle state updates with audit notes (e.g., Suspend tenant due to payment failure, Reactivate after resolution)
- Manage configuration templates (use case presets, module bundles, default settings)
- Enable bulk operations across multiple tenants (state updates, plan changes, notifications)
- Monitor tenant health and usage patterns (storage consumption, API usage, active users)
- Provide tenant impersonation capability for troubleshooting (with explicit consent logging)

**Business Rules / Constraints:**
- Support access requires approval from authorized admin (exception: P1 incidents allow immediate access with post-facto audit)
- Access sessions time-limited to 4 hours; extension requires re-approval
- All support actions logged with admin identity, timestamp, tenant ID, and action description
- Bulk operations limited to 100 tenants per batch (larger operations require staged execution)
- Lifecycle state changes require audit note (minimum 10 characters, describing reason)
- Tenant impersonation logged and flagged in tenant's audit trail (visible to tenant admin)
- Configuration template changes versioned with rollback capability

**Success Criteria (capability):**
- Tenant search returns results in <1 second for queries on 100K+ tenant database
- Support access granted within 30 seconds of approval
- Bulk operations process at ≥10 tenants/second with error isolation (one failure doesn't block batch)
- 100% of admin actions logged with complete audit trail (no gaps)

---

### Capability: `Security & Compliance`

**Purpose:** Maintain audit trails, enforce regional data residency, protect sensitive information, and ensure multi-tenant isolation with graceful degradation during outages.

**Actors:** Platform Admin, Compliance Officer, Security Representative

**Behavior (capability-level):**
- Log all onboarding activity with immutable audit trail (signup, provisioning, access, state changes, configuration)
- Enforce regional data residency requirements (EU tenants → EU region, US tenants → US region, etc.)
- Encrypt sensitive information at rest (AES-256) and in transit (TLS 1.3)
- Isolate tenant data and configurations (database-level, schema-level, or logical separation per architecture)
- Degrade service gracefully during partial outages (e.g., disable new provisioning, maintain read-only access)
- Validate compliance with GDPR, SOC2, ISO27001 per platform standards
- Provide tenant-specific compliance reports (audit logs, data residency proof, encryption status)

**Business Rules / Constraints:**
- Audit logs immutable and retained for 365 days minimum (GDPR/SOC2 baseline); configurable extended retention for HIPAA, FINRA, and other regulatory requirements (up to 7 years)
- Storage tiering for audit data: Hot storage (0-90 days) for real-time query access; Warm storage (91-365 days) for standard retrieval; Cold storage (>365 days) for compliance-only archival access
- Tenants with HIPAA/healthcare compliance requirements can configure extended retention with automatic tiering to cold storage after 365 days
- Data residency selection locked at provisioning; migration requires explicit tenant consent and compliance review
- Multi-tenant isolation validated quarterly via penetration testing (no cross-tenant data leakage)
- Encryption keys rotated every 90 days with automated key management (AWS KMS, Azure Key Vault)
- Graceful degradation must maintain read-only tenant access even if provisioning services down
- Compliance reports generated on-demand within 24 hours of request
- PII handling follows platform data classification policy (link in Section 8)

**Success Criteria (capability):**
- 100% of onboarding events captured in immutable audit log (no gaps, no tampering)
- Zero cross-tenant data leakage incidents (validated via quarterly penetration tests)
- Data residency enforcement accuracy 100% (no tenants provisioned in wrong region)
- Graceful degradation maintains 99.9% read availability during provisioning service outages
---

## 7 — Non-Functional Requirements

**Performance**
- Self-service tenant provisioning completes in <60 seconds (p95) from signup to Active state
- Database/schema creation completes in <30 seconds (p95)
- Onboarding API endpoints respond in <500ms (p95) for read/write operations
- Dashboard and analytics queries return in <3 seconds (p95) for standard date ranges

**Scalability**
- Platform must support ≥100,000 active tenants per region without degradation
- System must sustain ≥1,000 concurrent provisioning operations per region
- Tenant throughput capacity: ≥50 new tenants activated per hour during peak periods (baseline target for initial SaaS launch; scalable to ≥500/hour with infrastructure expansion in future growth phases)
- License pool allocation updates must scale to 10,000+ MSP customers without latency increase

**Reliability & Availability**
- Onboarding services availability SLA: 99.95% uptime (measured monthly)
- Email delivery reliability: 99.9% (primary + fallback combined)
- Provisioning failure rate: <2% (failures trigger automatic retry with exponential backoff)
- Rollback success rate: 100% (all failed provisioning must clean up completely)

**Security & Privacy (feature-specific)**
- Multi-tenant isolation enforced at database and application layers (no cross-tenant queries possible)
- All tenant data encrypted at rest (AES-256) and in transit (TLS 1.3 minimum)
- Audit events for tenant provisioning, access, and state changes immutable and retained 365 days
- Support access sessions logged with admin identity, timestamp, duration, and actions performed
- API authentication via OAuth 2.0 or API keys with rate limiting (1000 req/min per tenant)
- Secrets (database passwords, API keys, encryption keys) stored in secure vault (AWS Secrets Manager, Azure Key Vault)

**AI-specific NFRs**
- N/A — This PRD explicitly excludes all AI capabilities per requirements

**Observability & Monitoring**
- Provisioning pipeline instrumented with metrics at each stage (signup → validation → infra allocation → DB creation → activation)
- Alerts configured for: provisioning failure rate >5%, email delivery failure >1%, API latency >1s (p95), dependency outages
- Distributed tracing enabled for end-to-end tenant onboarding journey (trace ID propagated across services)
- Logs aggregated and searchable with <1-minute lag (ELK, Splunk, or equivalent)

---

## 8 — Compliance & Data Governance (references + exceptions)

**Platform compliance doc:**
Baseline compliance: GDPR, SOC2 Type II, ISO27001

**Feature exceptions / special requirements:**
- **Data Residency Enforcement:** Tenant region selection (EU, US, APAC) locked at provisioning; data residency validated at infrastructure layer (no cross-region replication without explicit consent)
- **MSP License Pooling:** License allocation data (parent-child relationships, quotas) subject to same retention as tenant metadata (365 days minimum)
- **Audit Trail Immutability:** Onboarding events (provisioning, access, state changes) logged to append-only storage (WORM compliance) with cryptographic hashing for tamper detection
- **Support Access Logging:** All platform admin and support engineer access to tenant workspaces logged with session recordings (optional for P1 incidents, mandatory for non-emergency access)

**Retention/residency note:**
Default platform policy: Tenant metadata retained 365 days post-deactivation; raw data archived per tenant's data retention policy (configurable 30-365 days). Onboarding-specific exception: Audit logs for failed provisioning attempts retained 90 days for operational troubleshooting, then archived for compliance (365 days total).

---

## 9 — Dependencies & Pre-conditions

| Dependency | Owner | Description | Status |
|------------|-------|-------------|--------|
| **Payment Gateway (Stripe/Razorpay)** | Billing Team | Merchant account setup, product/price catalog configuration, webhook signing for trial-to-paid conversion | **OPEN** |
| **Identity/Auth Service (OIDC/SAML via Okta/Azure AD)** | Platform Security | SSO configuration (OIDC/SAML) with MFA support for tenant management portal; optional SCIM for enterprise customers' internal user lifecycle sync | **OPEN** |
| **Email/Notification Service (SES/SendGrid)** | Infrastructure Team | Domain verification, SPF/DKIM/DMARC DNS records, sender reputation, multi-provider failover | **OPEN** |
| **Single-Region Infrastructure (AWS/Azure)** | Cloud Operations | Regional quotas, IAM roles, KMS key provisioning for encryption (single region for MVP; multi-region Phase 2) | **OPEN** |

**Pre-conditions for MVP Launch:**
- Payment Gateway blocker resolved (trial-to-paid conversion requires payment processing)
- Identity/Auth Service (SSO + MFA) and Email/Notification Service in production-ready state
- Single-region infrastructure provisioned with encryption and security controls tested

---

## 10 — Governance & Sign-off

Who must approve the PRD before FRD work starts:

| Role | Name | Required? | Approval Criteria |
|------|------|-----------|-------------------|
| Product Lead | Rakesh Rathod | Yes | Business outcomes, scope, success metrics aligned with roadmap |
| Security & Compliance | Pratik Patel | Yes | Multi-tenant isolation, encryption, audit trail compliance, GDPR/SOC2/data residency requirements validated |
| Platform Architecture | Amit Patel | Yes | Scalability, reliability, dependency readiness confirmed |
| DevOps/Cloud Operations | Pratik Patel | Yes | Infrastructure readiness, deployment automation, monitoring setup validated |
| CTO | Alpesh Dhamelia | Yes (Final Approval) | Strategic alignment, technical feasibility, resource allocation confirmed |
| AI Governance | N/A | No | No AI capabilities in this PRD |

**Sign-off Table:**

| Role | Name | Date | Status |
|------|------|------|--------|
| Product Lead | Rakesh Rathod | — | Pending |
| Security & Compliance | Pratik Patel | — | Pending |
| Platform Architecture | Amit Patel | — | Pending |
| DevOps/Cloud Operations | Pratik Patel | — | Pending |
| CTO | Alpesh Dhamelia | — | Pending |

---
