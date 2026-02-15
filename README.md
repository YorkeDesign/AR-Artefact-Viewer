# AR Artefact Viewer - Technical README

**Project:** Christchurch Archaeology AR Artefact Viewer
**Website:** https://www.christchurcharchaeology.org/online-exhibition
**GitHub (3D Models):** https://github.com/christchurcharchaeology/3D-artefacts
**Developed by:** YORKE LABS

---

## Overview

This system allows users to view 3D scanned archaeological artefacts in an interactive web viewer, and then place them in their real-world environment using Augmented Reality (AR) on compatible mobile devices.

The solution is built using Google's `model-viewer` web component, embedded into Squarespace pages via custom HTML code blocks.

---

## How It Works: The Two Model Formats

Every artefact requires **two separate 3D model files** to support both iOS and Android devices:

| Format | Used By | Why |
|--------|---------|-----|
| `.glb` | Android devices + Web 3D viewer | Open standard, supported by Android's Scene Viewer and WebGL |
| `.usdz` | iPhone / iPad (iOS) AR | Apple's proprietary format required by iOS AR Quick Look |

### File Naming Convention

```
[ArtefactName]_LOD0.glb
[ArtefactName]_LOD0.usdz
```

Example:
```
MaskJug_LOD0.glb
MaskJug_LOD0.usdz
```

`LOD0` stands for Level of Detail 0 (highest detail). Files are stored in the GitHub repository at:
```
https://github.com/christchurcharchaeology/3D-artefacts
```

### Target File Sizes

| Target | Maximum |
|--------|---------|
| 2-4MB ideal | 15MB absolute maximum |

Files larger than ~15MB may fail to load on devices with low storage or slow connections.

---

## How the Files Are Hosted and Served

The 3D model files are hosted on GitHub and served via two different CDNs depending on the use case:

**GLB files (Android + Web viewer):**
```
https://cdn.jsdelivr.net/gh/christchurcharchaeology/3D-artefacts@main/[filename].glb
```
jsDelivr is used for GLB because it provides fast global CDN delivery with correct CORS headers for WebGL/Android Scene Viewer.

**USDZ files (iOS AR):**
```
https://raw.githubusercontent.com/christchurcharchaeology/3D-artefacts/main/[filename].usdz
```
GitHub's raw content delivery is used for USDZ because iOS AR Quick Look requires direct file access.

---

## The Web 3D Viewer

### What It Is

The web 3D viewer is powered by Google's `model-viewer` library (v3.3.0). It displays an interactive 3D model inside the browser — on both desktop and mobile — before the user enters AR mode.

### Which Model File It Uses

The web viewer **always uses the GLB file**, regardless of device. This is because GLB is an open web standard that works in all modern browsers via WebGL. The USDZ file is only used when AR Quick Look is launched on iOS.

### How It Loads

```html
<script type="module" src="https://ajax.googleapis.com/ajax/libs/model-viewer/3.3.0/model-viewer.min.js"></script>
```

The model-viewer library is loaded from Google's CDN. It provides the `<model-viewer>` custom HTML element used to display the 3D model.

### Key Attributes Explained

```html
<model-viewer
  src="[GLB URL]"              <!-- Web viewer model (GLB) - used on ALL devices -->
  ios-src="[USDZ URL]"         <!-- AR model for iOS only (USDZ) -->
  ar                           <!-- Enables the AR button -->
  ar-modes="webxr scene-viewer quick-look"  <!-- AR system priority order -->
  camera-controls              <!-- Allows user to rotate/zoom with mouse/touch -->
  touch-action="pan-y"         <!-- Allows vertical page scrolling on mobile -->
  auto-rotate                  <!-- Model slowly spins when idle -->
  camera-orbit="0deg 180deg 105%"  <!-- Initial camera position -->
  min-camera-orbit="auto auto 5%"  <!-- Minimum zoom -->
  max-camera-orbit="auto auto 200%"  <!-- Maximum zoom -->
  crossorigin="anonymous"      <!-- Handles cross-origin file requests -->
  loading="eager"              <!-- Loads model immediately -->
  reveal="auto">               <!-- Shows model as soon as loaded -->
```

### Camera Orientation

The `camera-orbit` attribute controls how the camera views the model initially:

```
camera-orbit="[horizontal rotation] [vertical angle] [zoom distance]"
```

| Value | Controls |
|-------|---------|
| First (e.g. `0deg`) | Horizontal rotation around the model |
| Second (e.g. `90deg`) | Vertical angle (90deg = straight on, 180deg = from below) |
| Third (e.g. `105%`) | Zoom distance from model |

