---
name: ipos-integration-guide
description: choose the correct dejavoo/ipospays integration path and provide accurate implementation guidance. use when chatgpt needs to decide between spin, freedom to design, hosted payment page, ipos transact, recurring payments, tap to pay, feed, batch reporting, transaction status, swipe reader sdk, printer sdk, merchant onboarding, or clear card encryption. use for payload guidance, token flow clarification, pci-sensitive clear-card decisions, auth scope selection, and implementation gotchas. 
---

Use this skill to answer implementation questions about Dejavoo / iPOSpays products and APIs.

Core behavior:
- Identify the user's actual integration scenario before answering.
- Choose the correct product/API/SDK lane.
- Explain why that lane fits.
- Explain why nearby lanes do not fit when confusion is likely.
- Give implementation guidance that is explicit, cautious, and grounded in the bundled references.
- Prefer accuracy over completeness when the docs are ambiguous or inconsistent.

Mandatory response pattern:
1. Best integration choice
2. Why it fits
3. Required auth / tokens / credentials
4. Required request fields or flow
5. Important gotchas / constraints
6. Better alternatives if the chosen path is risky or over-scoped

Non-negotiable rules:
- Never describe Freedom to Design (FTD) as the full processor path. FTD is mainly for browser-side collection/tokenization and still depends on iPOS Transact to complete payment processing.
- Never confuse `paymentTokenId` with reusable `cardToken` / `iPOS Token`.
  - `paymentTokenId` = single-use token, typically from FTD
  - `cardToken` / `iPOS Token` = reusable token from prior flows like Transact, HPP, or SPIn
- Never recommend clear-card handling casually. If the user wants to handle raw PAN/expiry/CVV, warn that their application must be PCI compliant and route them to the clear-card encryption reference.
- Prefer iPOS Transact V3 when encrypted card data is involved.
- For regular card void/refund in iPOS Transact, use the original RRN-based flow.
- For ACH void/refund in iPOS Transact, use the ACH-specific transaction ID logic instead of assuming the normal card flow.
- Use documented auth scopes exactly as described in the references. Do not invent scopes.
- If a field meaning or endpoint behavior is uncertain, say so explicitly.

Use these references:
- For choosing the correct product/API, read `references/decision-guide.md`
- For token behavior and token-type confusion, read `references/token-model.md`
- For iPOS Transact details (auth, endpoints, transaction types, V3), read `references/transact-guide.md`
- For Freedom to Design (JS embed, Google/Apple Pay, ACH form, paymentTokenId), read `references/ftd-guide.md`
- For Hosted Payment Page (request params, notification modes, webhook/redirect, card tokenization), read `references/hpp-guide.md`
- For Recurring Payments API (subscription create/edit/cancel/list, token lookup), read `references/recurring-guide.md`
- For SPIn REST API (terminal operations, endpoints, error codes, GeneralResponse), read `references/spin-guide.md`
- For Tap to Pay, read `references/tap-to-pay-guide.md`
- For native terminal SDKs, read `references/native-terminal-sdks.md`
- For FEED (HMAC concatenation spec), Batch Report API, and Transaction Status Check, read `references/reporting-and-feed-guide.md`
- For clear-card / PCI-sensitive flows, read `references/clear-card-encryption.md`
- For onboarding merchants and TPNs, read `references/onboarding-guide.md`
- For errors, read `references/error-guide.md`

Decision discipline:
- If the user needs hosted/remote checkout or payment links, lean toward HPP.
- If the user needs browser-side card or ACH collection and tokenization, lean toward FTD.
- If the user needs backend transaction execution, lean toward iPOS Transact.
- If the user needs terminal/device control, lean toward SPIn.
- If the user needs subscriptions / recurring billing, lean toward the Recurring Payments API.
- If the user needs Android/iPhone softPOS, lean toward Tap to Pay SDK / deep-link paths.
- If the user needs terminal hardware features inside native Android apps, lean toward the Swipe Reader SDK or Printer SDK.
- If the user needs downstream posting of transaction or settlement events to their system, lean toward FEED.
- If the user needs historical closed batch detail, lean toward Batch Report API.
- If the user needs transaction recovery/status by original reference, lean toward Transaction Status Check.

Answering style:
- Be concrete and implementation-oriented.
- Use short labeled sections.
- Name exact tokens, endpoints, or required identifiers when known.
- Surface gotchas early, especially around PCI, token reuse, HMAC ordering, auth scopes, processor-specific constraints, and iPhone/Android entitlement requirements.
