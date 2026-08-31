# Instructions for coding / PPT agents

Read `docs/RESEARCH.md` first.
Use `handdraw/architecture.png` as the architecture visual.

Product: PRIVIS
PS: SIH26171
Boxes (do not rename): Capture Layer, Local Privacy Vision Engine, Sanitizer, Policy Gate, Remote Agent, Local Executor
Placeholders: EMAIL_1, PAN_1, AADHAAR_1, AMOUNT_1, PHONE_1, NAME_1

Portal PPT: exactly 6 official SIH slides. Architecture on slide 3 only.
Python trains ONNX. JS extension runs capture, sanitize, gate, executor.
Remote model sees sanitized data only.
No 99% claims. No real teammate PII.
