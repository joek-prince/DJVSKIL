# Tap to Pay guide

## Android
Use for Android in-app softPOS.
Key ideas:
- uses ToPService / iPOSgo SDK
- register device with TPN + merchant code
- registration returns session_key and auth_token
- validate session freshness before each transaction
- supports perform transaction, tip adjustment, batch settlement, reports

Production gotchas:
- production AAR / fin provisioning
- SHA-256 sharing
- finalization process before release

## iPhone deep linking
Two versions are documented:
- V1 = older, wrapper-based deep-linking
- V2 = newer, more structured and configurable

Use when:
- the app triggers iPOSgo flows externally rather than embedding the full in-app SDK

## iPhone in-app SDK
Use when:
- the user wants native in-app Tap to Pay integration on iPhone

Requirements:
- US region
- iPhone Xs or later
- iOS 16.9+
- Tap to Pay entitlement
- sandbox Apple tester account for UAT

Important:
- only one receipt channel per API call (SMS or email, not both in a meaningful dual-delivery sense)
- supports status check and external payment modes in the documented SDK

## Do not confuse with
- HPP
- SPIn
- generic browser card entry
