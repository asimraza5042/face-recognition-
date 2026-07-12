# Face Recognition Scanner

A fully client-side, in-browser face detection and recognition app. No backend, no server, no data ever leaves the device — everything runs locally in the browser using WebRTC and TensorFlow.js.

Open `index.html` in a browser, allow camera access, enroll a few faces, and it will detect and recognize them live, similar in spirit to how Google Photos surfaces "who is this?" for unrecognized faces.

## Features

- **Live face detection** via webcam, with bounding-box overlays drawn in real time
- **Face recognition/matching** against enrolled people, with a confidence percentage
- **Two ways to enroll a face**: live camera capture, or uploading a photo
- **"New Faces Detected" panel** — automatically surfaces thumbnails of unrecognized faces so you can name and add them on the spot (a quick "new entry" flow, like Google Photos' face suggestions)
- **Frame-to-frame label smoothing** (simple tracking + majority vote) to reduce flicker and give steadier, more confident-feeling recognition
- **Multi-sample enrollment** — add several captures per person for more robust matching
- **Recognition log** with timestamps
- Fully in-memory — no data is stored, uploaded, or persisted anywhere

## What's used in it

- **HTML5** — page structure and semantic layout
- **CSS3** — custom properties (CSS variables) for theming, CSS Grid/Flexbox for layout, animations for the scanning-line and status-dot effects
- **JavaScript (vanilla, ES2017+)** — async/await, no framework or build step required
- **[face-api.js](https://github.com/justadudewhohacks/face-api.js)** (v0.22.2, via CDN) — the face detection/recognition library, built on top of TensorFlow.js
- **Pretrained models** (loaded from the [vladmandic/face-api](https://github.com/vladmandic/face-api) model weights on jsDelivr):
  - `TinyFaceDetector` — fast face detection
  - `FaceLandmark68Net` — 68-point facial landmark detection
  - `FaceRecognitionNet` — generates 128-dimensional face descriptors used for matching
- **Browser Web APIs**:
  - `navigator.mediaDevices.getUserMedia` — webcam access
  - `Canvas API` — drawing overlays, cropping face thumbnails, image analysis
  - `FileReader API` — reading uploaded photos for the "enroll from photo" flow
  - Drag-and-drop file API — for the photo upload dropzone
- **Google Fonts** — `Space Mono` (HUD/technical readouts) and `Inter` (body text)
- **jsDelivr CDN** — used to load face-api.js and the model weights, no npm install needed

## How it works

1. On load, the three face-api.js models are fetched and initialized, and the webcam stream starts.
2. Every animation frame, the video feed is run through the detector to find faces, extract landmarks, and compute a face descriptor (a numeric "fingerprint") for each one.
3. Each descriptor is compared against enrolled people using Euclidean distance (via face-api.js's `FaceMatcher`). A close-enough match is labeled with that person's name and a confidence score; otherwise it's marked "Unknown."
4. A lightweight tracker links faces across frames (by proximity) and keeps a short rolling buffer of recent labels per tracked face, using the majority vote as the displayed label — this smooths out momentary misreads.
5. Faces that consistently read as "Unknown" and are meaningfully different from anyone already enrolled or pending trigger a suggestion card with a thumbnail, so they can be named and added in one click.
6. Enrollment (from camera or photo) stores the face descriptor in memory, tagged with a name, and rebuilds the matcher.

## Running it

No build step or server required:

1. Download/clone the repo
2. Open `index.html` directly in a modern browser (Chrome, Edge, or Firefox recommended), or serve the folder with any static file server
3. Allow camera access when prompted

## Limitations

- All enrolled faces are stored in memory only — refreshing the page clears them (by design, to keep everything local and private)
- Requires reasonable lighting and a working webcam
- This is face *matching* against people you enroll yourself, not identification against any external database
- Recognition accuracy depends on model quality/lighting/angle — for production use cases, a more robust pipeline (e.g. face alignment, liveness detection, server-side model) would be recommended

## License

MIT — free to use, modify, and distribute.
