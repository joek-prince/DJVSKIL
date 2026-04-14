# Merchant onboarding guide

Use when the user needs to create merchants, stores, and TPNs via API.

## Auth
Uses API key auth, not the same token model as iPOS Transact.
Header format: `Authorization: API KEY <your-api-key-value>`
Contact devsupport@denovosystem.com (cc your Dejavoo sales rep) to obtain credentials.

## Endpoints
- Sandbox: `https://externalapi.ipospays.tech/v1/onboardmerchant`
- Production: `https://externalapi.ipospays.com/v1/onboardmerchant`

## Main objects
Three mandatory objects in every request:
- `merchantData`
- `storeData`
- `tpnData`

### merchantData key fields
- `dba` (required): merchant DBA name, 2–35 chars
- `firstName` (required), `lastName` (optional)
- `email` (required)
- `addressLine1` (required), `addressLine2` (optional)
- `zipCode` (required): 5-digit; city/state derived from zip
- `phoneNumber` (required): 10–12 digits
- `ISDcode` (required): e.g., `+1`
- `ebt` (optional): 7-char alphanumeric
- `isoAgentReference`, `isoMerchant`, `merchantLogoUrl` (all optional)

### storeData key fields
- `mccCode` (required): 4-digit Merchant Category Code
- `firstName` (required), `lastName` (optional)
- `dba` (required), `email` (required)
- `addressLine1` (required), `zipCode` (required), `phoneNumber` (required), `ISDcode` (required)

### tpnData key fields
- `deviceManufacturer` (required): Cloud POS / Android Phone / WizardPOS / Kozen
- `deviceModel` (required): depends on manufacturer (see below)
- `tpnApplicationSign` (required): pre-defined value, typically `"CRDT"`
- `tpnRoutingType` (required): `"SIMPLE"` (default)
- `tpnAssignTemplate` (optional): template name from portal (5–30 chars)
- `defaultLabel` (optional): 18 chars

## Device model / manufacturer combinations
- Kozen → P1, P3, P5, P8, P12
- WizardPOS → Q2, QD2, QD3, QD4, QD5, QD6
- Cloud POS → Cloud POS
- Android Phone → Tap on Phone
- Apple → Apple Pay

## Templates
Templates must be created in the portal first, then assigned by name in `tpnAssignTemplate`.
Templates allow pre-configuring parameters for specific merchant verticals (e.g., restaurant settings).

## Processor-specific data (tpnProcessor object)
Each processor has unique required fields.

### TSYS
- `processorName`: "TSYS"
- `profileId`: "01" (with fee) or "02" (without fee)
- `Mid` (12 digits), `Agent` (6 digits), `Chain` (6 digits), `Store` (4 digits)
- `TermNo` (4 digits), `TermId` (8 alphanumeric), `tpnBin` (6 digits)
- `ABA_No` (9 digits, optional), `Agent_FIID` (4 alphanumeric, optional)
- `dsGroup` (1–30 chars, optional)

### Other processors
Each has its own required/optional field set. EBT placement rules differ between `merchantData` and `storeData`.

## Important gotchas
- Templates are created in portal, not via API — assign by name only
- Device manufacturer and model combinations are conditional — e.g., P1 is only valid under Kozen
- Processor object requirements differ significantly by processor
- EBT placement rules differ between merchantData and storeData
- `zipCode` determines city and state — do not send city/state separately
- Tokens from sandbox will not work in production
