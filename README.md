# Fairway Tycoon

Design golf courses, run your club, host tournaments and play your own holes.
Works in any modern browser on phones, tablets and computers, and can be installed as an app.

## Controls
- **Phone or tablet:** one finger builds, paints and aims; two fingers pan and pinch to zoom.
- **Computer:** click and drag to build, paint and aim; scroll to zoom; right-click drag (or the Move tool) to pan.

## Updating the game
1. Replace `index.html` with the new version.
2. Open `sw.js` and change `VERSION` (for example `fairway-v1` to `fairway-v2`) so installed copies pick up the update.

Saves are stored in each browser on each device.

## 3D engine
The game uses three.js (r128), loaded from cdnjs. The offline helper caches it after the first visit.
On a computer: right-click drag or Q / E to rotate, arrow keys to pan, + / − to zoom.
On a phone: twist two fingers to rotate.
