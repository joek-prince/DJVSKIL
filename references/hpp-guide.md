# Hosted Payment Page (HPP) guide

## Environments
- Sandbox URL: `https://payment.ipospays.tech/api/v1/external-payment-transaction`
- Production URL: `https://payment.ipospays.com/api/v1/external-payment-transaction`

## Auth
Token is generated manually from the iPOSpays portal:
Settings → Generate Ecom/TOP Merchant Keys → select TPN → Generate Token
Send as `token` header in all requests.

This token is a merchant secret equivalent to iPOS Transact credentials (same portal path, same TPN). Server-side only. Never ship it in page JavaScript or a frontend bundle; call `getHostedPaymentPage` from your backend. The vendor's AngularJs sample calls it from the browser — do not copy that pattern.

## getHostedPaymentPage request structure
Six primary objects:

1. `merchantAuthentication` — `merchantId` (TPN), `transactionReferenceId` (unique per transaction)
2. `transactionRequest` — transaction type, amount, fee/tax/tip settings
3. `notificationOption` — how the merchant is notified after payment
4. `preferences` — AVS, e-receipt, card token request
5. `personalization` — theme/color/branding for the payment form (optional)
6. `token` (header)

### transactionRequest key fields
- `transactionType`: 1 = SALE, 2 = CARD VALIDATION ($0 pre-auth to validate/tokenize card
- `amount`: string, multiplied by 100 (e.g., $125.25 → "12525"), length 8 chars
- `calculateFee`: true/false — whether portal-configured fees are added
- `feeAmount`: custom fee amount (×100), used when `calculateFee` is false
- `feeLabel`: label for the fee
- `tipsInputPrompt`: true/false — show tip prompt on payment page
- `calcuateTax`: true/false — whether tax is calculated from TPN params (ignored for L3 data)
- `taxAmount`: custom tax (×100) when `calcuateTax` is false
- `taxLabel`: label for tax

### notificationOption fields
- `notifyByReturn`: true — result returned in the API response synchronously
- `notifyBySMS`: true — payment status sent by SMS to `mobileNumber`
- `notifyByPOST`: true — payment status posted to `postApiUrl`
- `notifyByRedirect`: true — redirects to `callbackUrl` with payment status appended. These query params are unsigned and attacker-controllable; never mark an order paid from them alone
- `mobileNumber`: phone number for SMS notification
- `postApiUrl`: merchant's endpoint to receive payment status
- `postApiAuthKey`: auth token key for the postAPI callback. Always set a high-entropy value and verify it on your endpoint; an empty key means anyone can POST a fake `Successful` payload
- `callbackUrl`: redirect URL after payment (for `notifyByRedirect`)

### preferences fields
- `avsVerification`: true/false — require billing address verification
- `customerReceiptByEmail`: true/false — send e-receipt to customer
- `customerEmail`: customer email for receipt
- `requestCardToken`: true — HPP will return a reusable card token in the response
- `cardToken`: true — alternative to `requestCardToken`

### personalization fields (all optional)
- `logoUrl`: logo image URL shown on payment form
- `themeColor`: hex color for payment page theme
- `buttonColor`: hex color for payment button
- `cancelUrl`: URL to redirect customer if they cancel
- `cancelButtonText`: text for cancel button

## notifyByReturn response fields (POST API data for card)
Key fields returned in the callback/response:
- `transactionReferenceId`
- `transactionStatus`: Successful / Declined / Cancelled by Customer / Rejected by Customer / GeneratedURL Success / GeneratedURL Failure
- `amount`, `tipAmount`, `feeAmount`, `taxAmount`, `totalAmount`
- `cardType`
- `last4`
- `approvalCode`
- `rrn`
- `invoiceNo`
- `cardToken` (if `requestCardToken` was true)
- `consumerId`

### ACH-specific POST response fields
- `achToken`
- `bankName`
- `accountType`
- `routingNumber` (masked)
- `accountNumber` (masked)

## queryPaymentStatus API
Used to check payment status by `transactionReferenceId`. Call it server-side before fulfilling any order that was reported via `notifyByRedirect` or an unauthenticated `notifyByPOST`.
Endpoint: same base URL, different path (queryPaymentStatus)
Required: `merchantAuthentication.merchantId` + `merchantAuthentication.transactionReferenceId`

## Level 3 / VISA CEDP via HPP
- Send `l3Data` object in `transactionRequest`
- `calcuateTax` is ignored when L3 data is present (tax comes from L3 header)
- L3 data includes header (tax, discount, freight) and line items

## Card tokenization flow via HPP
- Set `requestCardToken: true` in preferences
- Or set `transactionType: 2` for $0 card validation
- The HPP response returns `cardToken` — this is a reusable token for future iPOS Transact calls

## Error codes
- 401 — Cancelled by Customer

## Important gotchas
- Amount must be multiplied by 100 (value in cents, last 2 digits treated as decimals)
- `notifyByRedirect` and `notifyByPOST` can be combined
- `notifyBySMS` is independent of the other notification modes
- `transactionReferenceId` must be unique per transaction. Generate it server-side (unguessable, not `Math.random()`) and persist it with the expected amount
- Build `transactionRequest.amount` from the server-side order, never from a client-supplied value
- Before fulfillment require `transactionStatus` = Successful AND `totalAmount` (and `transactionReferenceId`) matching the stored order. A customer who tampers with the amount can otherwise pay $0.01 and receive a full-price order
- `cardToken` returned from HPP is reusable; `paymentTokenId` from FTD is single-use — do not conflate
- Token from sandbox will not work in production
