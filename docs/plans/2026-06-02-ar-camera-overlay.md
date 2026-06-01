# AR Camera Overlay Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add an AR-style camera overlay mode that users can open from a QR Code URL or from the existing DREAMCATCHER guide.

**Architecture:** Keep the current static `index.html` architecture. Add a full-screen AR overlay, a visible entry button, and small camera controller functions that start/stop `getUserMedia` streams without introducing external dependencies.

**Tech Stack:** Static HTML, Tailwind CDN classes, local CSS, vanilla JavaScript, browser MediaDevices API.

---

## Context

Design doc: `docs/plans/2026-06-02-ar-camera-overlay-design.md`

Current site: `index.html` is a single-page static product guide. It already has a red power-button hotspot and step-1 copy about pressing the red power button. Do not replace the current guide. Add AR as an optional mode.

Important constraints:

- Final AR artwork is not ready. Use placeholder HTML/CSS arrow, red marker, and Chinese instruction copy.
- Do not add QR scanning inside the page. The QR Code should open a URL like `index.html?mode=ar`.
- Camera access requires HTTPS or localhost in most browsers.
- Stop camera tracks when closing AR mode.
- Do not add a build system or framework.

## Task 1: Add AR Entry UI

**Files:**
- Modify: `index.html`

**Step 1: Add a header AR button**

In the header action button area near `sound-toggle` and `restartDemo`, add a button:

```html
<button onclick="openArGuide()" class="hidden sm:flex items-center space-x-1 text-xs text-emerald-700 bg-emerald-50 px-3 py-2 rounded-xl font-semibold hover:bg-emerald-100 transition-colors">
    <i class="fa-solid fa-camera"></i>
    <span>AR 教學</span>
</button>
```

**Step 2: Add a mobile-visible AR button in step content**

Below the `instruction-tip` block, add a compact button:

```html
<button onclick="openArGuide()" class="inline-flex sm:hidden items-center justify-center gap-2 rounded-xl bg-emerald-600 px-4 py-3 text-xs font-extrabold text-white shadow-md shadow-emerald-500/20 hover:bg-emerald-700 transition-colors">
    <i class="fa-solid fa-camera"></i>
    <span>開啟 AR 相機教學</span>
</button>
```

**Step 3: Manual check**

Open the page and confirm:

- Desktop header shows `AR 教學`.
- Mobile-width layout shows `開啟 AR 相機教學` near the first instruction area.
- Existing `重新導覽` and sound buttons still display correctly.

## Task 2: Add AR Overlay Markup and Styles

**Files:**
- Modify: `index.html`

**Step 1: Add overlay markup before the main script**

Place this before `<!-- Script Controllers & Dynamic Physics Engine -->`:

```html
<div id="ar-overlay" class="fixed inset-0 z-[100] hidden bg-slate-950 text-white">
    <video id="ar-video" class="absolute inset-0 h-full w-full object-cover" autoplay playsinline muted></video>

    <div class="absolute inset-x-0 top-0 z-20 bg-gradient-to-b from-slate-950/85 to-transparent p-4">
        <div class="flex items-center justify-between">
            <div>
                <p class="text-[10px] font-extrabold uppercase tracking-[0.24em] text-emerald-300">DREAMCATCHER AR</p>
                <h2 class="text-lg font-black">開機按鈕定位教學</h2>
            </div>
            <button onclick="closeArGuide()" class="rounded-full bg-white/10 px-3 py-2 text-xs font-bold text-white backdrop-blur hover:bg-white/20">
                關閉
            </button>
        </div>
    </div>

    <div id="ar-live-layer" class="absolute inset-0 z-10 hidden pointer-events-none">
        <div class="absolute left-1/2 top-[48%] flex -translate-x-1/2 -translate-y-1/2 flex-col items-center gap-3">
            <div class="rounded-full border-4 border-red-400 bg-red-500/40 p-5 shadow-[0_0_40px_rgba(248,113,113,0.8)]">
                <div class="h-5 w-5 rounded-full bg-red-500 ring-4 ring-white"></div>
            </div>
            <div class="rounded-2xl bg-slate-950/80 px-4 py-3 text-center text-sm font-extrabold shadow-xl backdrop-blur">
                對準機身，按紅色電源鍵開機
            </div>
            <div class="h-20 w-1 rounded-full bg-emerald-300 shadow-[0_0_24px_rgba(110,231,183,0.9)]"></div>
            <i class="fa-solid fa-arrow-down text-3xl text-emerald-300 drop-shadow-lg"></i>
        </div>
    </div>

    <div id="ar-message-card" class="absolute left-4 right-4 top-1/2 z-30 mx-auto max-w-sm -translate-y-1/2 rounded-3xl bg-white p-6 text-center text-slate-900 shadow-2xl">
        <div class="mx-auto mb-4 flex h-12 w-12 items-center justify-center rounded-2xl bg-emerald-50 text-emerald-600">
            <i class="fa-solid fa-camera text-xl"></i>
        </div>
        <h3 id="ar-message-title" class="mb-2 text-lg font-black">準備開啟 AR 相機</h3>
        <p id="ar-message-text" class="mb-5 text-sm leading-relaxed text-slate-500">請允許相機權限，畫面會疊上開機按鈕指示。</p>
        <button id="ar-start-button" onclick="startArCamera()" class="w-full rounded-xl bg-emerald-600 px-4 py-3 text-sm font-extrabold text-white hover:bg-emerald-700">
            開啟相機
        </button>
    </div>
</div>
```

