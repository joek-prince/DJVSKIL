# Reporting and FEED guide

## FEED
Use when transactions and settlements need to be pushed to the integrator endpoint.

Auth options:
- Basic Authentication
- HMAC

Important:
- HMAC examples use HmacSHA512
- payload concatenation order matters
- transaction and settlement payload signing order differ
- the sample transaction logic includes both tpnLabel and tagLabel handling; be careful not to map the wrong source field in HMAC preimage construction

Transaction FEED supports eventType/subEventType/sourceType style metadata.
Settlement FEED supports settlement batch payloads.

## Batch Report API
Use for closed batch reporting.
Auth token scope:
- BatchReport

Request inputs:
- merchantId
- optionally batchDate
- optionally batchNo

Behavior:
- merchantId only = latest closed-batch date set
- merchantId + batchDate = all closed batches for that date
- merchantId + batchDate + batchNo = specific batch

## Transaction Status Check
Use when the caller has:
- merchantId
- original transactionReferenceId

This is useful for recovery if the original response was missed or interrupted.
Response differs somewhat between card and alternate-payment cases.
