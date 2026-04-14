# Reporting and FEED guide

## FEED
Use when transactions and settlements need to be pushed to the integrator endpoint.

Configure in iPOSpays portal (ISO account): Settings → FEED → paste endpoint URL + choose authentication.

Auth options:
- Basic Authentication (Authorization: Basic <base64> header on push)
- HMAC (signature field in payload, computed with HmacSHA512)

### FEED transaction payload fields
- `id`, `version`, `createdDt`, `eventType`, `subEventType`, `requestType`, `signature`, `data`

### FEED HMAC — Transaction preimage construction
Algorithm: HmacSHA512

**Outer fields** (in this exact alphabetical TreeMap order, joined with `|`):
1. `id`
2. `version`
3. `createdDt`
4. `eventType`
5. `subEventType`
6. `requestType`
7. `data` → the result of converting the nested data object to a string (see below)

**data object inner field order** (alphabetical TreeMap, joined with `|`):
`additionalDetails | approvalCode | baseAmount | buildNumber | cardCategory | cardInfo | cardLabel | cardToken | cardType | chName | commMedia | commercialTaxAmount | description | destType | deviceModel | dba | email | externalReferenceId | hostResponseCode | hostResponseDate | invoiceNumber | L2L3uploadflag | localTaxAmount | maskedPan | Mid | minorChannel | originalRRN | performedBy | phoneNumber | posEntryMode | posMode | posRequestDate | posRequestTime | rrn | responseText | sourceType | stateTaxAmount | tagLabel | tagValue | TermId | tipAdjAmount | tipAmount | totalFee | tpn | tpnBatchNumber | tpnLabel | transactionId | transactionType | trueCashDiscountFee | txDate | txDuration | txName | txTime | amount | baseAmount | commercialTaxAmount | localTaxAmount | stateTaxAmount | tipAmount | tipAdjAmount | totalFee | trueCashDiscountFee`

Note: field `tpnLabel` in the outer data object maps to `data.getTagLabel()` in the source — not to a separate `tpnLabel` field. Be careful not to map the wrong source field.

Null/missing values are replaced with empty string `""` before concatenation.

The concatenated string is HMAC-SHA512 signed with the secret key (UTF-8 bytes). The result is a hex-encoded string and placed in the `signature` field of the payload.

### FEED HMAC — Settlement preimage construction
Algorithm: HmacSHA512

**Settlement outer fields** (alphabetical TreeMap order, joined with `|`):
1. `id`
2. `tpn`
3. `version`
4. `createdDt`
5. `eventType`
6. `subEventType`
7. `requestType`
8. `batchNumber`
9. `settlementDate`
10. `settlementCount`
11. `settlementAmount`
12. `settlementTxnDetails` → converted to string (see below)

**settlementTxnDetails inner field order** (per item, alphabetical TreeMap, joined with `|`; items joined with `|`):
`txnDate | transactionType | transactionId | txnAmount | baseAmount`

### Transaction event types
- `eventType`: Transaction or Settlement
- `subEventType` (Transaction): BALANCE ENQ, INC AUTH, INC SALE, PREAUTH, REFUND, SALE, TICKET, TIP ADJUST, VOID, VOID PREAUTH, VOID REFUND, VOID SALE, VOID TICKET
- `subEventType` (Settlement): Closed Batch
- `sourceType`: ECOM, TPOS, ITAP, TOPH

---

## Batch Report API
Use for closed batch reporting.
Endpoint: `POST https://payment.ipospays.tech/api/v1/batch-report` (sandbox)
Endpoint: `POST https://payment.ipospays.com/api/v1/batch-report` (production)

Auth token scope: `"BatchReport"`
Send token in `token` header.

Request body:
- `merchantId` (required): TPN
- `batchDate` (YYYY-MM-DD): required if `batchNo` is used
- `batchNo`: required if `batchDate` is used

Behavior:
- `merchantId` only → latest closed-batch date set
- `merchantId` + `batchDate` → all closed batches for that date
- `merchantId` + `batchDate` + `batchNo` → specific batch (returns a single batch object)

Response structure:
- `batch_profiles[]`: array of profile objects, each with `batch_details[]` (per-transaction) and `batch_summary[]` (by type)
- `batch_total`, `total_transactions`, `batch_number`, `batch_status`
- `total_tip`, `total_fee`, `total_without_fee`
- `cash_payment`: `cash_txn_count`, `cash_payment_total`, `data[]`
- `alter_payments`: `alter_txn_count`, `alter_payment_total`, `total_tip`, `data[]`

Key `batch_details` fields per transaction:
`transactionId`, `invoiceNumber`, `type` (SALE etc.), `amount`, `baseAmount`, `feeAmount`, `tipAmount`, `approvalCode`, `rrn`, `account` (masked PAN), `card`, `voidFlag`, `posRequestDate`, `posRequestTime`, `externalRefId`

---

## Transaction Status Check
Use when the caller has:
- `merchantId` (TPN)
- original `transactionReferenceId`

Endpoint: `POST https://payment.ipospays.tech/api/v1/iposTransactStatus` (sandbox)
Endpoint: `POST https://payment.ipospays.com/api/v1/iposTransactStatus` (production)

Auth: `token` header (portal-generated, same as HPP/FTD).

Request body: `{ "merchantAuthentication": { "merchantId": "...", "transactionReferenceId": "..." } }`

Response for card transactions:
- `RespCode`, `AppCode`, `RespMsg`, `RespStatus` (Approved/Declined)
- `TxRefNo`, `DTxId`, `HTxId`, `BatchNo`, `InvoiceNo`, `Rrn`, `ProcessingCode`
- `Amount`, `Tips`, `CustomFee`, `FeeLabel`, `LocalTax`, `StateTax`, `TotalAmount`
- `TxnSettleStatus` (e.g., "Yet to Settle")
- `TransactionType` (SALE, REFUND, etc.)
- `Card.Label`, `Card.MaskedPan`, `Card.ChdToken`
- `TxDetail.DateNTime` (YYYYMMDDHHMMSS), `TxDetail.Tpn`

Response for ACH/wallet:
- `CardType`, `AchToken`, `TransactionType` = CHECK
- No `Card` nested object

Error response when `transactionReferenceId` not found:
- `errors[].field` = `"IPOSTRANCT_001"`, `errors[].message` = "No Transaction Registered with This TransactionRefId..."

This is useful for recovery if the original response was missed or interrupted.
Response structure differs between card and alternate-payment cases.