**Step 2: Manual check**

Temporarily remove `hidden` from `#ar-overlay` in dev tools and confirm:

- Overlay covers the viewport.
- Close button is visible.
- Placeholder red marker and text are readable.
- Intro card is centered.

## Task 3: Add Camera Controller JavaScript

**Files:**
- Modify: `index.html`

**Step 1: Add AR state near existing state variables**

Near existing variables like `productColor`, add:

```javascript
let arStream = null;
```

**Step 2: Add AR functions before `window.onload`**

Add:

```javascript
function openArGuide() {
    const overlay = document.getElementById('ar-overlay');
    const liveLayer = document.getElementById('ar-live-layer');
    const messageCard = document.getElementById('ar-message-card');
    const title = document.getElementById('ar-message-title');
    const text = document.getElementById('ar-message-text');
    const startButton = document.getElementById('ar-start-button');

    overlay.classList.remove('hidden');
    liveLayer.classList.add('hidden');
    messageCard.classList.remove('hidden');
    title.innerText = '準備開啟 AR 相機';
    text.innerText = '請允許相機權限，畫面會疊上開機按鈕指示。';
    startButton.classList.remove('hidden');
}

async function startArCamera() {
    const liveLayer = document.getElementById('ar-live-layer');
    const messageCard = document.getElementById('ar-message-card');
    const title = document.getElementById('ar-message-title');
    const text = document.getElementById('ar-message-text');
    const startButton = document.getElementById('ar-start-button');
    const video = document.getElementById('ar-video');

    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
        title.innerText = '此瀏覽器不支援相機';
        text.innerText = '請改用手機瀏覽器，或直接使用下方互動教學。';
        startButton.classList.add('hidden');
        return;
    }

    try {
        arStream = await navigator.mediaDevices.getUserMedia({
            video: { facingMode: 'environment' },
            audio: false
        });
        video.srcObject = arStream;
        await video.play();
        messageCard.classList.add('hidden');
        liveLayer.classList.remove('hidden');
    } catch (error) {
        console.warn('AR camera error', error);
        title.innerText = '無法開啟相機';
        text.innerText = '請允許相機權限後重新開啟 AR 教學。若仍無法使用，請改看下方互動教學。';
        startButton.classList.remove('hidden');
    }
}

function closeArGuide() {
    const overlay = document.getElementById('ar-overlay');
    const video = document.getElementById('ar-video');

    if (arStream) {
        arStream.getTracks().forEach(track => track.stop());
        arStream = null;
    }

    video.pause();
    video.srcObject = null;
    overlay.classList.add('hidden');
}

function initArModeFromUrl() {
    const params = new URLSearchParams(window.location.search);
    if (params.get('mode') === 'ar') {
        openArGuide();
    }
}
```

**Step 3: Update `window.onload`**

Change:

```javascript
window.onload = function() {
    initPowerButtonEvents();
    goToStep(1);
}
```

To:

```javascript
window.onload = function() {
    initPowerButtonEvents();
    goToStep(1);
    initArModeFromUrl();
}
```

**Step 4: Manual check**

Open `index.html?mode=ar` and confirm the AR intro card appears after load.

## Task 4: Verify Existing Tutorial Still Works

**Files:**
- Verify: `index.html`

**Step 1: Run local static server**

Run one of these from repo root:

```bash
python -m http.server 8000
```

or use another available static server.

**Step 2: Browser checks**

Visit:

- `http://localhost:8000/index.html`
- `http://localhost:8000/index.html?mode=ar`

Verify:

- Existing next/previous tutorial buttons still work.
- Power-button short click and long press behavior still work.
- AR button opens overlay.
- Close button hides overlay.
- On a camera-capable browser, `開啟相機` starts the camera.
- On unsupported/denied camera, fallback copy appears.

## Task 5: Final Review

**Files:**
- Review: `index.html`
- Review: `docs/plans/2026-06-02-ar-camera-overlay-design.md`
- Review: `docs/plans/2026-06-02-ar-camera-overlay.md`

**Step 1: Confirm scope**

Ensure the implementation only adds optional AR overlay behavior and does not remove or rewrite the existing guide.

**Step 2: Check for stream cleanup**

Confirm `closeArGuide()` calls `track.stop()` for every active camera track.

**Step 3: Check git diff**

Run:

```bash
git diff -- index.html docs/plans/2026-06-02-ar-camera-overlay-design.md docs/plans/2026-06-02-ar-camera-overlay.md
```

Expected: only AR UI, AR controller code, and planning docs changed.

## Notes for Commit

Only commit if the user explicitly asks for a commit. Suggested commit message:

```bash
git add index.html docs/plans/2026-06-02-ar-camera-overlay-design.md docs/plans/2026-06-02-ar-camera-overlay.md
git commit -m "feat: add AR camera guide plan"
```
