# Common — a canvas together

`index.html` is the entire app: one static HTML file with inline CSS and JavaScript. No build, account, backend deployment, or paid provisioning is needed. Serve it with any static HTTP(S) server, or download the file. Internet access is required to load the dependency and discover peers.

## Use
1. Enter your name and click **Create a room**.
2. Share the four uppercase letters. Other people open the same HTML/page, enter their name and code, and join.
3. Draw with mouse, pen, or touch. The legend labels each host-assigned color. Choose a brush width; save a PNG at any time.
4. Only the host can clear everyone's canvas. Leaving, refreshing, closing the host tab, or losing host connectivity ends the room. There is no host migration or persistence. Save PNGs before leaving.

## Dependency / connectivity
- External pinned script: `https://cdn.jsdelivr.net/npm/peerjs@1.5.5/dist/peerjs.min.js` (PeerJS 1.5.5).
- Default PeerJS public signaling service and default ICE configuration. Drawing travels over encrypted WebRTC data channels through a host-centered topology. The signaling service sees connection metadata, not application stroke storage. No app database or uploads.
- No paid TURN relay is provisioned. Restrictive NAT/firewalls, blocked CDN/signaling, and service outages can prevent connecting. A successful test on this LAN does **not** guarantee connectivity between arbitrary internet networks.
- Codes are a convenience, not authentication. Anyone knowing/guessing a code can join. Do not use for confidential drawings. Browser/background-tab throttling may trigger conservative heartbeat disconnects.
- Up to 24 identities per room lifetime, including departed people, preserving their unique colors. Maximum 20,000 line segments until the host clears. 16 KiB inbound JSON limit, coordinate/brush validation, 200 stroke messages/second/guest, bounded send queue, and host-authoritative colors/clear. These are defensive bounds, not production abuse protection.
- All geometry is normalized to a shared 4:3 surface, so different viewport sizes retain composition. Snapshot delivery is chunked, reliable and ordered; clear increments an epoch to reject old in-flight segments.

## Serve locally
```sh
cd /root/projects/shared-canvas
python -m http.server 8789 --bind 192.168.111.176
```
LAN URL: http://192.168.111.176:8789/ (private LAN only; not a public deployment).

## Verification
Tests reuse the installed Playwright at `/root/projects/reminders-calendar/node_modules/playwright`; update the require path for another machine, or install Playwright and use `require('playwright')`. These tests need Chromium and internet connectivity.
```sh
node tests/smoke.cjs
node tests/collaboration.cjs
node tests/edge-cases.cjs
```
Verified with **real browser pages in separate browser contexts using the default public PeerJS signaling service**, not a mocked transport or local signal server: create/join, distinct colors and safe literal names, two-way mouse drawing, identical shared stroke state, third-client late snapshot, 390px mobile layout, rejected guest clear and invalid coordinates/brush widths, host clear broadcast, guest disconnect legend, and host departure. No uncaught browser errors. Screenshots in `tests/desktop.png` and `tests/mobile.png`.

Additional edge tests passed: a real occupied signaling ID triggered automatic room-code collision retry; a real touch tap synchronized; an oversized inbound message disconnected its sender; and heartbeat timeout feedback appeared when the client's last-heartbeat timestamp was advanced to simulate an unresponsive host.

The browser clients ran on this same Linux host; cross-device/cross-NAT behavior has not been verified. The initial drawing test failed because its mouse coordinates were off-screen; scrolling the canvas into view fixed the test, and the full suite passed.

Optional **test-only** signaling override: `?signal=local&signalHost=127.0.0.1&signalPort=9001` uses an independently started PeerServer with path `/peerjs` and insecure WebSocket. No local signaling server is needed or installed for normal use; default behavior is unchanged. Do not use this override on HTTPS pages.

No GitHub repository was created or pushed. No existing service was modified.