**Important:** Currently the artefact models are exported with an incorrect orientation (lying on their side). This is compensated for by setting:
```html
camera-orbit="0deg 180deg 105%"
```
Once the model orientation is fixed in the export workflow, this should be changed to:
```html
camera-orbit="0deg 90deg 105%"
```

### Locking Vertical Rotation

To prevent the model from tilting up/down (which conflicts with page scrolling on mobile), the vertical angle can be locked:
```html
min-camera-orbit="auto 90deg auto"
max-camera-orbit="auto 90deg auto"
```
This restricts users to horizontal (turntable) rotation only. **Note:** This only works correctly when models have proper upright orientation. Currently disabled for the test model due to orientation issue.

---

## How the AR Button Works

### What Triggers AR

The `ar` attribute on `<model-viewer>` causes the library to automatically detect if the user's device supports AR. If it does, a "View in AR" button appears. If not (e.g. desktop browser), the button is hidden automatically.

### Our Custom AR Button

We replace the default model-viewer AR button with our own styled button:

```html
<button slot="ar-button" style="
  position: absolute;
  top: 20px;
  right: 20px;
  background: #4285f4;
  ...">
  View in AR
</button>
```

The button is positioned in the **top-right corner** so it is always visible regardless of device font size settings (accessibility consideration - users with large text settings could not see a bottom-positioned button without scrolling).

### AR Mode Priority

```html
ar-modes="webxr scene-viewer quick-look"
```

The viewer tries AR systems in this order:
1. **webxr** - Modern web-based AR (limited browser support)
2. **scene-viewer** - Android's native AR (Google Scene Viewer)
3. **quick-look** - iOS's native AR (Apple AR Quick Look)

---

## AR on iOS (iPhone / iPad)

### What Happens

When an iOS user taps "View in AR":
1. The browser hands control to **Apple AR Quick Look** (native iOS system)
2. AR Quick Look downloads and renders the **USDZ file** (specified in `ios-src`)
3. The camera activates and the artefact is placed in the real world
4. Apple's own UI is displayed - we have no control over this interface

### iOS AR Requirements

- iPhone 6s or later (iOS 12+)
- Safari or Chrome browser (NOT Instagram/Facebook in-app browser)
- Adequate free storage (minimum ~5GB recommended - devices with less than ~2GB may fail to load textures)

### iOS AR Limitations

- We cannot customise the AR Quick Look interface
- Photo/video capture uses Apple's built-in screenshot (side buttons) or screen recording
- In-app browsers (Instagram, Facebook) do not support AR Quick Look

### Model Orientation in iOS AR

iOS AR Quick Look automatically corrects the model's vertical orientation, so even if the GLB model appears sideways in the web viewer, the USDZ typically appears upright in iOS AR.

---

## AR on Android

### What Happens

When an Android user taps "View in AR":
1. The browser launches **Google Scene Viewer** (native Android AR system)
2. Scene Viewer downloads and renders the **GLB file** (the same file used in the web viewer)
3. The camera activates and the artefact is placed in the real world
4. Google's own UI is displayed - we have no control over this interface

### Android AR Requirements

- Android 7.0+ with ARCore support
- Google Play Services for AR installed and enabled
- Chrome browser (NOT Instagram/Facebook in-app browser)
- Camera permissions granted to the browser

### Android AR Limitations

- We cannot customise the Scene Viewer interface
- Some devices have AR settings restricted by corporate/work profiles (MDM)
- Samsung's Knox security can block AR functionality
- In-app browsers (Instagram, Facebook) do not support Scene Viewer

---

## In-App Browser Handling (Instagram / Facebook)

When users open the page from within Instagram or Facebook, those apps use their own restricted browser engine that does not support AR. The `model-viewer` library automatically detects this and hides the "View in AR" button.

### Our Solution

We detect in-app browsers using JavaScript and display a prominent button that opens the page in the device's default browser:

```javascript
function isInAppBrowser() {
  const ua = navigator.userAgent || navigator.vendor || window.opera;
  if (ua.indexOf('Instagram') > -1) return true;
  if (ua.indexOf('FBAN') > -1 || ua.indexOf('FBAV') > -1) return true;
  // ... etc
}
```

**Detected apps:**
- Instagram
- Facebook
- LinkedIn
- Twitter/X
- Line
- Snapchat
- TikTok

