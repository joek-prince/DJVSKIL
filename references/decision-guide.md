# Decision guide

Use this file to choose the correct Dejavoo / iPOSpays product lane.

## 1. SPIn
Use when the user needs terminal/device control or semi-integrated terminal operations.
Typical use cases:
- terminal-driven sale / auth / capture / void / refund
- terminal status
- settle / summary / daily report
- prompts / signature / card retrieval / balance style terminal actions

Do not choose SPIn for:
- hosted checkout links
- browser tokenization
- recurring subscriptions
- iPhone/Android softPOS SDK questions

## 2. Freedom to Design (FTD)
Use when the user needs browser-side payment data collection or tokenization.
Typical use cases:
- embedded card form
- embedded ACH form
- Google Pay / Apple Pay browser collection
- obtaining `paymentTokenId`

Important:
- FTD is mainly the collection/tokenization layer.
- It is not the full transaction processor path.
- `paymentTokenId` is single-use.

## 3. Hosted Payment Page (HPP)
Use when the user needs hosted checkout / payment links / redirect or send-link style flows.
Typical use cases:
- remote payment page
- payment links
- redirect checkout
- requestCardToken through hosted flow
- remote customer payment collection without building own form

## 4. iPOS Transact
Use when the user needs backend payment transaction execution.
Typical use cases:
- sale
- pre-auth
- void
- refund
- ticket
- tip adjust
- batch settlement
- open batch
- gift
- ACH
- HSA/FSA
- Level 3 / VISA CEDP
- encryptedCardData

Important:
- Prefer V3 for encrypted card data.
- This is the main backend transaction rail.

## 5. Recurring Payments API
Use when the user needs subscription setup or recurring billing behavior.
Typical use cases:
- create recurring subscription
- edit recurring subscription
- retrieve recurring transactions
- cancel/reactivate recurring subscription
- list customer payment tokens for recurring use

## 6. Tap to Pay
Use when the user needs softPOS acceptance on Android or iPhone.
Android:
- in-app SDK using device registration + session handling
iPhone:
- deep linking (V1/V2)
- in-app SDK

Do not treat Tap to Pay as HPP or SPIn.

## 7. Native terminal SDKs
Use when the user needs Android terminal hardware features.
- Swipe Reader SDK = read magstripe card tracks
- Printer SDK = print on supported terminal hardware

## 8. FEED
Use when the user needs transaction/settlement events posted to their own endpoint.
Supports:
- transaction events
- settlement events
- basic auth or HMAC validation

## 9. Batch Report API
Use when the user needs closed batch detail and settlement-era batch breakdown.

## 10. Transaction Status Check
Use when the user needs the status of a transaction using the original `transactionReferenceId`.

## 11. Merchant Onboarding
Use when the user needs to create merchants, stores, and TPNs through API.

## 12. Clear Card Encryption
Use only when the user intentionally wants a PCI-scoped direct card-entry path.
This is not the first recommendation unless they truly need raw card handling.
