# iPOS Transact guide

## Versions
- V1: transactions + AVS, portal-based token generation
- V2: transactions + AVS, portal-based token generation
- V3: latest, supports encrypted card data and API-based token generation

Prefer V3 when:
- encryptedCardData is used
- newer token API flows are required

## Common transaction types
- 1 = Sale
- 5 = PreAuth
- 2 = Void
- 3 = Refund
- 6 = Ticket
- 7 = Tip Adjust
- 4 = Batch Settlement
- 8 = Incremental Auth
- 9 = Open Batch
- 10 = ACH sale
- 11 = ACH void/refund
- 12 / response 23 = Gift flows are documented
- HSA/FSA and Level 3 / VISA CEDP are supported by documented payload extensions

## Auth
For V3 token generation, docs describe:
- apiKey
- secretKey
- scope = PaymentTokenization

Access token behavior:
- valid for 24 hours
- 30-minute grace period after expiry
- may include AuthenticationRefreshRequired: true during grace period

## Important request concepts
- merchantAuthentication.merchantId = TPN / merchant identifier
- transactionReferenceId = caller-generated unique reference
- amount is typically sent in cents/smallest-unit style string
- applySteamSettingTipFeeTax controls whether portal-configured fees/taxes/tips are calculated

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
