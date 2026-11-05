# MLR ex08 — Live Leaf Classifier (web)

**Live page: https://kovarski-c.github.io/mlr-ex08/**

Open it on a phone (or scan the QR code at the bottom of the page) and point the rear
camera at a leaf — the four class probabilities update live, entirely on-device in the
browser. A photo-upload fallback works anywhere.

## What it does
Classifies leaves into **apple, cherry, chestnut, maple** running in the browser via
[onnxruntime-web](https://github.com/microsoft/onnxruntime-web) (WASM, loaded from a CDN).
No server-side inference, no build step — a single `index.html` with plain JavaScript.

Two models sit next to the page and the buttons under the camera switch between them
(no reload, both want the same input tensor):

| Button | File | What it is |
|---|---|---|
| **Task 4** (default) | `leaf_model_task4.onnx` | pretrained MobileNetV3-Small, only the last layer trained (the best variant of Task 4) |
| **Task 2** | `leaf_model.onnx` | the network from the notebook, kept so the Task 2 numbers stay checkable |

`?model=<file>.onnx` loads any other model file instead and overrides the buttons.

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
| `index.html` | the whole app (UI, camera, model switch, preprocessing, inference, QR code) |
| `leaf_model_task4.onnx` | the Task 4 classifier — what the page runs by default (input `image` 1×3×96×96, output `logits` 1×4) |
| `leaf_model.onnx` | the Task 2 classifier, same input and output contract |
| `model_info.json` | model contract: classes, input size, mean, std (the same for both models) |
| `check_image_task4.jpg`, `check_expected_task4.json` | verification pair for the Task 4 model |
| `check_image.jpg`, `check_expected.json` | verification pair for the Task 2 model |
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
