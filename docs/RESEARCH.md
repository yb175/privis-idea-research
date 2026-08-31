# PRIVIS Research Brief

**Product:** PRIVIS  
**Problem statement:** SIH26171 — On-device Visual Perception for Light-weight Browser Agents  
**Organization:** Indian Space Research Organisation (ISRO) / Department of Space  
**Category:** Software  
**Audience:** teammates, mentors, SIH evaluators, and coding agents  
**Architecture source of truth:** `handdraw/architecture.png`

This file is the complete idea lock. If a slide, prompt, or module disagrees with this document, this document wins.

---

## 0. How to read this

| Reader | Start here |
|---|---|
| Human teammate | Sections 1–4, then the hand-drawn diagram |
| Mentor / judge | Sections 1, 4, 8, 11, 12 |
| Coding agent | Sections 3, 5–7, 9, glossary |
| PPT agent | Sections 1, 4, 8 + hand-drawn image on slide 3 only |

**One line:** local eyes, local eraser, remote brain.

**What we are not building:** a full local VLM that does passport applications by itself, a generic RPA bot, or a cloud PII SaaS.

---

## 1. The real-world problem

AI is moving from answering questions to operating a computer.

Today a user asks: "How do I apply for a passport?"  
Tomorrow a user will say: "Apply for my passport."

To do that an agent must see the page: fields, buttons, errors, already-filled values, documents, faces in photos. That visual context is useful. It is also full of PAN, Aadhaar, salary, passwords, emails, phone numbers, and faces.

Commercial computer-use and browser agents send screenshots or page state to an **off-device** model.

- Anthropic's computer-use tool takes screenshots and returns click / type actions. The model runs off-device. Anthropic's own Cowork safety page states that Claude takes screenshots and can see personal data visible on screen.  
  Sources: [Anthropic computer use tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool), [Safeguarding personal data in Cowork](https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork).
- A 2025–2026 measurement paper evaluated eight popular browser agents. All studied agents that use an LLM put the **model off-device**. The paper found 30 privacy issues across the set, including leaking personal information into sites and weakening browser privacy features.  
  Source: Ukani et al., *Privacy Practices of Browser Agents*, arXiv:2512.07725, https://arxiv.org/abs/2512.07725

So the core tension is not "can we automate a form?" Browser automation already exists.  
It is not "can we detect PII?" PII detectors already exist.

The intersection is:

```text
BROWSER
   |
   v
Visual environment
   |
   |- useful context     -> agent needs it
   |- sensitive data     -> agent must not see raw values
               |
               v
     THE PROBLEM SIH26171 ASKS TO SOLVE
```

A laptop cannot host a frontier VLM. A cloud VLM cannot be allowed to see raw Indian identity documents. PRIVIS sits in the middle.

---

## 2. What SIH26171 actually asks

Public statement title: **On-device Visual Perception for Light-weight Browser Agents**.

From the SIH 2026 problem text:

- Modern browser APIs (WebGPU, WebAssembly) and local inference (ONNX Runtime Web, Transformers.js) can run **lightweight** models on the client.
- Goal: use cloud / server reasoning **and** enforce privacy on the client.
- Build a privacy-preserving **vision agent that runs in the browser**.
- A local Vision Transformer or equivalent **reads the screen** and supports decisions.
- Teams must trade **latency vs accuracy**.

Expected prototype pieces:

1. Client extension / JS on Chrome and Firefox.
2. **Local vision processing** in the browser (example: WebGPU) that evaluates the current screen.
3. **Privacy-preserving filter** before data leaves: bounding-box redaction, semantic obfuscation, masking — must be demonstrated.
4. A server that reasons on the sanitized view and returns actions.
5. Client executes those actions.

Source: https://sih2026.vuce.in/en/ps/SIH26171

### Scoring the team should optimize

| Dimension | Weight we design for | What the demo must show |
|---|---|---|
| Visual context still usable after sanitize | 25% | Buttons, labels, form structure readable |
| PII detection precision / recall | 20% | PAN, Aadhaar, email, phone, face, password |
| Redaction precision | 20% | Mask hits the PII box, not Submit |
| Client resource use | 20% | Mid-range laptop, no 70B local model |
| End-to-end latency | 15% | One form step in interactive time |

If the official rubric on the portal differs, follow the portal. Do not invent 99% accuracy claims.

---

## 3. Why this is a market gap

### 3.1 Agents already send the screen away

| System | Where the browser runs | Where the model runs |
|---|---|---|
| Claude Computer Use | Local / container | Off-device |
| Claude for Chrome | Local extension | Off-device |
| ChatGPT Agent (Operator successor) | Cloud browser | Off-device |
| Perplexity Comet | Local | Off-device |
| Amazon Nova Act | Local Chrome | Off-device |

Source: Ukani et al., *Privacy Practices of Browser Agents*, arXiv:2512.07725.

### 3.2 Research already names the missing layer

