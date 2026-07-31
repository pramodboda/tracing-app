# Tracer — AR Drawing Aid

A browser-based tracing tool, It overlays a reference image on a live camera feed so you can trace it directly onto paper or canvas — with adjustable opacity, scale, rotation, and blend modes to help you match proportions accurately.

No app store, no backend server, no account. It's just static web pages that use your camera and run entirely in the browser.

---

## What's in this project

| File               | What it's for                                                                                                                                                                            |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tracing-app.html` | **Single-device mode.** Uses the camera of whatever device it's open on, with the reference image overlaid on the same screen.                                                           |
| `camera.html`      | **Two-device mode — camera source.** Open this on the phone you're placing over your art surface. It streams that camera live to another device.                                         |
| `viewer.html`      | **Two-device mode — viewer.** Open this on the device you want to actually watch and control (iPad, laptop, another phone). It receives the phone's video and adds the overlay controls. |
| `index.html`       | A simple landing page linking to the two two-device pages.                                                                                                                               |

---

## Which mode should I use?

**Single-device mode (`tracing-app.html`)** — use this by default. One phone or tablet sits face-down over your paper; you look at that same screen while you draw. It's the simplest, fastest, and lowest-lag option, since everything happens on one device with no network involved.

**Two-device mode (`phone-camera.html` + `ipad-viewer.html`)** — use this only if you specifically want the camera on one device (e.g. phone flat on the desk) while you look at and control everything from a second device (e.g. an iPad propped up in front of you). This adds a small amount of video delay and requires both devices to have a working connection to each other.

---

## Setup

This is a static site — any static host works. The version you're using is deployed with **GitHub Pages**:

1. All files live in this repository.
2. GitHub Pages serves them at:
   `https://<your-username>.github.io/<repo-name>/`
3. Camera access requires **HTTPS** — GitHub Pages provides this automatically. Opening the files directly from disk (`file://...`) will not work; the camera simply won't be accessible.

After pushing changes to the repo, GitHub Pages typically rebuilds within about a minute.

---

## How to use

### Single-device mode

1. Open `tracing-app.html` on your phone or tablet (over HTTPS).
2. Allow camera access when prompted.
3. Tap **Image** and choose the artwork you want to trace.
4. Place the device face-down over your paper.
5. Adjust with the controls at the bottom:
   - **Opacity** — how visible the overlay is
   - **Scale / Rotate** — pinch and twist with two fingers, or use the sliders
   - **Blend** — try _Difference_ to precisely align edges before switching back to _Normal_ to trace
   - **Flip H / Flip V** — mirror the image
   - **Grid** — toggle a reference grid
   - **Lock Image** — freeze position/scale/rotation once you're happy with image placement, so accidental touches don't move it
   - **Zoom-in and Zoom-out view** - When the image is unlocked, drag/pinch behave exactly as before — moving and scaling the overlay. Once you tap Lock, those same gestures switch purpose: one-finger drag pans the whole view (camera + locked overlay together), and pinch zooms in/out on both together (1x–4x) — so you can lean in on fine detail without ever disturbing your placement. There's also a View Zoom slider in the dock for precise control without pinching, and a Reset View button to snap back to 1x centered.
   - **Switch** — toggle front/rear camera

### Two-device mode

1. Open `camera.html` on the phone or any device. Wait for it to display a 5-character **connection code**. Place the phone face-down over your art surface.
2. Open `viewer.html` on the second device. Enter the code and tap **Connect**.
3. Once connected, use the same controls described above (Image, Opacity, Scale, Rotate, Blend, Flip, Grid, Lock) from the second device — they apply to the video feed coming from the phone.
4. Use **Reconnect** if the link drops, or **Disconnect** to end the session and re-enter a code later.

### Gestures

- one finger drag = move image,
- two-finger pinch = scale,
- two-finger twist = rotate.
- The "Difference" blend mode is worth trying — it's the trick real tracing apps use: aligned lines turn black, misaligned ones glow, so you can nail proportions precisely before switching back to Normal and turning the opacity down to draw.

---

## Technical notes

- **No backend, no database.** All logic runs client-side in the browser. Settings (last image, opacity, scale, etc.) are saved locally on each device using `localStorage` — they do not sync between devices.
- **Two-device video** uses **WebRTC**, connected peer-to-peer via [PeerJS](https://peerjs.com/), with:
  - **STUN** (Google's public servers) for direct connections — free, low-latency, works when both devices can reach each other directly (e.g. same WiFi).
  - **TURN** (a public relay, [Open Relay Project](https://www.metered.ca/tools/openrelay/)) as a fallback for when a direct connection isn't possible (e.g. devices on different networks/carriers) — free, but relayed traffic adds some latency.
- Camera lag in single-device mode is effectively zero: the live camera plays natively in a `<video>` element, and the reference image floats on top as a separate CSS-transformed layer — nothing re-processes the video frames.

---

## Troubleshooting

- **Camera not showing / permission errors** — make sure you're on the `https://` GitHub Pages URL, not the `github.com` repo page, and not a locally opened file.
- **Two-device mode: "No response"** — usually means the two devices couldn't establish a route. Confirm the code was typed correctly, that the phone page is still open, and that you're not on a network that blocks WebRTC entirely (rare, but some corporate/campus WiFi does this).
- **Two-device mode: connects but no video** — if this recurs, it typically means the WebRTC negotiation didn't set up a video channel properly; check the browser console (`ICE state:` logs) for diagnostics.
- **Overlay drifting while tracing** — use a small stand or clip to keep the camera device steady; any wobble moves the whole projected image.

---

## Limitations

- Two-device connections rely on a shared public TURN relay, which is free but not guaranteed capacity — for heavier or more reliable use, swap in your own free TURN credentials (e.g. from metered.ca).
- Settings and images are per-device and don't sync automatically.
- This is a personal-use tool, not a production app — there's no error reporting, analytics, or update mechanism beyond redeploying the files.
