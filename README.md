# Kiwi Sorter AR — XREAL One Client

The wearable component of the [Kiwi Sorter system](https://github.com/baboo-balan-1173535/ai-fruit-quality-inspection-ar):
an optical see-through AR overlay that locks bounding boxes and a quality card
onto **real fruit** seen through XREAL One glasses.

> **Requires the detection server** from
> [ai-fruit-quality-inspection-ar](https://github.com/baboo-balan-1173535/ai-fruit-quality-inspection-ar)
> running on the same network (`/detect`, `/ar-analyze`, UDP discovery beacon).

Unity 2022.3 LTS · XREAL SDK 3.0 · AR Foundation 5.1 · C# · package `com.kiwi.sorter.ar`

## What it does

Wearing the glasses, you look at fruit and see — overlaid on the real world — a
box on the fruit with its label, confidence, size and an AI quality panel
(grade, decay stage, days remaining, defects). No phone-screen mirroring, no
video passthrough; the overlay sits on the optical see-through display.

## How it works — world-anchored projection

A flat screen-space canvas can't map to the landscape optical FOV (the canvas is
portrait). Instead:

1. Each detection's image position + size is converted into a **real-world 3D
   point** using the current head pose and the Eye-camera FOV.
2. Every frame, Unity's XR camera **projects that world point back to the
   screen**, so the box stays on the fruit as the head turns (3DoF — rotation).
3. Box motion is smoothed with an exponential filter + catch-up boost.

The Eye camera is bolted to the head, so 3DoF (rotation only) is sufficient — no
positional tracking needed.

## Hardware chain

```
XREAL Eye (UVC camera) ─USB─▶ Samsung S24 ─5GHz WiFi─▶ Laptop (Flask :5000)
                              (Unity AR app)            /detect · /ar-analyze
XREAL One glasses ◀── optical see-through (3DoF)
```

The app **auto-discovers** the server over UDP (port 5006) on any WiFi — no
hardcoded IP, no rebuild on a network change.

## On-glass controls

| Button | Action |
|--------|--------|
| **REC** | Server assembles an annotated MP4 of the session |
| **SNAP** | Saves an annotated "what I see" image to the laptop |
| **AI** | Toggles the Claude quality pipeline on/off |
| **CAL** | Live calibration (FOV, box size, motion smoothing) |
| Home button | One-press quit |

## Key files (`Assets/`)

- `Scripts/FruitDetectionManager.cs` — capture, the two pipelines, world-anchored
  tracking, smoothing, discovery, snapshot/record, the calibration panel.
- `Scripts/FruitPanel.cs` — the per-fruit box + label card (placement only).
- `Editor/KiwiSceneBuilder.cs` — builds the scene and the FruitPanel prefab.
- `XR/Settings/XREALSettings.asset` — tracking mode 3DoF (required).

## Prerequisites — XREAL SDK (not bundled)

The XREAL Unity SDK 3.0 cannot be redistributed here. Download it from the
[XREAL developer site](https://developer.xreal.com), then edit
`Packages/manifest.json` so the `com.xreal.xr` entry points at your local copy:

```json
"com.xreal.xr": "file:<path-to-your-downloaded-sdk>/package",
```

Unity 2022.3 LTS is required (the project was built on 2022.3.62f3).

## Build & deploy

1. (If prefab visuals changed) `KiwiSorter ▸ Rebuild FruitPanel Prefab`, save.
2. Confirm Inspector calibration values on `FruitDetectionManager`.
3. `File ▸ Build Settings ▸ Android ▸ Build`.
4. Uninstall the old app, install the new APK, launch via ControlGlasses.

Source changes do not reach the glasses until rebuilt — "nothing changed" almost
always means "not rebuilt".

## Status

v1.0 verified on-glass (world-anchored tracking, smoothing, quality panel,
snapshot/record). Future: hand-tracking controls, on-device inference, and a
custom segmentation model for clustered fruit + kiwi.

## License

**All rights reserved.** Personal portfolio project published for review and
demonstration only — no permission is granted to reuse, copy, modify, or
redistribute any part of the code.
