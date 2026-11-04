# MLR ex08 — Live Leaf Classifier (web)

**Live page: https://kovarski-c.github.io/mlr-ex08/**

Open it on a phone (or scan the QR code at the bottom of the page) and point the rear
camera at a leaf — the four class probabilities update live, entirely on-device in the
browser. A photo-upload fallback works anywhere.

## What it does
Classifies leaves into **apple, cherry, chestnut, maple** with `leaf_model.onnx`, running
in the browser via [onnxruntime-web](https://github.com/microsoft/onnxruntime-web) (WASM,
loaded from a CDN). No server-side inference, no build step — a single `index.html` with
plain JavaScript.

Preprocessing follows `model_info.json` exactly:

1. take the **central square** of the camera frame (or of the uploaded photo),
2. resize it to **96 × 96**,
3. scale RGB pixels to **0–1**,
4. standardize each channel with `mean = [0.485, 0.456, 0.406]`, `std = [0.229, 0.224, 0.225]`,
5. feed the tensor `1 × 3 × 96 × 96` (float32, RGB, CHW) to the model,
6. apply **softmax** to the model's `logits` output to get class probabilities.

## Files
| File | Purpose |
|---|---|
| `index.html` | the whole app (UI, camera, preprocessing, inference, QR code) |
| `leaf_model.onnx` | the classifier (input `image` 1×3×96×96, output `logits` 1×4) |
| `model_info.json` | model contract: classes, input size, mean, std |
| `check_image.jpg`, `check_expected.json` | verification pair — uploading the image must reproduce the expected probabilities |
| `vendor/qrcode.js` | [qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator) 1.4.4 (MIT) — QR is drawn on a canvas **in the browser**, no external QR service |

## Notes
* Works in Safari on iPhone: `playsinline` video, `{facingMode: {ideal: 'environment'}}`,
  single-threaded WASM (`ort.env.wasm.numThreads = 1`) because GitHub Pages does not send
  COOP/COEP headers (no `SharedArrayBuffer`). Requires a browser with WASM SIMD
  (iOS/iPadOS 16.4+, recent Chrome/Firefox).
* Camera access needs HTTPS (provided by GitHub Pages) and one tap on **Start camera**
  (iOS requires a user gesture).
* Verified: the preprocessing pipeline reproduces `check_expected.json` for
  `check_image.jpg` to within 1e-6.

Course exercise (Czech Technical University, MLR), fall 2026.
