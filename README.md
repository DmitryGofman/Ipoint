# IMU Point Controller — ZUPT Prototype

A browser-based prototype that turns a phone's motion sensors into a 3D point
controller. Hold the phone still and the point **LOCKs**; nudge the phone in
any direction and the point is pushed along, integrating each gesture on its
own before drift is reset to zero at rest (Zero-Velocity Update, "ZUPT").

## How it works

The prototype reads the OS-fused device orientation plus gravity-free
acceleration and runs a small inertial pipeline in the browser:

- **Bias estimation** — while resting (LOCK), a slow EMA tracks the
  device-frame accelerometer bias so residual offsets don't accumulate.
- **Gesture detection** — leaving the deadband (with hysteresis) starts a
  gesture and switches to TRACK.
- **Integration** — device-frame acceleration is rotated into world space,
  integrated to velocity (with a leak term to tame residuals), then to
  position with an adjustable gain.
- **ZUPT** — once the phone is quiet again for the configured *rest hold*
  time, velocity is zeroed and the state returns to LOCK. This is what keeps
  double-integration drift from running away.

The scene (Three.js) shows the point, a trail, a drop line, and a floor disc
for depth perception. A one-finger drag orbits the camera.

## Controls

| Control | Effect |
| --- | --- |
| **Gain** | Scales integrated position — how far a gesture moves the point. |
| **Deadband** | Acceleration threshold (m/s²) below which motion is ignored. |
| **Rest hold** | How long the phone must be still before ZUPT re-locks. |
| **Magnetometer** | Anchors yaw to the compass (absolute orientation) when supported. |
| **Trail** | Toggles the motion trail. |
| **Re-zero bias** | Clears the estimated accelerometer bias. |
| **Center point** | Resets the point to the origin and re-locks. |
| **Clear trail** | Empties the trail buffer. |

## Running it

Motion and orientation sensors require **HTTPS** and a **real page** (not an
embedded iframe), and they only produce data on a phone.

- Open `index.html` on a phone served over HTTPS.
- Tap **Enable motion sensors** and grant the permission prompts (iOS
  requires an explicit user gesture to request motion access).

Quick local HTTPS options: serve the folder with any static HTTPS server, or
push to a static host (e.g. GitHub Pages) and open the URL on your phone.

## Files

- `index.html` — the entire self-contained prototype (Three.js loaded from CDN).

## Notes

- Double integration of consumer IMU data drifts quickly; the whole design
  leans on ZUPT plus a velocity leak to stay usable. It's a feel/interaction
  prototype, not a metrology-grade tracker.
- Absolute (compass-anchored) yaw drifts near metal and isn't exposed by
  every browser; the app falls back to relative orientation when it isn't.
