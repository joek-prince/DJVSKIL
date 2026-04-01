# Merchant onboarding guide

Use when the user needs to create merchants, stores, and TPNs via API.

## Auth
Uses API key auth, not the same token model as iPOS Transact.
Header format is documented as API KEY / api-key with the issued value.

## Main objects
- merchantData
- storeData
- tpnData

## Common use cases
- create merchant + store + TPN
- create store + TPN under an existing merchant
- assign templates during onboarding
- choose device manufacturer/model/routing/application sign
- provide processor-specific profile data

## Important gotchas
- templates are created in portal, then assigned by API
- device manufacturer and model combinations are conditional
- processor object requirements differ significantly by processor
- EBT placement rules differ between merchantData and storeData
