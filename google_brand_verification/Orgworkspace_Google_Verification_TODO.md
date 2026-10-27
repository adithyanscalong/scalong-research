# Orgworkspace — Google OAuth Verification TODO

Updated: 26 October 2026

Goal: remove Google's unverified-app warning for customers connecting approved Google permissions.

Status: **Have** = confirmed by user/screenshots; **Missing** = confirmed absent; **Unknown** = needs evidence; **Pending** = future validation. Unchecked items do not imply confirmed absence.

| Done | Requirement / next action | Current status | Suggested owner |
|---|---|---|---|
| [x] | Application/login page exists | Have | Product |
| [x] | Gmail authentication completes after bypassing warning | Have; API/refresh behavior not verified | Backend |
| [ ] | Create public homepage explaining Orgworkspace and Google connectors | Missing | Website/Product |
| [ ] | Publish accurate privacy policy on homepage domain; link same URL in homepage and OAuth branding | Unknown | Product/Legal |
| [ ] | Confirm consistent app name/logo, Google buttons, support and monitored contacts | Unknown | Product/Cloud |
| [ ] | Verify authorized-domain ownership through project Owner/Editor | Unknown | Cloud/DNS owner |
| [ ] | Confirm production project/client, External audience, redirects and enabled APIs | Unknown | Cloud/Backend |
| [ ] | Audit exact Gmail/Drive/Sheets scopes; minimize and justify each | Unknown | Backend |
| [ ] | Confirm permitted use, Limited Use statement, consent and AI/external destinations | Unknown | Platform/Legal |
| [ ] | Validate encryption, tenant isolation, prompt injection protection, disconnect and deletion; publish data-management help | Unknown | Security/Backend |
| [ ] | Determine CASA applicability and complete assessment if required | Unknown; conditional | Security |
| [ ] | Prepare demo video covering full English consent and actual scope-dependent features | Unknown | Product/QA |
| [ ] | Complete applicable branding and sensitive/restricted-scope review | Exact status unknown; warning observed | Cloud owner |
| [ ] | Test fresh external account without warning and verify connector actions/refresh | Pending approval | QA |
| [ ] | Maintain contacts, re-verification and assessment renewals as applicable | Pending | Cloud/Security |

First: identify the Cloud owner, production client IDs and exact scope list while the homepage is prepared. Terms of service are recommended, rather than a universal OAuth requirement. Publishing to production alone does not verify an app.

Detailed instructions: [production verification guide](Orgworkspace_Google_OAuth_Production_Verification_Guide.md).

Missing context: [questions for seniors](Orgworkspace_Google_Verification_Questions_For_Seniors.md).

Official requirements: [Google verification requirements](https://support.google.com/cloud/answer/13464321), [restricted scopes](https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification), [Workspace data policy](https://developers.google.com/workspace/workspace-api-user-data-developer-policy).
