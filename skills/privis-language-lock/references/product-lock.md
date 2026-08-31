# PRIVIS product lock

Product name: PRIVIS
PS ID: SIH26171
PS title: On-device Visual Perception for Light-weight Browser Agents
Organization: ISRO / Department of Space
Category: Software

One-liner: local eyes, local eraser, remote brain

## Architecture boxes (never rename)

1. Capture Layer
2. Local Privacy Vision Engine
3. Sanitizer
4. Policy Gate
5. Remote Agent
6. Local Executor

## Placeholders

- EMAIL_1
- PAN_1
- AADHAAR_1
- AMOUNT_1
- PHONE_1
- NAME_1
- FACE (blur / box)
- PASSWORD (black-out)

## Comparison line

Today: full screenshot leaves the device.
PRIVIS: sanitized screenshot + structured context only, after Policy Gate.

## Official metric weights

- Visual context accuracy — 25%
- PII precision and recall — 20%
- Redaction precision — 20%
- Client resource utilization — 20%
- End-to-end latency — 15%
