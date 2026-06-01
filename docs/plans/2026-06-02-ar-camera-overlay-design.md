# AR Camera Overlay QR Launch Design

Date: 2026-06-02
Project: DREAMCATCHER interactive product guide

## Goal

Add an AR-style camera experience to the existing single-page site. A customer scans a physical QR Code, opens the website in AR mode, grants camera access, and sees an overlay that points to the vacuum cleaner power button.

The final visual assets are not ready yet, so the first version uses built-in placeholder UI: an arrow, a red power-button marker, and short Chinese instructions. The placeholder should be easy to replace with image assets later.

## Current Context

The project is a static site with one main file, `index.html`. It already contains an interactive SVG product simulator and a red power-button hotspot for step 1. There is no build system or framework. Styling uses Tailwind from CDN, local CSS, Font Awesome, and inline JavaScript.

There is one untracked JPG file in the repository root. It is not treated as the final AR asset in this design.

## Recommended Approach

Use a browser camera overlay instead of full marker-based WebAR.

The QR Code should point to a URL such as `index.html?mode=ar`. When the page detects this query parameter, it offers or starts the AR camera overlay. Users can also enter the same mode from a new visible button on the normal page.

This approach is recommended because it matches the requested user experience, can be implemented in the current static page, and does not require 3D models, marker assets, or external AR libraries.

## User Flow

1. User scans the physical QR Code.
2. Browser opens `index.html?mode=ar`.
3. Site shows an AR camera intro state and requests camera permission after a user gesture if the browser requires it.
4. If permission is granted, the rear camera appears full-screen.
5. A UI overlay points toward the expected power-button area with placeholder arrow, red marker, and text: `對準機身，按紅色電源鍵開機`.
6. User can close AR mode and return to the normal guide.

## Components

### AR Entry Button

Add a button in the existing header or first-step content: `開啟 AR 教學`. This opens the same AR overlay used by the QR flow.

### AR Overlay

Add a full-screen fixed layer containing:

- A `<video>` element for the camera stream.
- A dark translucent top bar with title and close button.
- A centered instruction layer with an arrow, red power-button marker, and concise text.
- A fallback/error card for denied permission, missing camera, or unsupported browser.

### Camera Controller

Add JavaScript functions to:

- Start camera with `navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' }, audio: false })`.
- Attach the stream to the AR video element.
- Stop all tracks when AR mode closes.
- Detect `?mode=ar` on load and prepare AR entry.

## Error Handling

If the browser does not support camera access, show: `此瀏覽器不支援相機 AR 模式，請改用下方互動教學。`

If the user denies permission, show: `請允許相機權限後重新開啟 AR 教學。`

If the camera fails for another reason, show a general fallback and keep the existing interactive guide usable.

## Testing

Manual verification should cover:

- Normal page still loads and existing tutorial controls work.
- `?mode=ar` shows the AR entry path.
- Clicking AR entry requests camera permission on a supported device.
- Closing AR mode stops the camera stream.
- Permission denied and unsupported browser states show readable fallback text.
- Mobile layout remains usable in portrait orientation.

## Out of Scope

This first version does not perform real QR recognition inside the page, object tracking, 3D placement, or product-shape recognition. It also does not require final AR artwork. Those can be added later if marker assets or product images become available.
