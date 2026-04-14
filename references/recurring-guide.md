# Recurring Payments API guide

## Environments
- UAT: `https://api.ipospays.tech/v1`
- Pre-Prod: `https://papi.ipospays.tech/v1`
- Production: `https://api.ipospays.com/v1`

## Auth
Token generation uses the same Authentication Token API as other iPOSpays APIs.
Credentials:
- `apiKey`: assigned by iPOSpays
- `secretKey`: assigned by iPOSpays
- `scope`: must be `"Recurring"`

Pass the resulting token in the `token` header on all subsequent requests.

## Get Payment Tokens API
Path: `POST /subscription/get-cards`

Returns all tokenized payment instruments (cards and ACH) linked to a customer.

Request body:
- `consumer_id` (required): unique customer identifier

Response includes `tokens[]` array:
- `token`: payment token string
- `type`: `"CARD"` or `"ACH"`
- `brand`: card brand (only when type = CARD)
- `sec_code`: ACH SEC code (only when type = ACH) — values: TEL, PPD, CCD, WEB
- `last4`: last four digits

## Setup Recurring API (create and edit)
Path: `POST /subscription/setup`

### Create — required fields
- `customer_id`: unique customer identifier
- `recurring_frequency`: WEEKLY, BI_WEEKLY, MONTHLY, BI_MONTHLY, QUARTERLY, BI_ANNUALLY, ANNUALLY, CUSTOM
- `amount`: recurring payment amount in cents (1 to 999999)
- `currency`: e.g., `"USD"`
- `tpn`: Terminal Profile Number
- `end_criteria`: NEVER, COUNT, or DATE

### Create — conditional fields
- `start_date` (YYYY-MM-DD): required for all frequencies except BI_MONTHLY and BI_ANNUALLY
- `end_value`: required when `end_criteria` is COUNT (number 1–100) or DATE (YYYY-MM-DD)
- `preferred_day`: day of week (MONDAY–SATURDAY), required for WEEKLY or BI_WEEKLY
- `card_token`: required for card-based recurring
- `ach_token`: required for ACH-based recurring (provide one or the other, not both)
- `sec_code`: for ACH — PPD, WEB, CCD, TEL; if omitted, system auto-assigns in order: PPD → WEB → CCD → TEL
- `day_1` / `day_2`: required for BI_MONTHLY (day of month 1–31) or BI_ANNUALLY (YYYY-MM-DD dates)
- `reference_id`: 1–12 alphanumeric characters (optional, can be empty)

### Edit existing subscription
- Pass `subscription_id` instead of `customer_id`
- All other fields follow the same rules as create

### Success response
Returns `data.subscription_id`, `data.customer_id`, `data.recurring_frequency`.

### Error response (status 412)
Returns `errors[]` array with `field` and `messages[]` per validation failure.

## Get Recurring Transactions API
Path: `POST /subscription/transactions`

Request: `{ "subscription_id": "..." }`

Response includes:
- `data.subscription_id`, `data.tpn`, `data.status` (ACTIVE / CANCEL), `data.amount`, `data.currency`
- `data.recurring_frequency`, `data.next_payment_date`, `data.end_criteria`, `data.end_value`
- `data.type` (CARD or ACH), `data.reference_id`
- `data.transactions[]`: each has `transaction_id`, `amount`, `last4`, `payment_date`, `tpn`, `status` (SUCCESS / FAILED)

## Active/Cancel Recurring API
Path: `POST /subscription/status`

Request:
- `subscription_id` (required)
- `status`: `"ACTIVE"` or `"CANCEL"`

Response: confirmation message, e.g., "Subscription status updated to CANCELLED."

## Error codes
- 200: Success
- 400: Bad Request — missing or invalid input
- 401: Unauthorized — invalid or expired token
- 404: Not Found — no matching consumer_id or subscription_id
- 412: Precondition Failed — validation errors (see `errors[]` in response body)
- 500: Internal Server Error

## Important gotchas
- `scope` must be exactly `"Recurring"` — do not reuse the PaymentTokenization or BatchReport scope
- `amount` is in cents (smallest unit), not dollars
- `start_date` cannot be today or in the past — must be a future date
- Provide either `card_token` or `ach_token` — not both
- For BI_MONTHLY and BI_ANNUALLY, `day_1` and `day_2` are required instead of `start_date`
- `sec_code` auto-assignment order matters: if multiple ACH token types exist, the system picks the first available in order PPD → WEB → CCD → TEL
