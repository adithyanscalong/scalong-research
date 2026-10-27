# WhatsApp Connector Study — OrgWorkspace

**Reviewed:** 22 October 2026  
**Status updated:** 26 October 2026  
**Status:** Research ongoing; no integration approach, migration, pilot, or code fixes have been approved.  
**Pending clarification:** Why is personal WhatsApp specifically needed, which capabilities are essential, and what is its priority?

## 1. Executive assessment

- Personal-account automation works technically through an unofficial linked-device client. It carries unresolved authorization, account-restriction, compatibility, and data-protection risks.
- The official WhatsApp Business Platform / Cloud API is an alternative under evaluation. It does not provide unrestricted access to personal conversations; suitability depends on the use case still being clarified.
- An official integration still needs tenant isolation, secure credential handling, retention controls, and authorization outside the AI model.
- OrgWorkspace's exact AI use case must be checked against current platform terms. Official API availability does not establish eligibility for every AI product.
- Treat the pricing below as a planning estimate, not a verified Meta quotation: current public sources conflict about service-message charges.

## 2. Scope and evidence

**Inspected:** local `main.go`; upstream Go bridge, Python MCP tool definitions, Python database/API implementation, and dependency manifests in [lharries/whatsapp-mcp](https://github.com/lharries/whatsapp-mcp).

**Unavailable locally:** other Go files, `go.mod`, lockfiles, deployment configuration, gateway code, production databases, and access policies. No runtime or penetration testing was performed; production exposure is not established by this static review.

Evidence labels:

- **Verified:** directly observed in code or an accessible authoritative source.
- **Conditional:** impact depends on deployment or missing implementation.
- **Provisional:** published figures that could not be independently confirmed against the effective Meta documentation/account configuration.

## 3. How the existing bridge works

```text
Phone scans pairing QR
  → whatsmeow connects as a linked device
  → Go process receives decrypted messages/events
      → store/whatsapp.db: linked-device identity and protocol/session state
      → store/messages.db: messages, chats, media metadata, call metadata
      → store/<chat>/...: downloaded media
      → webhook helpers: forward data to application workflows

Application/MCP tools → local REST API → WhatsApp actions
```

The production variant loads the first stored device, reuses its session, supports pairing, and attempts reconnection. It adds authentication hooks, webhooks, reactions, read receipts, typing, call metadata, and history handling beyond upstream.

Upstream Python tools search local SQLite data and invoke the Go API for actions. **MCP is the tool interface; it does not supply WhatsApp authorization, consent, or tenant isolation.**

History synchronization is best effort. The full-history pairing flag requests 3,650 days and 102,400 MB; these are requested limits, not a guarantee that WhatsApp returns that history.

## 4. Production code findings

Line references refer to the supplied `main.go` snapshot.

These are research findings and possible mitigations, not approved fixes. Action-oriented suggestions below describe options for future evaluation.

| Priority | Verified observation | Potential mitigation for evaluation |
|---|---|---|
| High | Revokes stamp `deleted_at` while deliberately preserving content; `GetMessages` does not exclude it (`858–878`). | Define deletion semantics; remove inaccessible content from retrieval, media, indexes, embeddings, and downstream copies. |
| High | Live and historical message bodies are logged (`2036–2040`, `3566–3567`, `3639–3644`). | Remove content logging; assess existing logs, access, and retention. |
| High | Fixed storage paths, first-device selection, and no tenant ownership columns (`106–181`, `2830–2841`). | Treat as a single-account worker. Prove account/workspace routing and isolate volumes and identities. |
| High | Ordinary SQLite connections and plaintext media writes; no application-level encryption visible (`114`, `2241`, `2830`). | Protect sessions, content, backups, and keys. External disk encryption/ACLs remain unverified. |
| High | Disappearing-message metadata is handled, but no local expiry worker appears. | Implement retention and expiry across all retained copies. |
| High, conditional | `/qr` bypasses the API authentication wrapper and loads CDN JavaScript (`2303–2321`). | Authenticate and bind pairing to the intended workspace/account; bundle QR assets locally. Loopback binding reduces exposure but does not replace these controls. |
| High, conditional | The bridge token is also assigned as the outbound webhook credential (`2927–2940`). | Separate command and event-delivery credentials; verify receiver and rotation behavior. |
| Medium–High | Automatic media downloads use synchronous image work or background goroutines without visible concurrency/byte quotas (`1961–2006`). | Add bounded workers, size limits, disk quotas, deadlines, and attachment inspection. |
| Medium–High | No visible send idempotency key or transactional event outbox. | Prevent duplicate sends and provide durable, deduplicated event delivery. |

**Existing protections:** REST binds to `127.0.0.1`; action routes invoke `withAuth`; outgoing file paths invoke `validateMediaPath`; SQL message operations use parameters. The helper implementations are missing, so their effectiveness cannot be confirmed.

### Operational defects and gaps

- **Shutdown race:** main and the reconnection goroutine both receive from `exitChan`; one signal can stop only the goroutine and leave main waiting (`3278`, `3285`). Use broadcast cancellation.
- **Incomplete revocation:** `LoggedOut` only logs; shutdown disconnects the client without implementing account unlinking or data deletion (`3048`, `3289`).
- **Send/audit mismatch:** a successful WhatsApp send can still report success after local persistence fails (`1553–1564`).
- **Migration data loss:** the LID chat migration omits `deleted_at` and `quoted_message_id` when copying messages, then deletes source rows (`439–483`).
- **Unbounded operation lifetime:** send/upload/download use `context.Background()`; HTTP timeouts do not cancel those operations.
- **Incomplete history parity:** historical processing differs from live processing, including text extraction and revoke handling. Validate captions, ordering, metadata, and deletion behavior.
- **Incomplete startup failure propagation:** REST bind failure is logged while the worker may remain connected (`2795–2798`).

## 5. Integration options and capability boundaries

| Approach | Appropriate use | Main boundary |
|---|---|---|
| Official Business Cloud API | Customer support, notifications, approved business workflows | Business messaging rules and account eligibility; no unrestricted personal inbox access |
| Business App coexistence | Businesses needing app and API access on an eligible number | Onboarding, geography, history-sharing, and throughput conditions need verification |
| Official Third-Party Agents platform | Users communicating with a connected agent | Terms establish agent interactions, not authorization to read unrelated chats; developer access/markets unverified |
| Unofficial personal bridge | Restricted internal research with accepted risks | No official personal-account API authorization verified; possible account restrictions and protocol breakage |
| User-selected imports/sharing | Selected conversation context for AI | Limited automation; users explicitly choose the data supplied |

For Cloud API, receive new messages via webhooks, store authorized data, and search your own retained records. Do not promise full historical access, every phone contact, or existing personal-group access. Evaluate any specialized official group/history features separately.

WhatsApp's [personal-service terms](https://www.whatsapp.com/legal/terms-of-service) restrict unauthorized access/use and impermissible automation. User consent does not establish Meta authorization. The [Third-Party Agents terms](https://www.whatsapp.com/legal/third-party-agents-terms) describe a separate official agent experience and distinguish it from end-to-end encrypted personal conversations.

## 6. API limits: independent ceilings

| Constraint | Planning guidance | Evidence status |
|---|---|---|
| Number throughput | Standard default: 80 incoming + outgoing messages/second; eligible upgrade: up to 1,000/second | Confirmed by [AWS integration documentation](https://docs.aws.amazon.com/social-messaging/latest/userguide/increase-message-throughput.html); verify actual number configuration |
| Proactive recipient allowance | Published tiers: 250 → 2,000 → 10,000 → 100,000 → unlimited unique recipients outside service windows per rolling 24 hours | Provisional; current Meta page inaccessible. Reported at business-portfolio scope, shared across numbers |
| Recipient pacing | Commonly published sustained limit: approximately one message every six seconds per business-number/recipient pair | Provisional; exact current and burst rules require confirmation |
| API request quotas | Endpoint/app/account-specific; distinct from message throughput | Exact current formula unverified; inspect account usage and errors |
| Service window | Free-form replies within 24 hours of a customer message; outside it, use approved templates | Verified in [Business Messaging Policy](https://business.whatsapp.com/policy) |
| Content/media | Type-specific payload, size, and format constraints | Validate chosen endpoints before implementation |
| Provider constraints | A provider may add throttles, quotas, and fees | Check selected provider contract/configuration |

Multiple agents may call concurrently, but share the connection's capacity. Extra tokens do not create extra quota. A template send does not itself open an unrestricted service window; a customer reply does. “Unlimited” recipients does not remove other limits.

**Design:** shared queues and limiters by portfolio, sending number, and recipient. Handle transient failures with bounded backoff; distinguish accepted, delivered, read, and failed states. Do not retry permanent policy failures indefinitely.

## 7. Pricing and monthly cost model

### What is verified—and what is not

- Meta's accessible [pricing overview](https://business.whatsapp.com/products/platform-pricing) describes delivery-based pricing by recipient market and category, but still states service replies and qualifying utility replies are free.
- Providers describe an **October 2026 change** charging service replies and utility messages inside the service window. The detailed Meta pricing page returned HTTP 429.
- Therefore, **the change, exact INR rates, and free allowance below are provisional**. Obtain the effective rate card and billing confirmation before approving expenditure. Do not assume all support replies remain free.
- API access does not establish AI-product eligibility; any special AI-provider terms/pricing need separate confirmation.

### India planning rates

Assumption: Indian recipients and INR billing. These provider-published figures are not a verified Meta quote. See the [published rate breakdown](https://duedoor.com/whatsapp-business-api-pricing-india/).

| Category | Typical use | Provisional rate | 10,000 billable messages |
|---|---|---:|---:|
| Marketing | Offers/promotions | ₹0.8631 | ₹8,631 |
| Utility | Transactional updates | ₹0.1150 | ₹1,150 |
| Authentication | Domestic OTPs | ₹0.1150 | ₹1,150 |
| Service | Free-form support/AI replies | ₹0.1150 | ₹1,150 |

Reported service allowance: **1,000 messages per business number/month**. Confirm before applying it. International authentication can have separate rates. Recipient markets, billing currency, discounts, and applicable taxes change the invoice.

### Budget examples

One number; Indian recipients; no special discounts/free-entry exceptions; all outgoing messages classified as service. Both scenarios apply the provisional 1,000-message allowance.

| Monthly workload | Outgoing replies | Estimated Meta service charge |
|---|---:|---:|
| 1,000 interactions × 3 replies | 3,000 | (3,000 − 1,000) × ₹0.115 = **₹230** |
| 10,000 interactions × 3 replies | 30,000 | (30,000 − 1,000) × ₹0.115 = **₹3,335** |

Additional 10,000 marketing messages would add **₹8,631** at the provisional rate. These are calculated examples, not measured production costs.

### Direct integration versus provider

| Route | Cost components | Tradeoff |
|---|---|---|
| Direct Cloud API | Meta charges + hosting/storage + AI + engineering/operations | Avoid intermediary handling fees; own onboarding and operations |
| Managed provider | Meta charges + provider handling/subscription/add-ons + AI/application costs | Operational assistance; additional fees and dependency |

[Twilio's published handling fee](https://www.twilio.com/en-us/whatsapp/pricing) is **$0.005 per incoming or outgoing message**, plus applicable Meta charges. For 30,000 incoming + 30,000 outgoing messages, this is **$300** in handling fees alone, excluding add-ons, taxes, and negotiated discounts. Do not convert to INR using an assumed exchange rate.

```text
Monthly total = Meta delivery charges
              + provider fees, if applicable
              + AI inference / transcription / vision
              + queues, compute, databases, attachments, backups
              + engineering, monitoring, support, applicable taxes
```

Internal searches/tool calls are not automatically WhatsApp billable messages. They still consume OrgWorkspace infrastructure and possibly AI tokens. Estimate AI costs from measured tokens per interaction and actual model rates; no model or workload has been specified.

## 8. Candidate architecture — if the Business API is selected

This is an evaluation option, not an approved implementation plan. Clarify personal-account requirements before selecting an architecture.

```text
Supported business onboarding → connection registry + secret manager

Meta webhook → signature verification → durable queue → deduplication
  → trusted account/workspace mapping → scoped storage → workflow/agent

Agent proposes action → authorization + consent + service-window checks
  → human approval where required → shared rate-limited sending service
  → Cloud API → status webhooks → audit and delivery state
```

| Control | Candidate safeguard |
|---|---|
| Tenant isolation | Immutable account/number-to-workspace mapping; database and object access enforcement. Resolve ownership from trusted platform state |
| Credentials | Secret manager, least privilege, rotation, revocation; never expose tokens/session files to agents |
| Data minimization | New messages first; selected conversations; bounded retrieval; explicit retention and deletion propagation |
| Storage | Encryption, restricted operators, isolated attachments, backup policy, approved data residency |
| AI security | Messages, quotes, media, and transcripts are untrusted data; no permission grants from message content |
| Sending | Authorization outside the model; approval bound to exact connection, recipient, content, and attachment |
| Reliability | Durable events, deduplication, send command tracking, reconciliation for ambiguous sends, bounded retries |
| Audit/observability | Metadata and action audit without conversation bodies, secrets, or attachments in ordinary logs |
| Offboarding | Stop workflows, revoke credentials, disconnect provider access, purge data according to policy |

Customer-facing multi-tenant onboarding may require provider/Tech Provider setup, app review, and permissions. Confirm the applicable onboarding path before committing delivery dates.

## 9. Risk assessment

| Risk | Personal bridge | Official Business integration | Residual mitigation |
|---|---|---|---|
| Account/policy | High impact; likelihood unquantified | Lower unofficial-client risk; policy restrictions still apply | Confirm eligibility, opt-in, templates, escalation |
| Privacy | High: broad personal data, logging, retained revokes | High impact if business data mishandled | Scope, retention, deletion, access controls |
| Security | High impact: linked-device sessions and content | High impact: business credentials and messages | Isolation, encryption, secret management, audit |
| Reliability | High: unofficial protocol plus identified defects | Manageable with queues and recovery; external dependency remains | Monitoring, deduplication, failure drills |
| Maintenance | High: protocol changes and custom fork | Version upgrades and policy changes remain | Named owner, upgrade process, operational budget |
| AI misuse | High if unrestricted tools are exposed | Equally serious without action controls | Scoped retrieval, external authorization, approval |

## 10. Ongoing research and decision gates

1. **Clarify the personal WhatsApp need and priority:** identify the user problem, essential personal-account capabilities, and why business messaging or selected-data alternatives would not meet it. Inventory current deployment boundaries as read-only research.
2. **Obtain missing production code:** authentication, webhook, media-path and history helpers; exact Go/Python dependencies; gateway and deployment configuration.
3. **Confirm product scope:** business support/notifications versus personal knowledge access; new messages versus history; drafts versus autonomous sends.
4. **Verify platform eligibility and billing:** effective terms, target markets, current rate card, account limits, supported onboarding/coexistence features.
5. **Compare alternatives:** assess the personal bridge, Business API, dedicated-agent interactions, and selected-data approaches against the clarified requirements. No pilot is selected yet.
6. **Document future validation needs:** cross-tenant denial, invalid signatures, duplicate events, lost responses, credential revocation, deletion, queue failure, and spend limits. Implementation and testing scope require a later decision.
7. **Measure costs:** recipients by market/category, incoming/outgoing volume, replies per interaction, AI tokens, attachment storage, and operations effort.

If a personal-account POC is later selected, potential safeguards include internal consenting participants, isolated account workers/volumes, short retention, selected chats, history off by default, bounded media, approved sends, and a kill switch. These safeguards do **not** make it Meta-authorized or prevent account restrictions. No POC has been approved through this study.

### Questions requiring an owner

- Why is personal WhatsApp specifically needed, and what is its priority?
- Which exact capabilities require a personal account, and could business messaging or selected-data alternatives meet the same need?
- Who owns each business account, billing relationship, and connected number?
- Is OrgWorkspace's specific AI offering eligible in target markets?
- What are expected numbers of connections, peak traffic, proactive recipients, and monthly messages?
- Where may messages, media, embeddings, logs, and backups reside, and for how long?
- Who approves sending, supports incidents, and maintains the connector?
- What happens on revocation, deletion requests, provider suspension, or unexpected charges?

## 11. Current position — decision pending

**Research remains active.** The immediate next step is to understand why personal WhatsApp is required and how important that requirement is. No choice between personal WhatsApp, the Business API, or another approach has been made.

The findings identify risks and candidate safeguards; they do not approve or reject an integration approach. Once the use case is clarified, compare feasibility, security, privacy, policy, reliability, and cost. Code fixes, migrations, and pilots remain outside the current research scope.

### Source index and access limitations

- [Upstream bridge code](https://github.com/lharries/whatsapp-mcp/blob/main/whatsapp-bridge/main.go), [MCP tools](https://github.com/lharries/whatsapp-mcp/blob/main/whatsapp-mcp-server/main.py), [Python storage/API access](https://github.com/lharries/whatsapp-mcp/blob/main/whatsapp-mcp-server/whatsapp.py): inspected; production variant differs substantially.
- [Upstream Go dependencies](https://github.com/lharries/whatsapp-mcp/blob/main/whatsapp-bridge/go.mod), [Python dependencies](https://github.com/lharries/whatsapp-mcp/blob/main/whatsapp-mcp-server/pyproject.toml): not proof of deployed versions.
- [Meta developer pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing), [messaging limits](https://developers.facebook.com/documentation/business-messaging/whatsapp/messaging-limits), and [coexistence onboarding](https://developers.facebook.com/documentation/business-messaging/whatsapp/embedded-signup/onboarding-business-app-users): inaccessible during review; HTTP 429.
- [Business Platform terms entry point](https://www.whatsapp.com/legal/business-solution-terms): redirected to a login/block page; effective AI-provider restrictions could not be fully verified.
- Live pricing/account configuration takes precedence over this dated study. No claim of zero security risk or guaranteed policy compliance is made.
