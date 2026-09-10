# sendtra-store

The official widget/app registry for the **Sendtra Select** handheld. This repo *is* the store — the device reads straight from it over WiFi, and **Widget Studio** (the local design tool on your dev machine) is the only thing that should ever write to it.

## How it works

1. You design a widget in **Widget Studio** (runs locally at `localhost:8787`).
2. Hitting **Publish** pushes `draw.lua` + `icon.bin` into `widgets/<id>/` here, then updates [`widgets.json`](./widgets.json) with that widget's entry — including a SHA-256 checksum of each file.
3. The device polls `widgets.json` (raw, straight off `main`) when you open the Widgets tab. For every entry it doesn't already have, it downloads the script + icon, **verifies the checksum before writing anything to flash**, and only then adds it to your device's widget list.

```
sendtra-store/
├── widgets.json              <- the registry itself; the device fetches THIS file
└── widgets/
    └── <widget-id>/
        ├── draw.lua           <- the widget's Lua script (function draw(ox, oy, w, h))
        └── icon.bin           <- 196x52 RGB565, big-endian, no header -- cached thumbnail
```

## `widgets.json` schema

```json
{
  "widgets": [
    {
      "id": "be846147f797",
      "name": "333",
      "version": 2,
      "icon_url": "https://raw.githubusercontent.com/sendtra/sendtra-store/main/widgets/be846147f797/icon.bin",
      "script_url": "https://raw.githubusercontent.com/sendtra/sendtra-store/main/widgets/be846147f797/draw.lua",
      "icon_sha256": "...",
      "script_sha256": "..."
    }
  ]
}
```

- `icon_sha256` / `script_sha256` are optional but always set by Widget Studio on publish. The device treats a missing checksum as "not offered, skip the check" (so older entries don't break) but **fails closed** on a malformed one.
- `version` is a plain incrementing integer, bumped by Widget Studio on every republish.

## Don't hand-edit this repo

`widgets.json` and everything under `widgets/` is generated and owned by Widget Studio. Editing it by hand risks a checksum mismatch (the device will just silently refuse the file) or an entry that doesn't match what's actually in `widgets/<id>/`. Manage widgets from the Studio's **Manage Published** tab instead — edit, republish, or delete from there and it keeps this repo in sync for you.

## Visibility

This repo is **public** (so any device can pull `widgets.json` over plain HTTPS with no auth) but only accepts writes from Widget Studio's own saved GitHub token. Nobody else can publish here unless they have write access to the repo itself.
