# PMV Sync

A single-page, no-build web app that syncs a Bluetooth sex toy to a video.
Built for the **JoyHub MarsLion**, but it works with anything
[Buttplug.io](https://buttplug.io) supports (hundreds of toys).

Open `index.html` in Chrome or Edge. Nothing is uploaded anywhere: the video,
the script and the audio analysis all stay in your browser, and the only
network connection is a websocket to Intiface Central on your own machine.

## Setup (once)

1. Install [Intiface Central](https://intiface.com/central/) (free, open source,
   Windows/macOS/Linux/Android/iOS). It is the bridge between the browser and
   Bluetooth.
2. Make sure the JoyHub app is fully closed. A toy holds one Bluetooth
   connection at a time.
3. In Intiface Central press **Start Server**. The default address is
   `ws://127.0.0.1:12345`, which is what the app uses.

## Each session

1. Open `index.html`, press **Connect**, then **Scan for toys**. Switch the
   toy on. It shows up as *JoyHub MarsLion* with two outputs:
   **Vibrate** (255 steps) and **Constrict · Air Pump** (5 steps). Those come
   straight from Buttplug's device database, so other toys show their own
   outputs.
2. Drop a video onto the page. Drop a `.funscript` with it if you have one.
3. Pick a sync source and press play.

Use the per-output test sliders to check the toy responds before you commit
to anything. **Esc** or the red button stops everything, instantly, from any
state.

## Sync sources

| Source | When to use it | How it maps |
| --- | --- | --- |
| **Funscript** | You have a `.funscript` for the video (EroScripts and similar communities script a lot of PMVs). | *Stroke speed* mode (default): faster scripted strokes = stronger vibration. *Position* modes: vibration tracks the stroke position directly, which pulses with every stroke. |
| **Audio reactive** | No script. PMVs are cut to the beat, so bass energy is a good proxy. | Bass band RMS, auto-normalised to the track's loudness, with an onset detector for *Beat pulse* mode. Band and sensitivity are adjustable. |
| **BPM pattern** | Nothing else fits, or you just want a steady rhythm. | Sine, pulse, ramp or an 8-bar build, locked to the video clock. Tap tempo included. |

## Shaping

- **Master** and **Floor** scale the signal and set a minimum level.
- **Lead time** sends commands early to cancel Bluetooth latency. Start at
  +120 ms and adjust with `[` / `]` while watching a scripted section.
- **Smoothing** rounds off sharp changes.
- **Suction / air pump**: the MarsLion's pump is a slow mechanical thing, so by
  default it follows a 1.2 s envelope of the signal and changes at most about
  twice a second. It can also follow script position, sit at a fixed level,
  or stay off. The cap slider limits how hard it ever pulls.

## How it talks to the toy

The page speaks the Buttplug message protocol (spec v3) directly over the
websocket: `RequestServerInfo`, `StartScanning`, `DeviceAdded`, `ScalarCmd`,
`StopAllDevices`. No SDK, no bundler, no dependencies. Commands are sent at
5–20 Hz, only when a quantised level actually changes, with a 1.5 s keepalive
resend. Pause, seek, end-of-video, disconnect and page close all send zero or
`StopAllDevices`.

## Notes

- Chrome and Edge are the tested browsers. They treat `ws://127.0.0.1` as a
  secure origin, so the page works whether opened from disk or from an
  `https://` host.
- Audio-reactive mode needs the browser to be able to decode the file's audio
  track. Local files are fine.
- Settings persist in `localStorage`.