**PrivWeb** (CHI 2026) — local add-on redacts DOM with a local LLM (Qwen3-8B via Ollama in their impl), pauses on high-sensitivity data. Formative N=15, user study N=14.  
Sources: arXiv:2509.11939, doi:10.1145/3772318.3790919

**Available but Invisible** (Zhao et al., 2026) — type-preserving placeholders such as `PHONE_NUMBER#a1b2c`. Layers: PII Detector, UI Transformer, Secure Interaction Proxy, Privacy Gatekeeper.  
Source: arXiv:2602.10139

**MINIM** (ICML 2026) — local broker scores sensitivity and task necessity; keep / abstract / remove. WebArena trees: TCNP 0.9491, TISL 0.1010 vs full observation TISL 1.0.  
Sources: arXiv:2606.13949, https://github.com/yyyyhx/MINIM

**WebPII / WebRedact** — 44,865 synthetic UIs. LayoutLMv3 + GPT-4o-mini: 0.357 mAP@50 at 2,900 ms. WebRedact: 0.753 mAP@50 at 20 ms CPU.  
Sources: arXiv:2603.17357, https://webpii.github.io/

### 3.3 Gap SIH26171 leaves open

| Existing work | Covers | Missing for SIH26171 |
|---|---|---|
| PrivWeb | DOM + local LLM redaction + HITL | Heavy 8B local LLM; not WebGPU ViT + screenshot sanitize as specified |
| Available but Invisible | Placeholders + gatekeeper | Mobile / AndroidLab, not Chrome extension |
| MINIM | Task-conditioned a11y minimal view | Not screenshot redaction + local ViT |
| WebRedact | Fast visual PII boxes | Detection only, not agent loop |
| Claude / Operator / Comet | Strong remote reasoning | Raw screen leaves the device |

**PRIVIS =** on-device vision + sanitize-before-network + remote reason + local act + Indian PII + student-hardware latency.

---

## 4. The idea (PRIVIS)

PRIVIS is a hybrid browser agent privacy pipeline.

1. User states a goal.
2. **Capture Layer** reads screenshot + DOM/A11y + bounding boxes + browser state. Memory only.
3. **Local Privacy Vision Engine** finds faces, text regions, sensitive patterns, and DOM PII.
4. Engine emits boxes + element IDs + categories.
5. **Sanitizer** blurs / blacks out pixels and replaces strings with `EMAIL_1`, `PAN_1`, `AMOUNT_1`.
6. Layout, button labels, non-sensitive text stay.
7. **Policy Gate** allows, pauses for human, or blocks.
8. **Remote Agent** sees only the sanitized screenshot + sanitized JSON.
9. **Local Executor** runs click / type / scroll on the real DOM.

The remote model never receives raw PAN. The executor can still type the real value locally if the gate allowed it.

This is the available-but-invisible principle inside a browser extension.

---

## 5. Hand-drawn architecture (source of truth)

File: `handdraw/architecture.png`

```text
User + goal
    -> CAPTURE LAYER
         Screenshot | DOM+A11y+visible text+element bounding boxes | BrowserState
    -> LOCAL PRIVACY VISION ENGINE
         Vision model -> faces, text regions, sensitive patterns
         DOM Detector (Rule + ML)
    -> OUTPUT
         list of sensitive bounding boxes + element IDs + categories
    -> SANITIZER
         Visual: blur faces, black-out passwords, mask text regions
         Structural: email -> EMAIL_1, PAN -> PAN_1, salary -> AMOUNT_1
         Preserve layout, button labels, form structure, non-sensitive text
    -> Sanitized Screenshot + Sanitized Context
    -> POLICY GATE
         Residual -> Human Approval
         Safe -> Remote Agent -> Local Executor
```

Slide numbers: 1 Capture Layer, 2 Local Privacy Vision Engine, 3 Output, 4 Sanitizer, 5 Policy Gate, 6 Remote Agent, 7 Local Executor.

Do not rename these boxes.

---

## 6. Layer contracts

### 6.1 Capture Layer

Manifest V3 extension.

| Piece | Who | Why |
|---|---|---|
| Screenshot | Service worker | `tabs.captureVisibleTab` is not available to the page script |
| DOM, A11y, visible text, boxes | Content script | Only the page world can read the live DOM |
| Browser state | Either, then merge | URL is needed by Policy Gate |

- Bounding box: `[x, y, width, height]` in viewport pixels.
- A11y tree: roles/names (`textbox`, `button`).
- Browser state: URL, title, viewport. Not cookies.
- Storage: memory only. Do not write raw screenshots to disk.

### 6.2 Local Privacy Vision Engine

Vision path (PS requirement): WebGPU / ONNX Runtime Web / Transformers.js / MediaPipe for faces and text regions.

DOM path: `input[type=password]`, PAN regex `[A-Z]{5}[0-9]{4}[A-Z]`, Aadhaar, Indian mobile, email, nearby labels.

