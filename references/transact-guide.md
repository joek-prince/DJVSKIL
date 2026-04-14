# iPOS Transact guide

## Versions
- V1: transactions + AVS, portal-based token generation
- V2: transactions + AVS, portal-based token generation; adds Google Pay, Paya Token, ACH Token, HSA/FSA
- V3: latest; supports encrypted card data and API-based token generation (Authentication Token API)

Prefer V3 when:
- encryptedCardData is used
- API-based token generation is needed (instead of manual portal)

## Endpoints
- V1/V2 Sandbox: `https://payment.ipospays.tech/api/v1/ipostransact`
- V3 Sandbox: `https://payment.ipospays.tech/api/v3/ipostransact`
- V1/V2 Production: `https://payment.ipospays.com/api/v1/ipostransact`
- V3 Production: `https://payment.ipospays.com/api/v3/ipostransact`

Use the V3 endpoint exclusively for `encryptedCardData` transactions.

## Common transaction types
- 1 = Sale
- 5 = PreAuth
- 2 = Void
- 3 = Refund
- 6 = Ticket
- 7 = Tip Adjust
- 0 = Zero Dollar Tip Adjust
- 4 = Batch Settlement
- 8 = Incremental Auth
- 9 = Open Batch
- 10 = ACH sale
- 11 = ACH void/refund
- 12 / response 23 = Gift flows are documented
- HSA/FSA and Level 3 / VISA CEDP are supported by documented payload extensions

## Auth — V1 / V2 (portal-based)
Log in to iPOSpays portal → Settings → Generate Ecom/TOP Merchant Keys → select TPN → Generate Token.
Send the resulting token in the `token` request header.

## Auth — V3 (API-based Authentication Token API)
POST credentials to the Authentication Token API:
```json
{
  "apiKey": "your_api_key_here",
  "secretKey": "your_secret_key_here",
  "scope": "PaymentTokenization"
}
```
- `apiKey` and `secretKey` are issued by iPOSpays
- `scope` must be exactly `"PaymentTokenization"` for iPOS Transact
- All three fields are required for V3; may not be mandatory for V1/V2

### Token lifetime and grace period
- Token is valid for 24 hours
- After expiry: 30-minute grace period during which the expired token still works
- During grace period: response header includes `AuthenticationRefreshRequired: true`
- Must call the Refresh Token API to get a new token during the grace period
- Both the expired (within grace period) and new token work in the 30-minute window
- After grace period ends: expired token is invalid; only new token accepted

## Important request concepts
- `merchantAuthentication.merchantId` = TPN / merchant identifier
- `transactionReferenceId` = caller-generated unique reference
- `amount` is typically sent in cents/smallest-unit style string
- `applySteamSettingTipFeeTax` controls whether portal-configured fees/taxes/tips are calculated

## Card transaction input choices
1. paymentTokenId
- single use
- from FTD

2. cardToken / iPOS Token
- reusable token

3. encryptedCardData
- use V3
- PCI-sensitive path

## Void / refund / ticket / tip adjust
For regular card flows:
- use original RRN

## ACH
ACH has separate logic.
- ACH sale uses transactionType 10
- ACH void/refund uses ACH-specific transaction ID behavior
- Do not assume the standard card RRN-only flow is enough

## Processor-specific gotcha
- reconId is documented as conditionally important for Fiserv and effectively mandatory there with 10 characters

## Level 3 / VISA CEDP
Use only when the merchant has real line-item data.
Do not invent placeholder commercial data.
Bad L3 quality can hurt qualification.

## HSA/FSA
Only use HSA object fields when the transaction is actually an HSA/FSA flow.
