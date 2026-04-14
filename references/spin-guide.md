# SPIn REST API guide

SPIn is the terminal/device control API. Use it for semi-integrated terminal operations.

## Base URLs
- Production: `https://api.spinpos.net`
- Sandbox: `https://test.spinpos.net/spin`

## Authentication / common required fields
Every request requires terminal identification. Use either:
- `Tpn` (Terminal Profile Number, 10–12 chars) — preferred
- `RegisterId` (2–50 chars, obsolete) — required only if no Tpn

Plus:
- `Authkey` (10 chars): merchant's authorization password. Required if no SPInToken.

All endpoints also accept:
- `SPInProxyTimeout` (int): timeout in seconds (1–720). Default 120s, settlement default 420s.
- `CustomFields` (object): key-value custom fields.

## Payment endpoints

### POST v2/Payment/Sale
Most common request. Processes a payment on the terminal.
Key body fields:
- `Amount` (double, required): total amount
- `TipAmount` (double): tip
- `PaymentType` (required): Credit / Debit / EBT_Food / EBT_Cash / Card / Cash / Check / Gift
- `ReferenceId` (string, 1–50, required): unique alphanumeric SPIn transaction identifier within the batch
- `PrintReceipt`: No / Both / Merchant / Customer
- `GetReceipt`: No / Both / Merchant / Customer
- `InvoiceNumber` (string, max 50)
- `CaptureSignature` (bool)
- `GetExtendedData` (bool)
- `IsReadyForIS` (bool): whether register can receive intermediate status
- `CallbackInfo.Url`: callback URL for async result
- `MerchantNumber` (1–5): for multi-merchant environments (required if multi-merchant, else transaction cancelled)
- `Cart` (object): optional cart with Amounts[], CashPrices[], Items[]

### POST v2/Payment/Return
Independent return — does NOT link to a prior Sale. Unique ReferenceId required.
Charges back and adds money to card balance (opposite of Sale).
Key fields: `Amount`, `PaymentType`, `ReferenceId` (all required)

### POST /v2/Payment/TipAdjust
Adjusts tip on an existing transaction before settlement.
Requires knowing the `ReferenceId` and `Amount` of the original transaction.

### POST /v2/Payment/Auth
Pre-authorization (reserve funds). A Tip amount is NOT allowed in Auth requests.

### POST /v2/Payment/Capture
Captures a previous Auth.

### POST v2/Payment/Void
Cancels a transaction before settlement. Must be done before Settle.
Cannot void after settlement — use Return instead.
Void may be done for: Sale, Return, Auth, Capture.
Requires `Amount`, `PaymentType`, `ReferenceId` of the transaction being voided.

### POST v2/Payment/Status
Check status of a transaction by ReferenceId.

### POST v2/Payment/StatusList
Check status of multiple transactions.

### POST /v2/Payment/OfflineStatus
Check offline transaction status.

### POST v2/Payment/Settle
Settles the batch.
- `SettlementType`: Close / Clear / Force
- `ReferenceId` (optional)
- `GetReceipt` (bool)

### POST /v2/Payment/AbortTransaction
Abort a transaction in progress.

### POST /v2/Payment/Cart
Cart-style payment.

### POST v2/Payment/GetCard
Retrieve card data.

### POST v2/Payment/Balance
Check balance (e.g., EBT, gift card).

## Report endpoints
- `POST v2/Report/Summary` — summary report
- `POST v2/Report/Daily` — daily report

## Common/utility endpoints
- `POST v2/Common/Printer` — print on terminal
- `POST v2/Common/GetSignature` — capture customer signature
- `POST v2/Common/UserChoice` — present a choice to the customer on terminal
- `POST v2/Common/UserInput` — request text input from customer
- `POST v2/Common/Disclaimer` — display disclaimer on terminal (requires touch-capable terminal)
- `GET v2/Common/TerminalStatus?request.tpn=...&request.registerId=...` — terminal connection status

## Table app endpoints
- `POST /v2/TableApp/SelectInvoice`
- `POST v2/TableApp/PaymentInvoice`

## Gift card endpoints
- `POST v2/Gift/Reload`
- `POST v2/Gift/Reissue`
- `POST v2/Gift/Refund`
- `POST v2/Gift/Redeem`
- `POST v2/Gift/Inquire`
- `POST v2/Gift/Deactivate`
- `POST v2/Gift/Activate`
- `POST v2/Gift/Void`

