# Error guide

Use this file when the user asks what an error means or when choosing between likely causes.

## Guidance
- Do not guess at processor error meanings.
- Use the documented code family when known:
  - standard numeric response codes
  - SAFE risk rules (R-codes)
  - TSYS
  - Fiserv Omaha/North
  - Worldpay
  - Elavon
  - EPX

## High-value examples
- 05 can mean different things depending on context; do not oversimplify it
- R12 may indicate invalid CVV or invalid ZIP depending on the SAFE risk rule context
- D3 refers to missing/invalid 3D-Secure cryptogram in the documented error list
- B1 surcharge restrictions are processor/card-network specific
- K-series relate to tokenization/configuration contexts
- Q1 can refer to card authentication failure
- 91 often indicates host/issuer unavailable, but wording varies by processor

When answering:
1. name the processor family if known
2. quote the meaning
3. explain the likely operational implication
4. mention if the same code can differ across contexts