When detected, a blue "Click Here to Open in Browser for AR" button appears. Clicking it:
- **iOS:** Attempts to open in Safari using `x-safari-` URL scheme
- **Android:** Uses Android Intent URL to open in default browser

---

## Known Issues and Workarounds

| Issue | Cause | Workaround |
|-------|-------|------------|
| Model appears sideways in web viewer | Incorrect export orientation from Aspose USD→GLB conversion | Set `camera-orbit="0deg 180deg 105%"` as compensation. Fix properly by rotating model in Blender before export |
| AR fails on device with very low storage | iOS restricts AR Quick Look when <2GB free | Ask user to free up storage space |
| AR not working in Brave browser | Brave Shields blocks CDN scripts and cross-origin requests | Ask user to disable Shields for the site, or use Safari/Chrome |
| Purple/missing textures in AR | Texture loading failure (often storage-related) | Ensure USDZ has embedded textures, reduce file size |
| AR button not visible on large font settings | Button was at bottom of viewer, pushed off screen | Fixed - button now positioned top-right |
| Android AR not launching | Google Play Services for AR not installed, or restricted by MDM/Knox | Install ARCore from Play Store, or use different device |

---

## Complete Code Template

Use this template for each artefact page. Replace `[FILENAME]` and `[ARTEFACT NAME]` for each page:

```html
<style>
  .sqs-block,
  .sqs-block-content,
  .code-block,
  .page-section,
  section[data-section-id] {
    padding-top: 0 !important;
    margin-top: 0 !important;
  }
  .content-wrapper,
  .content {
    padding-top: 0 !important;
  }
</style>

<script type="module" src="https://ajax.googleapis.com/ajax/libs/model-viewer/3.3.0/model-viewer.min.js"></script>

<div style="padding-top: 30px;">
  <a href="/online-exhibition" style="
    display: inline-block;
    margin-bottom: 20px;
    padding: 10px 20px;
    background: #f5f5f5;
    color: #333;
    text-decoration: none;
    border-radius: 6px;
    font-family: Arial, sans-serif;
    font-size: 14px;
    font-weight: 500;
    border: 1px solid #ddd;">
    ← Back to Gallery
  </a>
</div>

<div id="browser-button" style="display: none; margin-bottom: 20px; text-align: center;">
  <a id="open-browser-btn" href="" style="
    display: inline-block;
    padding: 14px 28px;
    background: #4285f4;
    color: white;
    text-decoration: none;
    border-radius: 8px;
    font-family: Arial, sans-serif;
    font-size: 16px;
    font-weight: 600;
    box-shadow: 0 2px 8px rgba(0,0,0,0.2);
    text-align: center;">
    Click Here to Open in Browser for AR
  </a>
</div>

<div style="width: 100%; max-width: 100%; margin: 0 auto; position: relative;">
  <model-viewer
    src="https://cdn.jsdelivr.net/gh/christchurcharchaeology/3D-artefacts@main/[FILENAME].glb"
    ios-src="https://raw.githubusercontent.com/christchurcharchaeology/3D-artefacts/main/[FILENAME].usdz"
    alt="[ARTEFACT NAME]"
    ar
    ar-modes="webxr scene-viewer quick-look"
    camera-controls
    touch-action="pan-y"
    auto-rotate
    camera-orbit="0deg 90deg 105%"
    min-camera-orbit="auto 90deg auto"
    max-camera-orbit="auto 90deg auto"
    crossorigin="anonymous"
    loading="eager"
    reveal="auto"
    style="width: 100%; height: 600px; background-color: #f5f5f5; display: block; position: relative;">

    <button slot="ar-button" style="
      position: absolute;
      top: 20px;
      right: 20px;
      background: #4285f4;
      color: white;
      padding: 14px 28px;
      border: none;
      border-radius: 8px;
      font-family: Arial, sans-serif;
      font-size: 16px;
      font-weight: 600;
      cursor: pointer;
      box-shadow: 0 2px 8px rgba(0,0,0,0.2);
      z-index: 10;">
      View in AR
    </button>
  </model-viewer>

  <div style="
    position: absolute;
    top: 20px;
    left: 20px;
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    font-size: 10px;
    color: #546e7a;
    opacity: 0.7;
    letter-spacing: 0.2px;
    line-height: 1.2;
    z-index: 5;">
    Powered by <span style="font-weight: 600; letter-spacing: 0.4px;">YORKE</span> <span style="font-weight: 300;">LABS</span>
  </div>
</div>

<style>
  model-viewer {
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    min-height: 600px !important;
  }
  model-viewer:not(:defined) {
    display: block;
    height: 600px;
    background: #f5f5f5;
  }
  a:hover { background: #e8e8e8 !important; }
  #open-browser-btn:hover { background: #3367d6 !important; }
</style>

<script>
function isInAppBrowser() {
  const ua = navigator.userAgent || navigator.vendor || window.opera;
  if (ua.indexOf('Instagram') > -1) return true;
  if (ua.indexOf('FBAN') > -1 || ua.indexOf('FBAV') > -1) return true;
  if (ua.indexOf('LinkedInApp') > -1) return true;
  if (ua.indexOf('Twitter') > -1) return true;
  if (ua.indexOf('Line/') > -1) return true;
  if (ua.indexOf('Snapchat') > -1) return true;
  if (ua.indexOf('TikTok') > -1) return true;
  return false;
}

if (isInAppBrowser()) {
  document.getElementById('browser-button').style.display = 'block';
  const currentUrl = window.location.href;
  const btn = document.getElementById('open-browser-btn');
  const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent);
  if (isIOS) {
    btn.href = 'x-safari-' + currentUrl;
  } else {
    btn.href = 'intent://' + currentUrl.replace(/^https?:\/\//, '') + '#Intent;scheme=https;end';
  }
}
</script>
```

