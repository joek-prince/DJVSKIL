# Native terminal SDKs

## Swipe Reader SDK
Use for:
- Android native magstripe card reading on supported terminal hardware

Behavior:
- returns success/failure/timeout callbacks
- returns track1 / track2 / track3 in responseData JSON string when available

Do not describe it as a full payment processor.
It reads card track data.

## Printer SDK
Use for:
- Android native printing on supported Dejavoo hardware

Important:
- only Kozen device models are supported
- limited HTML-like tags supported
- width defaults to 24 chars
- image data must be base64 in img tag
- font size is SDK controlled and not currently customizable

Do not confuse with SPIn printer-style terminal APIs.
