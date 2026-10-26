# Clear card encryption

Use this file only when the user intentionally wants a direct card-entry / encrypted clear-card path.

## Warning
If the application accesses or handles clear card data, the integrating application must be PCI compliant.

Do not recommend this flow casually.
Prefer hosted/tokenized paths when possible:
- HPP (fully hosted; keeps card data off the merchant page)
- FTD (reduces scope: PAN never reaches your server, but inputs live in your page, so the page itself stays in PCI scope — see ftd-guide.md)
- Tap to Pay
- SPIn
- reusable token flows

## Auth
Use Authentication Token API with:
- apiKey
- secretKey
- scope = PaymentTokenization

## Card object
The documented example card object includes:
- ccNumber
- ccExpiry
- ccCvv

## Encryption flow
- Encrypt the raw card object using the RSA public key provided by iPOSpays
- The output becomes `encryptedCardData`
- Send `encryptedCardData` to iPOS Transact
- Prefer iPOS Transact V3 for this path

## Do not confuse with
- paymentTokenId from FTD
- reusable cardToken / iPOS Token
- HPP returned tokens