Fusion: union of boxes. Python trains ONNX. Inference runs in the extension, not a local Python process.

### 6.3 Sanitizer

Visual: canvas copy; blur faces; black-out passwords; mask PII text.
Structural: session-stable placeholders. Mapping table stays on device.
Preserve Submit, Leave Balance, Reason, nav labels.

### 6.4 Policy Gate

| Decision | When |
|---|---|
| Allow / Safe | High-sensitivity items redacted, high confidence |
| Human Approval | Low confidence leftover, password, pay/submit on sensitive host |
| Block | Bank/payroll URL with residual raw PII |

### 6.5 Remote Agent

Sees sanitized screenshot + sanitized JSON + goal. Returns `{ "type": "click", "target": "#submit" }`. Must not receive raw PAN.

### 6.6 Local Executor

Content script on the real page. Resolves target, clicks/types, loops back to Capture.

---

## 7. Implementation map

| Layer | Language | Runtime |
|---|---|---|
| Extension, capture, sanitize, gate, executor | JS / TS | Chrome / Firefox MV3 |
| Vision weights | Python train, ONNX export | ONNX Runtime Web |
| Remote reasoner | Python FastAPI + VLM | Team server |

`manifest.json` is the permission sheet, not the algorithm. Background cannot read the DOM. Content cannot take `captureVisibleTab`.

---

## 8. Indian PII and law

| Class | Sanitize as |
|---|---|
| PAN | PAN_1 |
| Aadhaar | AADHAAR_1 |
| Indian mobile | PHONE_1 |
| Email | EMAIL_1 |
| Salary | AMOUNT_1 |
| Password / OTP | black-out |
| Face | blur |

DPDP Act 2023 (Act 22 of 2023). s.6(1): consent limited to personal data necessary for the specified purpose.  
https://cadp.in/resources/official-texts/dpdp-act-2023/  
https://www.certinal.com/blog/data-minimization-under-dpdp-explained

PRIVIS is architecture, not a legal opinion.

---

## 9. Threat model

In scope: honest-but-curious remote model that logs images; accidental PAN/salary/face on a normal portal.
Out of scope for SIH: compromised store listing, kernel malware, homomorphic crypto.
Gate exists for PDF-viewer misses, over-redaction, empty-DOM iframes.

---

## 10. What maximum cases means

A case is handled if capture got a viewport shot, a detector or the gate stopped raw send, sanitizer applied a box or placeholder, the remote model can still see Submit, and the executor can hit the real control.

Demo set: fake employee portal, gov-style form clone, login page.

---

## 11. Related systems vs PRIVIS

| | Typical browser agent | PRIVIS |
|---|---|---|
| Screen to cloud | Full screenshot | After Policy Gate, sanitized |
| Privacy | Warning | Local filter + gate |
| Agent utility | Sees raw PAN | Sees PAN_1 and Submit |
| Action | Cloud or model | Local Executor |
| Hardware | Vendor GPU | Student laptop |

---

## 12. 36-hour build order

1. Capture screenshot + DOM boxes
2. Regex + password detector
3. Canvas black-out
4. Placeholder JSON
5. Gate blocks raw PAN in outbound payload
6. Stub remote click #submit
7. Local Executor
8. Real VLM
9. Face/text ONNX if time

Steps 1-7 already match the PS shape.

---

## 13. Glossary

| Term | Meaning |
|---|---|
| Bounding box | Pixel rectangle used to find or hide something |
| Content script | JS in the page; can read DOM |
| Service worker | Background; can screenshot; cannot read DOM |
| Manifest | Extension permissions |
| Type-preserving placeholder | EMAIL_1 still looks like an email field |
| Policy Gate | Last check before network |
| Local Executor | Clicks the real page |
| ViT | Vision Transformer or equivalent |
| HITL | Human approval branch |

---

## 14. Sources (use on slide 6)

1. SIH26171 — https://sih2026.vuce.in/en/ps/SIH26171
2. Ukani et al. *Privacy Practices of Browser Agents*. arXiv:2512.07725
3. Anthropic computer use tool docs
4. Anthropic Cowork computer-use personal-data warning
5. Zhang et al. PrivWeb. arXiv:2509.11939
6. Zhao et al. Available but Invisible. arXiv:2602.10139
7. Yu et al. MINIM. arXiv:2606.13949
8. Zhao. WebPII. arXiv:2603.17357 — https://webpii.github.io/
9. DPDP Act 2023 official text
10. DPDP s.6(1) minimization explainer

---

## 15. Agent instructions

```text
Read docs/RESEARCH.md and handdraw/architecture.png before writing code or slides.
Product name PRIVIS. Do not rename architecture boxes.
Portal PPT: 6 official SIH slides. Architecture image on slide 3 only.
Do not claim 99% accuracy.
Do not put raw teammate PII in artifacts.
Python trains models. JS extension runs capture, sanitize, gate, executor.
Remote model sees sanitized data only.
```
