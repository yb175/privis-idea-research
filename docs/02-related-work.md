# Related work and gap

## Products send the screen away

From Ukani et al., arXiv:2512.07725:

| System | Browser location | Model location |
|---|---|---|
| Claude Computer Use | Local / container | Off-device |
| Claude for Chrome | Local extension | Off-device |
| ChatGPT Agent | Cloud browser | Off-device |
| Perplexity Comet | Local | Off-device |
| Amazon Nova Act | Local Chrome | Off-device |

## Papers that almost solve SIH26171

**PrivWeb** (CHI 2026, arXiv:2509.11939, doi:10.1145/3772318.3790919)
Local add-on redacts DOM with a local LLM (Qwen3-8B / Ollama in their impl), pauses on high-sensitivity fields. N=15 formative, N=14 user study: lower perceived risk, no extra cognitive load.
Gap vs PS: 8B local LLM is heavy for a student extension; not specified as WebGPU ViT + screenshot sanitize.

**Available but Invisible** (arXiv:2602.10139)
Type-preserving placeholders (`PHONE_NUMBER#a1b2c`). Layers: PII Detector, UI Transformer, Secure Interaction Proxy, Privacy Gatekeeper. Sensitive values usable for the task but never visible to the cloud model.
Gap: mobile GUI / AndroidLab, not Chrome extension + ISRO PS.

**MINIM** (ICML 2026, arXiv:2606.13949, https://github.com/yyyyhx/MINIM)
Local broker scores sensitivity and task necessity; keep / abstract / remove before observation leaves the device. WebArena trees: TCNP 0.9491, TISL 0.1010 vs 1.0 full observation.
Gap: a11y-tree minimization, not local screenshot ViT + canvas redaction.

**WebPII / WebRedact** (arXiv:2603.17357, https://webpii.github.io/)
44,865 synthetic UIs, 993,461 boxes. LayoutLMv3+GPT-4o-mini 0.357 mAP@50 at 2900ms. WebRedact 0.753 mAP@50 at 20ms CPU.
Gap: detection only, no agent loop / policy / executor.

## Gap PRIVIS fills

On-device vision + sanitize-before-network + remote reason + local act + Indian PII + student-hardware latency.
