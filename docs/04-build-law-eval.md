# Law, eval, plan, glossary, sources

## Indian PII the demo must hit

| Class | Sanitize as |
|---|---|
| PAN (5 letters + 4 digits + 1 letter) | PAN_1 |
| Aadhaar (12 digits) | AADHAAR_1 |
| Indian mobile (10 digits, 6-9 start) | PHONE_1 |
| Email | EMAIL_1 |
| Salary | AMOUNT_1 |
| Password / OTP | black-out |
| Face | blur |

No teammate real PAN on slides.

## DPDP Act 2023

- Act 22 of 2023, 11 August 2023. https://cadp.in/resources/official-texts/dpdp-act-2023/
- s.6(1): consent limited to personal data necessary for the specified purpose. https://www.certinal.com/blog/data-minimization-under-dpdp-explained
- Sending a full HR screenshot to a foreign model to click Apply Leave is hard to call necessary. Sending a redacted view plus click Submit is closer to minimization.
- This is architecture, not a legal opinion.

## Threat model

In scope: honest-but-curious model vendor that logs images; accidental PAN/salary/face on a normal portal.
Out of scope for SIH: compromised store listing, kernel malware, homomorphic crypto.
Gate exists because vision can miss a PAN inside a PDF viewer, or over-redact Submit.

## 36-hour order

1. Capture screenshot + DOM boxes
2. Regex + password detector
3. Canvas black-out
4. Placeholder JSON
5. Gate that blocks raw PAN in outbound payload
6. Stub remote `click #submit`
7. Local Executor
8. Real VLM
9. Face/text ONNX if time

1-7 already matches the PS shape.

## Glossary

- Bounding box: pixel rectangle to find or hide something
- Content script: can read DOM
- Service worker: can screenshot, cannot read DOM
- Manifest: permission sheet, not the algorithm
- Type-preserving placeholder: EMAIL_1 still looks like an email field
- Policy Gate: last check before network
- Local Executor: clicks the real page
- ViT: vision transformer or equivalent allowed by the PS
- HITL: human approval branch

## Sources for slide 6

1. SIH26171 — https://sih2026.vuce.in/en/ps/SIH26171
2. Ukani et al. Privacy Practices of Browser Agents. arXiv:2512.07725
3. Anthropic computer use tool docs
4. Anthropic Cowork computer-use personal data warning
5. Zhang et al. PrivWeb. arXiv:2509.11939 / CHI 2026
6. Zhao et al. Available but Invisible. arXiv:2602.10139
7. Yu et al. MINIM. arXiv:2606.13949 / ICML 2026
8. Zhao. WebPII. arXiv:2603.17357 / https://webpii.github.io/
9. DPDP Act 2023 official text
10. DPDP s.6(1) minimization explainer
