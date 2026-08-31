# Architecture and layer contracts

Visual source of truth: [`../handdraw/architecture.png`](../handdraw/architecture.png) (team hand-drawn diagram).

## Idea

PRIVIS is a hybrid pipeline: local eyes, local eraser, remote brain.

1. User gives a goal.
2. Capture Layer reads screenshot + DOM/A11y + bounding boxes + browser state (memory only).
3. Local Privacy Vision Engine finds faces, text regions, sensitive patterns, DOM PII.
4. Output: boxes + element IDs + categories.
5. Sanitizer blurs/blacks pixels and replaces strings with EMAIL_1 / PAN_1 / AMOUNT_1.
6. Policy Gate allows, asks a human, or blocks.
7. Remote Agent sees only sanitized screenshot + JSON.
8. Local Executor clicks the real DOM.

This is the *available but invisible* idea (arXiv:2602.10139) inside a browser extension.

## Hand-drawn flow (must match the PNG)

```text
User + goal
  -> CAPTURE LAYER
       Screenshot | DOM+A11y+visible text+element bounding boxes | BrowserState
  -> LOCAL PRIVACY VISION ENGINE
       Vision model -> faces, text regions, sensitive patterns
       DOM Detector (Rule+ML)
  -> OUTPUT (boxes + element IDs + categories)
  -> SANITIZER
       Visual redaction + type-preserving placeholders
       Preserve layout, button labels, non-sensitive text
  -> Sanitized Screenshot + Sanitized Context
  -> POLICY GATE
       Residual -> Human Approval
       Safe -> Remote Agent -> Local Executor
```

Slide numbering: 1 Capture, 2 Privacy Engine, 3 Output, 4 Sanitizer, 5 Policy Gate, 6 Remote Agent, 7 Local Executor. Do not rename.

## Capture Layer

MV3 extension. Background takes the screenshot (`tabs.captureVisibleTab`). Content script reads DOM. They cannot swap jobs — that is a Chrome rule.

Bounding box = `[x,y,w,h]` in viewport pixels, used to paint redaction.
Screenshot stays in memory. Do not save raw shots to disk.

## Local Privacy Vision Engine

Vision (PS requirement): ONNX / Transformers.js / MediaPipe / WebGPU for faces and text regions.
DOM: password inputs + PAN / Aadhaar / phone / email regex + nearby labels.
Fusion: union of boxes, keep confidence.
Python trains and exports ONNX. Inference is in the extension, not a local Python process.

## Sanitizer

Visual: blur faces, black-out passwords, mask PII text on a canvas copy.
Structural: session-stable placeholders. Mapping table never leaves the device.
Keep Submit, Leave Balance, Reason, nav labels.

## Policy Gate

| Decision | When |
|---|---|
| Allow | High-sensitivity items redacted, high confidence |
| Human Approval | Low confidence leftover, password, pay/submit on sensitive host |
| Block | Bank/payroll URL with residual raw PII |

Same role as PrivWeb pause + Zhao et al. Privacy Gatekeeper.

## Remote Agent

Input: sanitized screenshot + sanitized JSON + goal.
Output: `{ "type": "click", "target": "#submit" }`
Must never receive ABCDE1234F.

## Local Executor

Content script on the real page. Resolves target, clicks/types, loops back to Capture.
If a real saved value must be typed and the gate allowed it, type from the on-device map.

## Languages

| Layer | Language |
|---|---|
| Extension, capture, canvas sanitize, gate, executor | JS / TS |
| Train vision weights | Python -> ONNX |
| Remote reasoner | Python FastAPI + VLM |