**Note:** The template above uses `camera-orbit="0deg 90deg 105%"` with locked vertical rotation. This assumes correctly oriented models. For models with the current orientation issue, change to `camera-orbit="0deg 180deg 105%"` with `min-camera-orbit="auto auto 5%"` and `max-camera-orbit="auto auto 200%"`.

---

## Model Export Workflow (Important)

### Current Issue
Models are currently exported with incorrect orientation (lying on their side). This is a known issue with the USDZ → USD → GLB conversion via Aspose.

### Correct Workflow for Future Models
1. Scan artefact using photogrammetry software
2. Export as USDZ from scanning software
3. Unzip USDZ (rename to .zip, extract) to get USD file
4. Convert USD → GLB using Aspose OpenUSD converter
5. Import resulting GLB into Blender
6. Rotate to correct upright orientation: **R → X → -90 → Enter** (or as needed)
7. Export as GLB from Blender with **+Y Up checked**
8. Verify orientation in https://gltf-viewer.donmccurdy.com/
9. Upload both GLB and USDZ to GitHub repository

### Recommended File Size Targets
- **USDZ:** 2-4MB (iOS AR)
- **GLB:** 2-6MB (Android AR + Web viewer)
- Texture resolution: 1024x1024 or 2048x2048

---

## Device Testing Results

| Device | Web Viewer | iOS AR | Android AR | Notes |
|--------|-----------|--------|------------|-------|
| iPhone 13 Pro | ✅ | ✅ | N/A | Primary test device |
| iPhone 14 Plus (low storage) | ✅ | ❌ | N/A | Only 1.8GB free - storage issue |
| Samsung S24 | ✅ | N/A | ❌ | AR restricted by device policy |
| ~14 other devices | ✅ | ✅ | ✅ | All working correctly |

---

## Squarespace Integration Notes

- **Plan required:** Business plan or higher (for custom code blocks)
- **Code blocks:** Each artefact page uses a single HTML code block
- **Padding fix:** CSS overrides are required to remove Squarespace's default section padding (`padding-top: 0 !important`)
- **Page structure:**
  - Gallery page: `/online-exhibition` (grid of artefact thumbnails)
  - Individual pages: `/artefact-001` through `/artefact-006` (not linked in main navigation)
- **Page naming:** Artefact 001, Artefact 002 etc. (numbered for future-proofing QR codes)

---

## Branding

- **Credit:** "Powered by YORKE LABS" displayed top-left of viewer
- **AR Button colour:** `#4285f4` (Google Blue)
- **Credit colour:** `#546e7a` at 70% opacity (subtle navy-grey)
- **Font:** Inter / system sans-serif

---

## Key URLs

| Resource | URL |
|----------|-----|
| Gallery page | https://www.christchurcharchaeology.org/online-exhibition |
| GitHub repo | https://github.com/christchurcharchaeology/3D-artefacts |
| GLB CDN base URL | `https://cdn.jsdelivr.net/gh/christchurcharchaeology/3D-artefacts@main/` |
| USDZ raw base URL | `https://raw.githubusercontent.com/christchurcharchaeology/3D-artefacts/main/` |
| model-viewer docs | https://modelviewer.dev |
| GLB viewer (testing) | https://gltf-viewer.donmccurdy.com |
