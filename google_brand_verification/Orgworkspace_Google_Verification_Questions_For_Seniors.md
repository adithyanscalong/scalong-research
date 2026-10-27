# Orgworkspace: Google verification questions for seniors

Prepared: 23 October 2026

Updated: 26 October 2026

## Objective and confirmed context

Remove the Google OAuth unverified-app warnings when business clients connect Google products to Orgworkspace automations and agents.

- Application/login URL: https://www.orgworkspace.com/. Latest user clarification: no qualifying public product homepage exists; creation is pending.
- Connectors identified: Gmail, Drive, and Sheets.
- Screenshots show both the full-page “Google hasn’t verified this app” warning and an unverified-app notice within the additional-access consent screen.
- The developer contact displayed is docqdev@scalong.com. This is a useful starting contact; project ownership is not yet confirmed.
- The blue six-month Gmail access-review notice is a separate issue from app verification. Selecting ongoing access does not verify the app.
- Exact scopes, project configuration, architecture, and review history remain unconfirmed.

## Message to forward

We need to complete the applicable Google OAuth verification for Orgworkspace so customers no longer see the unverified-app warnings. Please help answer the questions below or identify the person who can. Screenshots of configuration and exact scope lists are useful; please do not send client secrets, passwords, or access/refresh tokens.

### 1. Google Cloud project and verification status — Cloud owner / backend lead

1. Which Google Cloud project and OAuth client IDs does the live Orgworkspace app use for Gmail, Drive, and Sheets? Are clients shared across connectors or separate? Are development and production in separate projects?
2. Who owns/administers that project, and who monitors docqdev@scalong.com and verification emails?
3. In Google Auth Platform, what are the current Branding verification/publication status, Audience type (Internal/External), publishing status (Testing/In production), and Verification Center requirements?
4. Have we already submitted brand or scope verification? Please share any approval, rejection, pending-action, or security-assessment emails and existing demo-video links.
5. Are customers expected to use personal Gmail accounts as well as organizational Google Workspace accounts?

### 2. Exact permissions and implemented features — connector / backend engineer

6. Please export the complete scope strings from Google Auth Platform → Data Access, and confirm the scopes actually sent by production OAuth authorization requests for each connector. Are permissions requested together or separately as features are connected?
7. For Gmail, do implemented workflows send messages, read bodies/attachments, search mail, manage drafts, or modify/delete messages? Which actions are essential today?
8. For Drive, do workflows access only files selected/shared by the user, or search/read/manage the entire Drive? For Sheets, do they read, edit, create, or delete spreadsheets?
9. For each scope, which implemented feature requires it, and could a narrower scope support that feature? Please separate live features from planned features.
10. Are we using our own shared OAuth credentials, a third-party connector platform's credentials, or OAuth credentials supplied by each customer? What are the production redirect URLs?

### 3. Google data flow and retention — platform / AI / security lead

11. Where does Google data travel after authorization? Please describe: Google API → connector/backend → agent/workflow → AI provider or external destination → storage/logs.
12. Which data is read or retained, including email bodies, attachments, document contents, spreadsheet cells, metadata, workflow history, caches, traces, and embeddings? Where is it hosted and for how long?
13. Which LLM providers and other subprocessors receive Google data? Can customer workflows send it to arbitrary webhooks, external APIs, email recipients, or customer-selected AI providers?
14. Is Google data used for model training or fine-tuning by us or any downstream provider? What contractual/settings evidence confirms the answer?
15. How do users see and approve data destinations and automated actions? Can agents act unattended, and what controls exist for sending emails or changing/deleting data?
16. How are OAuth tokens protected and isolated across customers? What happens to tokens and retained data when an account is disconnected or a deletion request is made, including downstream copies?

### 4. Website, domain, and submission readiness — product / website / legal owner

17. What are the live privacy-policy, terms, and support URLs? Do the privacy policy and in-product disclosures accurately describe Google data use, AI sharing, retention, deletion, and revocation?
18. Who controls orgworkspace.com, its website, and DNS? Has ownership of every OAuth authorized domain been verified in Search Console by an account that is also a project Owner/Editor?
19. What exact app name, logo, homepage, support email, developer contacts, and authorized domains are configured in Google Auth Platform? Are they consistent with the public product?
20. Who can prepare a working reviewer account, representative workflows, per-scope justifications, and a demo showing consent and actual use of every requested sensitive/restricted scope?
21. If restricted scopes and the data architecture require a security assessment, do we already have an applicable assessment? If not, who owns the budget and assessor coordination?

## First answers needed

Start with questions 1–6 and 10. These establish which production app is requesting access, its status, and the permissions involved. Questions 7–21 establish the scope justifications and readiness for the applicable review.

Brand approval alone should not be assumed to remove these warnings. The requested production scopes and applicable review requirements must also be confirmed.

## Official references

- Verification requirements: https://support.google.com/cloud/answer/13464321
- Brand verification: https://developers.google.com/identity/protocols/oauth2/production-readiness/brand-verification
- Restricted-scope verification: https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification
