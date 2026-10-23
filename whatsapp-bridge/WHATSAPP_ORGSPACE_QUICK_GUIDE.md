# WhatsApp connector for Orgspace

**Reviewed: 23 October 2026 · Implementation proposal**

**Recommendation:** Use the official WhatsApp Business Cloud API for production. For personal WhatsApp, start with user-selected chat imports/sharing; keep the unofficial bridge as an explicitly accepted internal experiment.

## 1. Implement the Business connector

```text
Customer WhatsApp → Meta webhook → Orgspace connector → workspace inbox/search
Orgspace reply → permission checks + send queue → Cloud API → customer
```

1. **Connect:** Create a Meta app, WhatsApp Business Account and business number; configure billing, credentials and webhook subscriptions. For customer-owned accounts, use Embedded Signup and obtain the required app permissions/review.
2. **Receive:** Verify webhook signatures, map the business number to its workspace, deduplicate events, and store messages/media with workspace access controls. Process delivery/read/failure updates.
3. **Expose to Orgspace:** Add tools to search retained messages, read threads, draft replies and send approved messages. Enforce authorization outside the AI model; start with human approval for sends.
4. **Operate:** Encrypt credentials and retained data, add deletion/retention controls, audit actions, queue sends and retry transient failures without duplicate delivery. Provide human escalation for automated support.

Use the current Go bridge's message model as a reference; implement a separate Cloud API adapter. The official connector covers conversations with the business number, **not unrestricted access to a user's personal inbox or old chat history**. Check Orgspace's exact AI use case against current platform terms before onboarding. [Business policy](https://business.whatsapp.com/policy)

## 2. API and voice-calling limits

| Area | Practical constraint |
|---|---|
| Message throughput | Standard numbers: 80 incoming + outgoing messages/second; eligible numbers can reach 1,000. Confirm the actual account configuration. [AWS documentation](https://docs.aws.amazon.com/social-messaging/latest/userguide/increase-message-throughput.html) |
| Proactive messages | Separate recipient allowance and quality controls apply; read the account's live allowance. More workers/tokens do not increase it. |
| Reply window | Free-form replies within 24 hours of the customer's latest message; approved templates outside it. Obtain opt-in and respect opt-outs. [Meta policy](https://business.whatsapp.com/policy) |
| API calls | Endpoint quotas are separate from message throughput. Use shared limiters, bounded backoff and account usage/error monitoring. |
| Voice calls | A separate Calling API integration is needed, with account/country eligibility and explicit permission for outbound calls. Twilio currently documents 1 permission request/24 hours, 2/7 days and up to 1,000 concurrent calls per sender; confirm selected-provider limits before launch. [Calling documentation](https://www.twilio.com/docs/voice/whatsapp-business-calling) |

The supplied `main.go` records call events/metadata; it does not implement a voice media connection. Start with messaging; add calling as a separate phase.

## 3. Cost: what to budget

**Monthly total = Meta message charges + voice usage + provider fees (if any) + hosting/storage + AI usage + operations.** Direct Cloud API avoids a provider's additional markup; Orgspace still runs the connector.

Meta message pricing depends on delivered-message category, recipient market and applicable volume tier. **Current pricing needs billing confirmation:** Meta's public overview still describes free service/qualifying utility replies, while provider publications describe October 2026 changes; the detailed Meta rate card returned HTTP 429 during review. Do not promise free support replies. [Meta pricing and rate-card links](https://business.whatsapp.com/products/platform-pricing)

**Illustrative budget, not a verified tariff:** at an assumed ₹0.115 per billable utility message, 10,000 messages cost ₹1,150; at ₹0.8631 per marketing message, 10,000 cost ₹8,631. Replace these assumptions with the effective account rate card; add taxes and the other costs above.

For voice, obtain the destination-specific calling rate and provider fees separately; message prices do not cover call minutes.

## 4. Best practical approach to personal WhatsApp

- **Default:** Let users import selected chat exports or share selected messages into Orgspace. Offer summaries/search and draft replies that the user sends manually. This gives useful personal context with explicit data selection.
- **Business workflow:** Use a dedicated Business number. If keeping the mobile Business App is necessary, evaluate official App/API coexistence eligibility; this does not provide general personal-inbox access.
- **If live personal sync is essential for an internal pilot:** The existing `whatsmeow` bridge pairs by QR as a linked device, stores messages locally and exposes REST/webhook access. Isolate one account per worker/volume, authenticate pairing and APIs, allowlist chats, default to read-only, and implement unlink/delete. History sync is best effort; there is no published safe automation quota or guaranteed ban-free configuration.

## 5. Cons of unofficial personal-account automation

- **Account loss:** Unauthorized automation can trigger restriction or banning, including loss of access to an important personal number. User consent does not equal Meta authorization. [WhatsApp guidance](https://faq.whatsapp.com/361005896189245/?eea=0&locale=en_US)
- **Unreliable service:** Protocol changes, session expiry and re-pairing can interrupt sync; no official API support or SLA.
- **Privacy exposure:** The bridge decrypts messages and creates additional local/AI copies. Session theft, broad chat access, logs and backups increase exposure.
- **Maintenance burden:** History gaps, duplicate events and deletion/disappearing-message behavior need custom handling; low messaging fees can be outweighed by engineering and incidents.

**Delivery order:** Business messaging pilot → personal imports/drafts → optional official calling. Keep unofficial personal sync outside the default production connector.
