# Phalanx DMS Setup Automation — Requirements

**Status:** Draft — pending team review
**Author:** Rick Gray (draft assisted by Claude during DMS-2304)
**Date:** 2026-05-19
**Related:** [DMS-2304 Dominion UI Smoke](2026-05-19-dms-ui-smoke-dominion-design.md), [Confluence DMS Settings Configuration Matrix](https://drivecentric.atlassian.net/wiki/spaces/DDT/pages/5782372403/DMS+Settings+Configuration+Matrix+By+Onboarding+Phase)

## Purpose

Inventory every manual step currently required to bring a Phalanx environment to a state where a DMS provider's smoke tests can run, classify each step by automation feasibility, and propose concrete automation approaches for the steps that can be automated. The result is presented for team review and priority assignment.

The motivating problem: the DMS-2304 smoke (Dominion) showed that "provision a Phalanx env and run a smoke" involves a long tail of manual steps that aren't documented in one place. Each new DMS smoke author hits the same friction. Automating the automatable steps and clearly delineating "this stays manual because X" shortens onboarding for the next eight DMS smokes.

## Scope

**In scope:** the full path from "I have a Phalanx env GUID" to "the smoke has validated this DMS works end-to-end." Includes:

- Phalanx provisioning
- Environment-specific config and fixture population
- Local secret injection
- The smoke test itself — which is *not* separable from "configuration" because we've made the deliberate decision to merge them. A single per-DMS smoke walks: configure the DMS via the WebCRM UI → create a customer + deal → push the deal to the DMS → mark the deal as delivered (the delivered deal is then called a "delivery") → pull the delivery back and verify it returned through the DMS path.

**Out of scope:**
- Anything Phalanx already provisions automatically — confirmed during DMS-2304 investigation that API keys, vendor dealer credentials, database seeding, and per-DMS environment config (outside of the user-facing matrix flags) are seeded by Phalanx provisioning itself and require no test-side intervention.
- Production DMS configuration and customer onboarding workflows.

## Classification framework

Each step is classified as one of:

- **A — Automatable now.** The step can be added to existing tooling (`setup:phalanx`, a Cypress task, a new Node script, a Terraform module, a CI workflow step) with no external dependency or policy change required. Implementation effort and proposed approach are listed.
- **B — Automatable with changes.** Automation is possible but requires a non-trivial precondition: an API the Phalanx team needs to expose, a permissions change, a new shared secret, a backend service capability we don't yet have, or a coordinated change with another team. The change required and the proposed approach are listed.
- **C — Must remain manual.** Automation is infeasible or undesirable. Justification (compliance, security, one-time setup, vendor-side action, etc.) is listed.

## Setup steps — full inventory

### Phase 1: Environment provisioning

| # | Step | Classification | Notes |
|---|---|---|---|
| 1.1 | Provision Phalanx environment via Phalanx UI (`toolshed.drivecentricops.com/phalanx2`) | **B** | Phalanx provisioning takes ~30 minutes. Provisioning API exists (`getPhalanxConfig` already calls `GET /api/deployment/get`), but no `POST /api/deployment/create` has been used from the testing repo. **Proposed approach if API exists:** new `scripts/provisionPhalanx.js` that POSTs to the create endpoint, polls for `Ready` status, prints the deployment ID. **Why not A:** 30-min provisioning makes per-PR auto-provisioning impractical regardless of API availability; better suited to a CI orchestration layer than per-test triggers. |
| 1.2 | Wait for environment status to reach `Ready` | **A** (paired with 1.1) | Polling logic is straightforward once a create API exists. |
| 1.3 | Note the deployment GUID for downstream commands | **A** (paired with 1.1) | Capture from create-API response. |

### Phase 2: Local config + fixture population

| # | Step | Classification | Notes |
|---|---|---|---|
| 2.1 | `npm run setup:phalanx -- <deployment_id>` populates `config/phalanx.json`, rewrites `usersList.v2.json`, `usersList.v2_playwright.json`, `storesList.v2.json` with cluster-specific values | **A** (already automated) | This step works today. The script is at `scripts/setupPhalanx.js`. No change needed. |
| 2.2 | `cypress.env.json` must contain test-runner secrets (`PHALANX_API_KEY`, `GMAIL_API`, `PLAYWRIGHT_ELASTICSEARCH_API_KEY`, and others) | **B** | These are test-runner-side secrets the developer's machine needs (distinct from Phalanx-internal credentials, which are already provisioned). Secrets live in 1Password; new contributors copy them manually. **Why not A:** 1Password CLI (`op read op://...`) is the right automation path but adds an external dependency, and IT/security needs to sign off on automated secret retrieval per developer machine. **Proposed approach if approved:** add `npm run setup:secrets` that uses the `op` CLI to pull the canonical `cypress.env.json` template; document the 1Password CLI install + auth step. |
| 2.3 | Cypress test users created in Phalanx env's DB via batch `admin:createUsersBatch` task | **A** (already automated via `setup:phalanx`; improvable) | Currently invoked via `npx cypress run` inside `setup:phalanx`, which forces a Cypress dependency on Playwright-only contributors and brings the Gmail OAuth dependency. **Improvement (already filed as a follow-up chip):** decouple user creation from the Cypress task system — move to a pure-Node admin API call. Classification: **A** for the existing behavior; **B** for the improved (decoupled) version, because it requires confirming the admin user-creation API surface is callable from pure Node. |

### Phase 3: The smoke test itself (configure → push → deliver → pull)

The per-DMS smoke is one combined test that walks configuration AND the deal lifecycle. Steps 3.1–3.6 are the configuration half (largely automated by DMS-2304); steps 3.7–3.11 are the lifecycle half (partially automated, partially to-be-built).

| # | Step | Classification | Notes |
|---|---|---|---|
| 3.1 | Switch the logged-in user to the test store (e.g., Store11 for the dealerSetup spec) | **B** | The Gee power user defaults to Store01. For per-DMS tests that need a specific store, a store-switcher click-through is required. **Why not A:** the WebCRM store picker isn't currently a Cypress page-object verb. **Proposed approach:** new `cy.selectStore(store)` custom command in `cypress/support/ui-commands/`, plus a method on `CommonLocators`. Implementation cost: small (one UI click flow). |
| 3.2 | Navigate to Store Settings → Early Access; reconcile the matrix-required flags per the DMS | **A** | DMS-2304's `cypress/page-objects/storeSettings/earlyAccessPage.js` already automates this against the matrix in `dmsEarlyAccessMatrix.json`. Reusable across all DMSes — each provider just needs its row in the matrix fixture. Implementation cost for next DMS: trivial (add fixture row, run the smoke against it once to confirm). |
| 3.3 | Navigate to Store Settings → DMS Settings → Save Global Settings (Confluence-flagged prerequisite) | **A** | Automated in DMS-2304's `dmsSettingsPage.saveGlobalSettings()`. Reusable. |
| 3.4 | Add DMS → select provider → fill per-provider fields | **A** | Provider-agnostic page object in DMS-2304 (`dmsSettingsPage.js`); per-provider field maps as small files (`dominionFormFields.js`). Each new DMS adds one field-map file modeled on Dominion's. |
| 3.5 | Save → Activate → handle Migrate popup | **A** | Automated in DMS-2304. |
| 3.6 | Verify configuration persists across page reload | **A** | Automated in DMS-2304. |
| 3.7 | Create a test customer and deal (API setup, not UI) | **A** | Existing helpers `CommonLocators.API().createCustomerWithDeal(...)` + `addVehicle(...)` handle this — already used by the existing `gee.pushDealToDMS.spec.js` for CDK/Reynolds/DealerTrack. Reused as-is for Dominion's push spec. |
| 3.8 | Push the deal to the DMS via the WebCRM UI (desking view → Push to DMS) | **A** | The `CustomerCard` page object exposes `.pushDeal().pushDealToDMSProcess().assertPushDealSuccess(DMS_PROVIDER)`. Already used by existing push specs; reused for Dominion. |
| 3.9 | Mark the deal as delivered (the delivered deal becomes a "delivery") | **B** | Not yet automated for the Dominion smoke. **Why not A:** the delivery-mark step's UI / API surface needs confirmation per-DMS — some providers may require a DMS-side action, others may have a direct WebCRM control. **Proposed approach:** investigate during the next push spec iteration; if it's a WebCRM UI action, add a method to `CustomerCard`/`DesckingPage`; if it's a DMS-side action, add an API helper to `apiRequestsStagingDMS.js`. **[TEAM CONFIRM]:** is the "mark as delivered" action consistently the same across all 9 DMSes, or per-DMS? |
| 3.10 | Pull the delivery back and verify it returned (the inbound side of the DMS round-trip) | **B** | Not yet automated as a UI step. Today, the existing `webAdminDmsApi.assertDMSImportKibanaLogs` (Playwright API) verifies pulls via Kibana log search — a comparable Cypress helper exists or can be added. **Why not A:** for Dominion specifically, we don't yet know whether a pull is automatic after delivery (cron-driven on the DMS side) or requires a manual trigger via WebCRM/WebAdmin. **Proposed approach:** assert via the existing Kibana log helper; if a UI trigger is needed first, capture it in the page object. **[TEAM CONFIRM]:** is the pull triggered automatically post-delivery for every DMS, or is the test expected to fire a "request pull" action? |
| 3.11 | Cleanup: remove the test customer and deal (API teardown) | **A** | `CommonLocators.API().removeCustomer(...)` handles this. Already in the existing push specs' `afterEach`. |

### Phase 4: Verification / observability prereqs

| # | Step | Classification | Notes |
|---|---|---|---|
| 4.1 | `PLAYWRIGHT_ELASTICSEARCH_API_KEY` populated in `cypress.env.json` (for tests that assert via Kibana logs, e.g., the push/pull verification in 3.10) | **B** | Same situation as 2.2 — test-runner-side secret injection. |
| 4.2 | Phalanx env's Elasticsearch endpoint reachable from the test runner | **A** (already automated) | `setup:phalanx` writes `ELASTICSEARCH_ENDPOINT` into `config/phalanx.json`. |

## Prioritization recommendation (for team to confirm / reorder)

Order of implementation, ranked by [impact × ease]:

1. **Decouple Phalanx setup from Cypress** (Phase 2.3 improvement). Already flagged as a chip during DMS-2304. Small change with a meaningful ergonomics win for Playwright-only contributors and removes the Gmail-OAuth-secret dependency from the setup path.
2. **Store-switcher Cypress command** (Phase 3.1). Unblocks the "test against any specific Gee store" requirement for all DMS smokes. Small implementation; high reuse.
3. **Complete the deal-lifecycle half of the Dominion smoke** (Phases 3.9, 3.10). Mark-as-delivered + pull verification are the gap between "Dominion is configured" and "Dominion round-trips end-to-end." Until these land, the per-DMS smoke isn't a true smoke. Investigate during the next Dominion push spec iteration.
4. **Build out per-DMS fixtures + field maps** (Phases 3.2, 3.4 applied to each remaining DMS). Mechanical work — each DMS is roughly the same effort as Dominion was. Track as one ticket per DMS or one umbrella ticket with sub-tasks. Should land after #3 so each new DMS smoke covers the full lifecycle from day one.
5. **Secret injection automation via 1Password CLI** (Phases 2.2 / 4.1). Needs IT/security sign-off; impact is per-developer one-time so not urgent, but reduces friction for every new contributor.
6. **Phalanx auto-provisioning** (Phase 1.1). Lowest priority despite the biggest single time cost (~30 min) because the provisioning latency itself, not the automation gap, is the blocker for PR-gated smokes. Worth doing only after the provisioning latency is addressed on the Phalanx side, or if a "warm pool" of pre-provisioned envs becomes available.

## Review process

1. **Author** (Rick) circulates this draft to the Testability and DMS teams via the `#testability-guild` Slack channel.
2. **Reviewers** push back on classifications they disagree with, confirm `[TEAM CONFIRM]` items, and surface any setup steps this inventory missed. Two-week comment window.
3. **Resolution meeting** (≤30 min, only if open comments remain after the window): walk through unresolved items, assign owners.
4. **Final priorities** captured as Jira tickets in the DMS or TAFP project, linked back to this doc.
5. **This doc becomes the canonical reference** for the next DMS smoke author and lives at this path (or moved to Confluence if preferred — see Open question 1).

## Open questions

1. **Should this doc's final form be in Confluence?** The DMS Settings Configuration Matrix lives on Confluence; this requirements doc may make more sense there too for visibility outside the test repo.
2. **What's the actual Phalanx provisioning API surface?** Confirm with the Phalanx team whether create/destroy endpoints exist and are usable from the testing context.
3. **Is 1Password CLI an acceptable secret-injection mechanism** per IT/security policy?
4. **Did this inventory miss anything?** Reviewers should flag any setup step they perform manually today that isn't captured above.

## `[TEAM CONFIRM]` index

For convenience, all items that need team input:

- **2.2 / 4.1:** 1Password CLI as secret injection mechanism — IT/security sign-off.
- **2.3 (improved):** admin user-creation API callable from pure Node — confirm endpoint exists and is safe to call without the Cypress task layer.
- **3.9:** is "mark deal as delivered" the same action across all 9 DMSes, or per-DMS?
- **3.10:** is the pull triggered automatically post-delivery for every DMS, or is the test expected to fire a "request pull" action?
- **Open questions 1–4 above.**