## Level 2/3 endpoints
- `POST v2/L2L3/AddLevel3Item`
- `GET v2/L2L3/ViewLevel3Item`
- `POST v2/L2L3/RemoveLevel3Item`

## CEDP
- `POST v2/CEDP/Sale`

## Intermediate status
- `GET v2/IntermediateStatus/GetIntermediateStatus`
- `POST v2/IntermediateStatus/PostIntermediateStatus`

## Response structure — GeneralResponse
Every response includes a `GeneralResponse` object:
- `HostResponseCode`: processor response code (ISO 8583)
- `HostResponseMessage`: processor response message
- `ResultCode`: 0 = success, 1 = terminal error, 2 = SPIn proxy error
- `StatusCode`: 4-digit code (see below)
- `Message`: short message
- `DetailedMessage`: English description of the status
- `DelayBeforeNextRequest`: seconds to wait before next request (returned when proxy is busy)

## StatusCode families
### 0xxx — Successful responses from terminal
- `0000` Approved
- `0001` Partial Approval

### 1xxx — Error responses from terminal
- `1000` Terminal Busy (or in SPIN bypass mode)
- `1001` Not Found (transaction/parameter not found)
- `1002` Not Implemented (unsupported service)
- `1003` Not Supported (e.g., Disclaimer on non-touch terminal)
- `1004` Not Allowed (e.g., Void Auth on unsupported processor, tip on Auth request)
- `1005` Low Battery
- `1006` Internal Error
- `1007` Format Error (malformed or unsupported markup)
- `1008` Wrong Payment Or Transaction Type
- `1009` Authentication Failed (AuthKey mismatch — check AuthKey or pull parameters on terminal)
- `1010` Missing Reference ID
- `1011` Duplicate Reference ID (already exists in batch)
- `1012` Cancelled (user cancelled or unknown interruption)
- `1013` Bad Request (various validation failures: 0 amount, invalid fields, tip disabled, etc.)
- `1014` Communication Error (third-party host error)
- `1015` Declined (offline or host declined)
- `1016` Payment Type Mismatch
- `1017` Incorrect Merchant ID
- `1018` PINpad Communication Error
- `1019` No Debit Keys Loaded
- `1020` No Open Batch
- `1021` Pending SnF Transaction
- `1022` Untipped Transactions Remain
- `1023` Open Tab

### 20xx — Error responses from SPIn Proxy
- `2001` Terminal not connected to SPIn Proxy (check internet + SPIn proxy status)
- `2002` Active AuthKey not found (wrong AuthKey or doesn't exist)
- `2003` Register not found (wrong TPN, or wrong server — test vs prod)
- `2004` Route not found (TPN has no SPIn proxy config)
- `2005` Active route not found (connection blocked, terminal inactive, or two devices sharing same TPN)
- `2006` Not parseable request (extra space in values, malformed request)
- `2007` Operation timed out (terminal didn't respond; default 120s, settlement 420s)
- `2008` Terminal in use (previous request in progress — use `DelayBeforeNextRequest`)
- `2009` Transaction not found
- `2010` Communication error
- `2011` Terminal not available

### 21xx — Async callback errors
- `2101` Callback URL not specified
- `2102` Invalid XML document
- `2110` Internal exception

## Transaction type rules
- Void and Tip Adjust can only follow a Sale (Sale root)
- Auth → Capture root is separate
- Return is an independent root (not linked to a prior Sale)
- Cannot void after settlement — use Return

## Extended data
- Response includes `ExtendedDataByApplication` when `GetExtendedData: true`
- Includes `CardData` (CardType, EntryType, Last4, First4, BIN, ExpirationDate, Name, CardBrand)
- Includes `EMVData` (ApplicationName, AID, TVR, TSI, IAD, ARC)
- Response includes `IPosToken` — can be used for future tokenized transactions via iPOS Transact
- Response includes `Token` for future card-based transactions
- Response includes `RRN` (Reference Retrieval Number) for iPOS gateway API use

## Important gotchas
- `ReferenceId` must be unique within a batch
- Multi-merchant: if `MerchantNumber` is absent in a multi-merchant environment, the transaction is cancelled
- Tip Adjust requires the transaction to be unsettled and the terminal to be in restaurant mode
- Disclaimer endpoint requires a touch-capable terminal (1003 error otherwise)
- Two devices sharing the same TPN causes 2005 errors
- Default timeout is 120s (not 30 or 60); settlement is 420s
