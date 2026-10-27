# Orgworkspace — Google OAuth Production Verification Guide

**Purpose:** Get Orgworkspace's Google connectors (Gmail, Drive, Calendar, Sheets) approved and reliable for external production users.  
**Updated and official references checked:** 26 October 2026  
**Status:** Implementation checklist; actual OAuth scopes, data flows, and approval status must be confirmed by engineering.

## Current Orgworkspace status

- User-confirmed: the application/login page exists and Gmail authentication succeeds after bypassing Google's warning. Successful API calls and token refresh are not yet verified.
- User's latest clarification: **no qualifying public product homepage exists**. Create one; earlier discussion about a Scalong homepage must not be treated as evidence that this requirement is satisfied.
- Identified connectors: Gmail, Drive, and Sheets. Calendar below is an optional example, not a confirmed implemented connector.
- Screenshots confirm the unverified warning. Exact Cloud verification status, requested scopes, privacy policy, domain verification, security controls, demo and assessment status remain unknown.
- Track work in [verification TODO](Orgworkspace_Google_Verification_TODO.md) and obtain missing details using [questions for seniors](Orgworkspace_Google_Verification_Questions_For_Seniors.md).
- This is a readiness guide, not evidence of approval or a completed implementation audit. Google may request additional evidence for the actual scopes and architecture.

## 1. What the screenshots mean

Google displays **“This app hasn't been verified”** because Orgworkspace is requesting Google account permissions that have not completed the applicable OAuth verification. The six-month Gmail access-review notice is a **separate consent/access-duration issue**; selecting ongoing access does **not** verify the app or repair connector errors.

**Three distinct outcomes:**
1. **Verified OAuth app:** Google approves branding and requested scopes.
2. **Working connector:** OAuth callback, token storage/refresh, API calls, and permissions work correctly.
3. **Production-ready automation platform:** User consent, AI data transfers, security, tenant isolation, and data deletion are compliant.

You need all three.

## 2. Decide the verification path

Check Google's documented verification exceptions first. Serving business customers outside our own Workspace organization does not by itself qualify for the internal-only exception. Record any claimed exception and its supporting evidence.

| What Orgworkspace requests | Typical requirement |
|---|---|
| Basic identity / non-sensitive scopes | Branding and basic OAuth compliance |
| Sensitive scopes | Branding + sensitive-scope review |
| Restricted scopes | Branding + restricted-scope review; security assessment if restricted data passes through/gets stored on your servers |

**Examples (verify the actual requested scopes):**

| Feature | OAuth scope suffix | Google classification |
|---|---|---|
| Gmail send | `gmail.send` | Sensitive |
| Gmail read | `gmail.readonly` | Restricted |
| Gmail read/change | `gmail.modify` | Restricted |
| Gmail draft management | `gmail.compose` | Restricted |
| Drive files explicitly selected for the app | `drive.file` | Non-sensitive |
| Drive read all files | `drive.readonly` | Restricted |
| Drive full access | `drive` | Restricted |
| Calendar read | `calendar.readonly` | Sensitive |
| Sheets read | `spreadsheets.readonly` | Sensitive |
| Sheets read/edit/create/delete | `spreadsheets` | Sensitive |

