# Problem and SIH26171

**Product:** PRIVIS  
**PS:** SIH26171 — On-device Visual Perception for Light-weight Browser Agents  
**Org:** ISRO / Department of Space  
**One-liner:** local eyes, local eraser, remote brain.

Architecture source of truth: `handdraw/architecture.png`

## Real-world problem

AI is moving from answering questions to operating a computer. An agent that can apply for a passport must see fields, buttons, errors, documents and faces. That screen also contains PAN, Aadhaar, salary, passwords, email and phone.

Commercial computer-use agents send screenshots to an **off-device** model.

- Anthropic computer use takes screenshots and returns click/type actions. Cowork safety docs state Claude can see personal data visible on screen.
  - https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool
  - https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork
- Ukani et al. measured eight browser agents. LLM-backed agents put the model off-device and the paper reports 30 privacy issues.
  - *Privacy Practices of Browser Agents*, arXiv:2512.07725, https://arxiv.org/abs/2512.07725

This is not a generic RPA problem and not a standalone PII-detector problem. It is the intersection: visual agent + privacy + on-device compute + low latency.

## What SIH26171 asks

Source: https://sih2026.vuce.in/en/ps/SIH26171

- Run lightweight vision in the browser (WebGPU, WASM, ONNX Runtime Web, Transformers.js).
- Use server reasoning while enforcing privacy on the client.
- Local ViT or equivalent reads the screen.
- Demonstrate sanitizing sensitive visual data (bounding-box redaction, masking, obfuscation) **before** network send.
- Chrome and Firefox extension + server.
- Show latency vs accuracy tradeoff.

## Demo contract the team designs for

| Dimension | Weight | Demo must show |
|---|---|---|
| Visual context still usable | 25% | Buttons and form structure readable |
| PII precision / recall | 20% | PAN, Aadhaar, email, phone, face, password |
| Redaction precision | 20% | Mask hits PII box, not Submit |
| Client resources | 20% | Laptop, no 70B local model |
| End-to-end latency | 15% | One form step in interactive time |

Follow the official portal rubric if it differs. Never claim 99% accuracy.
