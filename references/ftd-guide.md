# Freedom to Design (FTD) guide

FTD is a browser-side data collection and tokenization layer.
It is NOT a full payment processor. To complete a transaction, pass the resulting token to iPOS Transact.

FTD is NOT hosted fields. `freedomtodesign.js` and `ipospays-ftd-ach.js` read values from native `<input>` elements in your own page (no iframe isolation), so any XSS, third-party script, or browser extension on the checkout page can read PAN/CVV or bank/SSN data. PAN never reaches your server, but the page stays in PCI scope (SAQ A-EP at best). Use HPP when you need real scope reduction.

localhost is blocked by PCI compliance rules. Contact ISV Sales or support@dejavoo.io to enable it for testing.

## Auth / setup
- Log in to iPOSpays portal → Settings → Generate Ecom/TOP Merchant Keys → select TPN → Generate Token
- Add your domain to the Whitelisted Domain list
- Use the generated token as `security_key` in the script tag
- `security_key` is visible to anyone who views page source. Generate a token for the single TPN used by FTD and use it for nothing else; never reuse your HPP/Transact token here. Domain whitelisting limits where the script runs, not who can read the key

## Card embed (HTML)
Script URL (sandbox): `https://payment.ipospays.tech/ftd/v1/freedomtodesign.js`
Script URL (production): `https://payment.ipospays.com/ftd/v1/freedomtodesign.js`

Add the script tag with `id="ftd"` and `security_key="<your_auth_token>"` and `defer`.
Required input IDs: `ccnumber`, `ccexpiry`, `cccvv`, and a submit button with `id="payButton"`.

On submit, call `await postData()`. The response contains `payment_token_id`.

## paymentTokenId behavior
- Single-use only
- Expires after 24 hours if unused
- Pass to iPOS Transact to complete the transaction
- After a $0 Pre-Auth via iPOS Transact, the response returns a reusable `Card Token / iPOS Token`
- Do not confuse `paymentTokenId` with a reusable token

## Google Pay embed
Script URL (sandbox): `https://payment.ipospays.tech/ftd/v1/gpay-ftd.js`
Script URL (production): `https://payment.ipospays.com/ftd/v1/gpay-ftd.js`

Container div: `<div id="ipospays-gpay-btn"></div>`

Initialize with `initializeGooglePay(transactionInfo, merchantId, Mode, buttonStyles, googlePayFields)`.

`transactionInfo` fields:
- `countryCode` (2-letter ISO)
- `currencyCode` (ISO)
- `totalPriceStatus` — FINAL or ESTIMATED
- `totalPrice` — string amount shown on the wallet sheet. Display-only: the charged amount is `amount` in your server-side iPOS Transact call. Do not call `updatePrice()` from a user-editable input
- `totalPriceLabel`

`buttonStyles` fields: `buttonColor`, `buttonType`, `buttonRadius`, `buttonLocale`, `buttonHeight`, `buttonWidth`

`googlePayFields`: `requestBillingAddress`, `requestPayerEmail`, `requestPayerPhone`, `requestShipping`

`Mode`: `"TEST"` or `"PRODUCTION"`

Callback: `function getPaymentInfo(paymentData)` — receives the payment token to forward to iPOS Transact. This is a token handoff, not payment completion.

## Apple Pay embed
Script URL (sandbox): `https://payment.ipospays.tech/ftd/v1/ipospays-apple-pay-ftd.js`
Script URL (production): `https://payment.ipospays.com/ftd/v1/ipospays-apple-pay-ftd.js`

Add script with `id="applepayftd"`.
Container div: `<div id="ipospays-apple-pay-button"></div>`

Initialize with:
```
initializeApplePay(transactionInfo, buttonStyles, merchantId, applePayFields, applePayMerchantIdentifier)
```

`transactionInfo` fields: `countryCode`, `currencyCode`, `totalPriceStatus`, `totalPrice`

`buttonStyles` fields: `buttonColor` (black/white), `buttonType`, `buttonRadius`, `buttonLocale`, `buttonHeight`, `buttonWidth`

`applePayFields`: `requestBillingAddress`, `requestPayerEmail`, `requestPayerPhone`, `requestShipping`, `requestPayerName`

`merchantId` — provided by iPOSpays
`applePayMerchantIdentifier` — from Apple Developer portal

Callback: `function getApplePaymentInfo(paymentData)` — send `paymentData` to your backend as `applePayToken` for iPOS Transact processing. Your backend sets `amount` from the stored order and only reports success after iPOS Transact approves; a `success` flag from your own endpoint is not proof of payment.

After Apple domain verification is complete, switch to production mode.

## ACH embed
Script URL (sandbox): `https://payment.ipospays.tech/ftd/v1/ipospays-ftd-ach.js`
Script URL (production): `https://payment.ipospays.com/ftd/v1/ipospays-ftd-ach.js`

Add script with `id="ftdach"` and `security_key="<auth_token>"` and `defer`.

Form field container `div` IDs (rendered dynamically by the script):
FIRST_NAME, LAST_NAME, PHONE_NUMBER, EMAIL, ADDRESS1, ADDRESS2, CITY, STATE, ZIP, DL_NUMBER, DL_STATE,
ACCOUNT_NUMBER, ROUTING_NUMBER, ACCOUNT_TYPE, IDENTITY, SSN, DOB, CUSTOM_TAG1, CUSTOM_TAG2, CUSTOM_TAG3

Initialize the form: `await initializeAchForm({ security_key: SECURITY_KEY })`

Metadata callback: `window.getAchPaymentInfo(data)` — receives `terminalId`, `entryClass`, `usersId`, `identity`

These metadata fields are required when submitting to iPOS Transact via `achData`:
- `identity`, `terminalId`, `entryClass`, `usersId`
- Use `sourceType: "ACH-FTD"` and `transactionType: 10` in the iPOS Transact request
- Set `requestAchToken: true` in preferences to get an `achToken` back in the response

## Important gotchas
- FTD only handles data collection and tokenization — iPOS Transact is always needed to process the payment
- `paymentTokenId` is single-use; the reusable `cardToken`/`iPOS Token` comes back from iPOS Transact
- `localhost` is blocked; must whitelist domain in portal and get it enabled for local dev
- For ACH, the IDENTITY field controls conditional display of SSN and DOB — that logic is the integrator's responsibility
- ACH fields are native inputs and `getallFieldsData` returns clear account/routing/SSN/DOB/DL values to your page JS. Never log them, and the merchant backend that forwards `achData` to Transact is handling raw bank data and PII (GLBA/NACHA scope)
- Amount is never taken from the browser. Set `amount` server-side from the order for card, wallet, and ACH alike
