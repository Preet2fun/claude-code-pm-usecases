# Guided Onboarding - Impact Estimation

**Feature:** Guided Onboarding with Sample Project
**Date:** November 26, 2025
**Prepared by:** Senior PM, Activation

---

## Current State

**Funnel Performance:**
- **5,000 new signups/month** (current rate based on Q4 data)
- **45% activation rate** (users who complete first task)
- **2,250 activated users/month** (5,000 × 45%)
- **45 minutes median time-to-first-completed-task** (current friction point)

**Key Problem:**
60% of users who create a task never complete it. Survey data shows they're overwhelmed by the blank canvas and don't know what to create.

---

## Projected Impact (Realistic Scenario)

### Users Affected
**3,500 users/month** (70% of signups)

**Rationale:**
- Gradual rollout to new signups only (not existing users)
- Assume 70% adoption based on similar onboarding features
- Some users will skip or dismiss the guided onboarding
- Enterprise users may bypass it (have their own workflows)

### Expected Lift
**45% → 58% activation** (+13 percentage points)

**Reasoning:**
Survey data shows 60% of drop-offs are due to confusion about what to create. If we:
- Provide clear examples (eliminates "don't know what to create" - 35% of responses)
- Show templates (eliminates "needed examples" - 28% of responses)
- Remove blank canvas anxiety (eliminates "felt overwhelmed" - 22% of responses)

We can conservatively recover **30% of the 60% drop-off**:
- 60% drop-off = 4,320 users lost (out of 7,200 who create tasks)
- Recover 30% = +1,296 users
- New activation: 2,880 + 1,296 = 4,176 out of 7,200 = 58%

**Conservative estimate:** 58% activation (not 63%) to account for implementation imperfections and user behavior variability.

### Business Impact Calculation

**Current activated users (among those who see feature):**
- 3,500 users/month × 45% = **1,575 activated/month**

**Projected activated users:**
- 3,500 users/month × 58% = **2,030 activated/month**

**Incremental impact:**
- **+455 activated users/month**

### Revenue Projection

**Value per activated user:**
- $12/month ARPU × 60% paid conversion × 24 month lifetime = **$172.80 LTV**

**Monthly recurring revenue impact:**
- 455 new activated users/month × $12 ARPU × 60% conversion = **$3,276 MRR**

**Annual recurring revenue impact:**
- $3,276 MRR × 12 months = **$39,312 ARR**

**3-year lifetime value:**
- 455 activated users/month × 12 months = 5,460 incremental activated users/year
- 5,460 users × $172.80 LTV = **$943,488 in total LTV**

---

## Investment Required

**Engineering effort:**
- Backend: Pre-populate sample project logic, API updates (2 eng-weeks)
- Frontend: Sample project UI, dismissal flow, mobile optimization (6 eng-weeks)
- QA: Testing across user types, edge cases (2 eng-weeks)
- **Total: 10 eng-weeks** = **$100,000** (fully-loaded cost)

**Design effort:**
- Sample project design, example task content (1 week)

**PM effort:**
- Spec, coordination, launch (included in role)

---

## ROI Analysis

### Year 1
- **Revenue:** $39,312 ARR
- **Cost:** $100,000
- **ROI:** 0.39x (does not break even in Year 1)

### 3-Year LTV
- **Total value:** $943,488
- **Cost:** $100,000
- **ROI:** 9.4x 🎯

### Payback Period
- Break-even at ~30 months (2.5 years)
- Positive cash flow thereafter

---

## Additional Benefits (Not Modeled)

1. **Retention improvement:** Activated users stay 2.5x longer than non-activated users (historical data). Better activation = higher LTV than modeled.

2. **Viral growth:** Activated users are 3x more likely to invite teammates. More activations = more organic growth.

3. **Strategic unlock:** Can't scale beyond 5,000 signups/month if activation stays at 45%. This feature unlocks future growth.

4. **Support reduction:** Users who understand the product require less support. Estimated 15% reduction in onboarding-related tickets.

---

## Assumptions & Risks

**Key Assumptions:**
1. **70% adoption** - Based on similar onboarding features, but could be lower if users skip it
2. **58% activation** - Conservative recovery estimate, but actual lift could vary
3. **No cannibalization** - Assumes this doesn't pull resources from other growth initiatives
4. **Small teams benefit most** - Survey data shows they're most affected by blank canvas

**Risks:**
1. **Lower adoption than expected** - Users might dismiss the guided onboarding
2. **Feature complexity** - Building sample project logic could take longer than estimated
3. **Enterprise users confused** - Sample tasks might feel too basic for complex workflows
4. **Retention not improved** - Activation doesn't guarantee engagement

**Mitigation:**
- Start with small teams (5-20 people) where problem is most acute
- Make guided onboarding dismissible but re-accessible
- A/B test to validate assumptions before full rollout
- Build analytics to track feature engagement and iterate

---

## Recommendation

**Ship it.** The realistic scenario shows 9.4x ROI over 3 years for a relatively small investment ($100k). Even if we're off by 50%, this is still a 4-5x return.

More importantly, **this unblocks growth**. We can't scale if nearly half of new users don't activate. This feature addresses the root cause (blank canvas overwhelm) with a proven solution (examples and templates).

**Next steps:**
1. Build ROI scenarios (pessimistic/realistic/optimistic) to show range of outcomes
2. Get engineering estimate validation (is 10 weeks realistic?)
3. Design sample project content (what tasks should we include?)
4. Plan A/B test structure (control vs. treatment, metrics to track)

---

**Prepared by:** Senior PM, Activation | **Reviewed by:** TBD