The Sheets API also supports `drive.file` for files used with the app; consider this narrower alternative where it supports the implemented feature. Sheets scopes apply to spreadsheet files, not individual tabs. See [Sheets scope documentation](https://developers.google.com/workspace/sheets/api/scopes).

Full scope strings normally begin with `https://www.googleapis.com/auth/`. **Do not request broader scopes simply to support hypothetical future workflows.** Confirm other scopes (Sheets, Contacts, etc.) against their official API scope documentation.

**Key decision:** If Orgworkspace needs to ingest full Gmail messages and run them through server-side agents, plan for restricted-scope review **and** a likely annual independent security assessment. First establish that your proposed Gmail use cases are permitted under Google's Gmail/Workspace developer policies.

## 3. Exact production setup (Google Cloud)

1. Open [Google Cloud Console](https://console.cloud.google.com/) and select/create a **separate production project**. Keep development/testing OAuth clients in a separate project.
2. Open [Google Auth Platform](https://console.cloud.google.com/auth/overview).
3. **Branding:** enter exact public app name, support email, logo (if used), homepage, privacy-policy URL, and developer contact emails.
4. **Audience:** select **External** if users outside your Google Workspace organization, including personal Gmail users, must connect. Internal is only for eligible organizational users.
5. **Clients:** create a Web application OAuth client; configure **exact** HTTPS redirect URI(s) and JavaScript origins where applicable. Store client secret only on the backend.
6. **APIs & Services → Library:** enable Gmail, Drive, Calendar, Sheets APIs actually used.
7. **Data Access:** add the exact scopes requested by the code; remove obsolete or excessive scopes.
8. Verify domain ownership in [Google Search Console](https://search.google.com/search-console/) and ensure authorized domains, homepage, privacy policy, and redirect URIs are consistent.
9. Add correct project Owners/Editors and monitored verification contacts.

**Engineering check:** Search frontend/backend code and OAuth authorization URLs for every scope. A scope missing from the consent configuration, or a different OAuth client being used at runtime, can derail verification.

## 4. Website, privacy and product readiness

Before submission, publish:

- **Public homepage:** clear description of Orgworkspace and its Google connector/agent capabilities; no login required to understand the product.
- **Privacy policy:** which Google data is collected, why, where stored, retention/deletion, sharing with subprocessors and LLMs, user choices, revocation, and contact information.
- **Terms of service:** recommended for transparency and enterprise readiness.
- **User-facing controls:** connect/disconnect accounts; show granted access; provide clear workflow-level data destinations and action permissions.
- **Support contact:** reachable and consistent with branding.

Privacy statements must match actual backend behavior. Do not claim “no data stored” if messages, workflow logs, embeddings, caches, or traces retain content.

Website acceptance checks:

- Host the privacy policy within the homepage's domain, link it visibly from the homepage, and configure the identical policy URL in OAuth branding.
- A Scalong product page may link to an Orgworkspace application on another domain if the product relationship is clear. This is not automatic approval: identify Orgworkspace clearly, verify all authorized domains, and use consistent branding. A generic company page or redirect to login does not substitute for product information.
- A project Owner/Editor must verify ownership of authorized domains through Search Console. Follow the Google branding guidelines for Google connection/sign-in buttons.
- Provide user help explaining how to manage and delete data. Publish an affirmative statement that Google Workspace data use complies with Google's User Data Policy, including Limited Use restrictions; ensure the statement matches actual behavior.
- Terms of service are recommended; they are not a universal prerequisite for every OAuth verification request.

## 5. Special requirements for an n8n-style AI agent platform

Google reviews **actual data movement**, not just connector names.

Map these paths for every Google-enabled agent:

`Google account → Google API → Orgworkspace connector → workflow/agent → LLM provider / external app → output, logs, storage`

Document:

- Which user initiates/authorizes the workflow; which Google account is connected.
- What data is read, modified, sent, or retained, including attachments and metadata.
- Whether the workflow can send Google data to **arbitrary webhooks, APIs, email addresses, or third-party LLMs**.
- Which subprocessors receive Google data and on what terms.
- Whether workflows run unattended; what controls exist for high-impact actions.
- How consent, disclosure, revocation, retention and deletion work across downstream systems.
- Whether Google data is used for **model training**; do not assume ordinary OAuth consent authorizes training.

**Important:** Google has permitted-use and Limited Use rules. Being a general-purpose automation builder does **not** mean unrestricted Gmail-to-any-destination transfers are automatically approved. Review the [Google Workspace API User Data and Developer Policy](https://developers.google.com/workspace/workspace-api-user-data-developer-policy) and [verification requirements](https://support.google.com/cloud/answer/13464321) with engineering/legal before submission.

**Practical architecture improvements:** Use incremental, connector-specific consent; prefer `gmail.send` over mailbox-wide permissions for send-only workflows; prefer `drive.file` with a file picker over full-Drive access where feasible; make external data destinations visible and controllable; isolate each tenant's credentials and data.

Policy checks that must be explicit in the audit:

- Disclosures must clearly explain the actual data access and transfers. Where additional prominent disclosure is required, show it immediately before affirmative consent and before collecting the data; a privacy-policy link alone is insufficient.
- Limited Use covers derived data too. Prohibited uses include selling data and advertising-related uses. Restrict human access to Google's allowed exceptions and document applicable consent.
- Do not use or transfer Workspace data to train or improve general-purpose/shared AI models. The policy's specific-user personalized-model allowance is narrow; ordinary OAuth consent does not establish compliance. Check provider contracts/settings and permitted purpose before relying on it.
- Review storage, caching, embeddings and permanent-copy behavior against applicable API terms and cache restrictions, not just the retention period selected by the team.

## 6. Security assessment preparation (restricted scopes)

If restricted Google data is accessed from or transmitted through Orgworkspace servers, Google generally requires an **independent, Google-approved security assessment** using the CASA framework. Plan for periodic reassessment (typically at least every 12 months).

Prepare evidence of:

- Encrypted tokens at rest/in transit; secrets manager; least-privilege service access.
- Secure OAuth authorization-code flow, `state` validation, and PKCE where appropriate.
- Tenant/workspace isolation and role-based access control.
- Audit trails for agent execution, credential use, data access, and exports.
- Secure logs (avoid leaking email bodies, tokens, attachments).
- Data minimization, retention schedules, account disconnect, deletion workflows.
- Dependency scanning, vulnerability management, incident response, and backups.
- Third-party AI/service data-handling contracts and restrictions.
- Prompt injection protection for agents handling untrusted Google content; encryption of Google data and credentials at rest, modern secure transport, and appropriate key management.
- An incident-response process that meets Google's current notification requirements; assign a security owner to check the applicable policy deadlines and reporting channel.

The assessor will specify the actual testing scope and remediation requirements. **Do not treat this list as a substitute for the assessment.**

## 7. Verification demonstration video — what to record

Record the real Orgworkspace production-like app, with consistent branding:

1. Show the homepage and the user-facing Google connector setup.
   If the homepage is on Scalong and the app is on Orgworkspace, show that navigation and explain that Orgworkspace is a Scalong product. Keep submitted app identity consistent throughout.
2. Start Google OAuth; show the **entire consent screen in English**, including requested permissions.
3. Connect a Google account and return successfully to Orgworkspace.
4. Build and execute **multiple representative workflows**, for example:
   - Gmail → AI summary → approved destination.
   - User-selected Drive file → document extraction → agent output.
   - Calendar read → agent → Gmail send.
   For the confirmed Sheets connector, show a real implemented spreadsheet read/write workflow instead of an unimplemented example. Multiple workflows are useful when needed to cover scopes; Google requires coverage of actual scope-dependent functionality, not a fixed number of workflows.
5. Show **direction of data flow**, destination disclosures, and relevant user consent.
6. Show actual functionality requiring each sensitive/restricted scope; explain why narrower scopes do not suffice.
7. If applicable, demonstrate **separate explicit consent for permitted personalized AI training**.

Use test accounts and non-sensitive test data. Upload the recording as **Unlisted** on YouTube and supply the link in the verification submission. Do not demonstrate features that are not implemented.

## 8. Submit for verification — in order

1. Go to [Google Auth Platform → Branding](https://console.cloud.google.com/auth/branding).
2. Complete branding information, verify your domain, run **Verify Branding**, resolve findings, and **Publish branding** when approved.
3. Go to [Data Access](https://console.cloud.google.com/auth/scopes) and confirm all production scopes are declared.
4. Open the [OAuth Verification Center](https://console.cloud.google.com/auth/verification).
5. Submit the sensitive/restricted scope review with:
   - Detailed feature/use-case description.
   - **Per-scope justification** and why narrower alternatives are insufficient.
   - Public feature documentation and privacy policy.
   - Demo video and test/reviewer access if requested.
   - Explanation of data handling and transfers.
6. Respond promptly to Google review emails; correct scopes, product behavior, policy text, or videos if requested.
7. If Google requires a CASA security assessment, engage an approved assessor, fix findings, and provide the resulting assessment evidence.
8. Confirm approval for **every scope used in production**, set/maintain the appropriate **In production** publishing status, and test the full OAuth and connector flows with external accounts.

Before submission, record project ID, runtime client IDs, requested scope strings, each scope's classification, feature justification, narrower-alternative analysis, demo timestamp, data destinations, and owner. Supply reviewer instructions/access if requested without exposing production secrets or customer data. Follow the current Verification Center flow; sensitive-scope review can include brand verification as a subset rather than always requiring two separate submissions.

After approval, maintain monitored contacts and any applicable annual assessment (at least every 12 months from assessment approval for covered apps). Branding edits may require verification and publishing again; new sensitive/restricted scopes require the applicable review before relying on their approval. Workspace administrators may still restrict access independently of verification.

**Warning:** Switching the app to “In production” by itself does **not** make it verified.

## 9. Production connector testing checklist

Test with a new external Google account after approval:

- [ ] Consent screen shows correct app identity and scopes, without unverified warning for approved scopes.
- [ ] OAuth redirect returns to the right workspace/user.
- [ ] Backend securely exchanges authorization code for tokens.
- [ ] Only granted scopes are assumed; handle denied permissions gracefully.
- [ ] Gmail read/send, Drive file access, Calendar/Sheets calls work with intended scopes.
- [ ] Refresh token rotation/storage and expiration/revocation are handled.
- [ ] Disconnection revokes access where appropriate and triggers retention/deletion processes.
- [ ] No cross-tenant data or credential access.
- [ ] Agent failures expose actionable errors without exposing secrets.
- [ ] OAuth errors are logged safely (`redirect_uri_mismatch`, `access_denied`, `invalid_grant`, `insufficientPermissions`, API disabled, etc.).

**Testing-mode trap:** For External apps in Testing, refresh tokens involving non-basic identity scopes commonly expire after **7 days**. This is distinct from Google's six-month Gmail access-review message. Production verification does not mean tokens can never expire or be revoked.

## 10. What to do this week (recommended order)

| Priority | Action | Owner |
|---|---|---|
| P0 | Export exact Google OAuth scopes and identify runtime client IDs | Backend engineer |
| P0 | Decide Gmail/Drive minimum scopes and check permitted use cases | Tech lead + compliance |
| P0 | Draw Google → agent → LLM/API data-flow map | Agent/platform engineer |
| P1 | Audit website, privacy policy, data deletion, third parties | Product + legal |
| P1 | Verify domain and production OAuth configuration | Cloud/backend engineer |
| P1 | Prepare demo workflows and video | Product + QA |
| P1 | Submit branding then scope verification | Cloud project owner |
| P1 | Arrange CASA assessment if applicable | Security lead |
| P2 | Perform end-to-end production acceptance tests | QA + backend |

## 11. Common blockers and fixes

| Problem | Likely next step |
|---|---|
| Unverified warning | Check publishing/verification status and exact requested scopes; submit review |
| `redirect_uri_mismatch` | Match the redirect URI exactly in OAuth client settings |
| User cannot connect in Testing | Add test user; inspect account/admin restrictions |
| Connector works then stops | Check refresh-token expiration/revocation and `invalid_grant` |
| Scope review rejected | Reduce scope, strengthen justification, demonstrate real feature |
| Security review required | Assess restricted data path; engage approved assessor |
| Google objects to AI transfers | Review Limited Use, permitted purpose, consent, downstream recipients, and architecture |

## 12. Official references

1. [Google — Verification requirements](https://support.google.com/cloud/answer/13464321)
2. [Google — Restricted scope verification](https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification)
3. [Google — Sensitive scope verification](https://developers.google.com/identity/protocols/oauth2/production-readiness/sensitive-scope-verification)
4. [Google — Production OAuth policy compliance](https://developers.google.com/identity/protocols/oauth2/production-readiness/policy-compliance)
5. [Google — Gmail scopes](https://developers.google.com/workspace/gmail/api/auth/scopes)
6. [Google — Drive scopes](https://developers.google.com/workspace/drive/api/guides/api-specific-auth)
7. [Google — Workspace API User Data and Developer Policy](https://developers.google.com/workspace/workspace-api-user-data-developer-policy)
8. [Google — OAuth token behavior](https://developers.google.com/identity/protocols/oauth2/)
9. [Google — Brand verification](https://developers.google.com/identity/protocols/oauth2/production-readiness/brand-verification)
10. [Google — Sheets scopes](https://developers.google.com/workspace/sheets/api/scopes)

## 13. Documentation audit — 26 October 2026

The existing guide covered branding, scope minimization, Cloud setup, AI data flows, assessment preparation, demo evidence, submission and acceptance testing. This revision adds current confirmed/unknown status, homepage absence, same-domain privacy-policy placement, cross-domain navigation guidance, Sheets scopes, exceptions, Limited Use disclosure, human-access/training restrictions, prompt injection protection, reviewer evidence and post-approval maintenance.

Coverage is sufficient as a working preparation guide for the identified Gmail/Drive/Sheets use case. Readiness is **not confirmed** until engineering supplies the actual scopes/configuration and the team validates implementation against current official policies. Scope-specific requirements and Google's reviewer feedback take precedence over this checklist.

---

### Definition of done

**Orgworkspace is production-ready when:** all requested production Google scopes are approved; any required security assessment is complete; real user-facing data flows meet Google policy; external users can connect without the unverified-app warning for those scopes; and connector/agent executions reliably handle tokens, permissions, revocation, and user data.
