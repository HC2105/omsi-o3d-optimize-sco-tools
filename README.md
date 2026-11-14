# OMSI O3D Optimize & SCO Tools

A single-file browser tool for **optimizing `.o3d` material slots**, **scanning texture alpha channels**, and **generating `.sco` `[matl_alpha]` blocks** for the OMSI bus simulator.
Load an individual `.o3d`, a full Sceneryobject folder, or a `model/` batch folder, inspect duplicate materials and transparency percentages, merge redundant slots, and export optimized `.o3d` files or ready-to-paste `.sco` blocks.

**→ [https://hc2105.github.io/omsi-o3d-optimize-sco-tools/](https://hc2105.github.io/omsi-o3d-optimize-sco-tools/)**

Hosted via GitHub Pages. First visit loads two small CDN libraries (Tailwind CSS, Lucide icons) and then the app runs entirely in your browser — no account, no login, no server upload, no telemetry. All `.o3d` binary parsing, `.tga` / `.dds` decoding, and `.zip` packaging happen locally in memory.

## What it does

Three import workflows, walked through in-app:

| Mode | What you load | Output | 
| ----- | ----- | ----- | 
| **1 — Individual `.o3d`** | Single `.o3d` + optional texture folder | Optimized `.o3d` + `.sco` `[matl_alpha]` snippet | 
| **2 — Sceneryobject Folder (By `.sco`)** | Entire Sceneryobject folder (`.sco`, `model/`, `texture/`) | Per-`.sco` `[matl_alpha]` blocks + individual or batch `.zip` `.o3d` export | 
| **3 — Model Folder Batch (By `.o3d`)** | `model/` folder of `.o3d` files + optional texture folder | Step-through `.o3d` inspection + batch-fixed `.zip` preserving folder paths | 

Extras shipped with the pipeline:

* **Duplicate material detection & merging** — flags material slots sharing the same texture (such as `XYZ.png.001` / `002` carried over from Blender) and combines them into a single slot, remapping triangle indices to cut file size and draw calls.

* **Export lock safety** — keeps individual `.o3d` export locked until duplicate slots are actually merged so you never accidentally re-download an identical unfixed file.

* **One-click Batch Fix & ZIP export** — in Modes 2 and 3, merges all duplicate slots across the folder and downloads a `.zip` archive preserving the original directory structure.

* **Built-in `.tga` & `.dds` decoders** — parses uncompressed/RLE `.tga` and DXT1/DXT3/DXT5 `.dds` textures directly in JavaScript to measure exact cutout (`A = 0`) vs. half-transparent (`0 < A < 255`) pixel ratios, with click-to-enlarge alpha checkerboard previews.

* **Search, sort & command logfile** — filter by category (`Duplicates`, `Alpha 1 & 2`, `No Alpha`, `Missing`), search texture names, sort by slot index or alphabetically, and track every action in the live logfile.

> **Note:** Model Folder Batch Scan (Mode 3) is intended for scenery objects, not vehicle `.o3d` folders (`Vehicles\...\model`), which rely on intentional multi-slot materials for rain glass, lightmaps, `[matl_envmap]`, and `[matl_change]` scripts.

## Why combine duplicate material slots?

When building or joining meshes in Blender, it is common to end up with duplicate materials (`Material.001`, `Material.002`, etc.) that all point to the exact same texture file. When exported to `.o3d`, each duplicate material becomes its own separate `0x26` material slot.

In both **legacy OMSI 2** and **modernized engines like openOMSI**, **every material slot still incurs separate rendering work**:

- **In OMSI 2 (DirectX 9)** — OMSI 2 is heavily single-thread CPU-bound, and every material slot triggers its own draw call and render-state switch. Merging 5 duplicate slots on a building into 1 cuts those draw calls by 80%, which directly improves FPS and reduces stutter when the object is placed dozens of times across a map.
- **In modernized engines (e.g. openOMSI)** — Even with multi-threaded command recording and modern graphics APIs, unmerged material slots still fragment a mesh into extra sub-meshes, descriptor/bind-group lookups, and GPU draw submissions. They also hurt **GPU instancing and batching** efficiency across repeated scenery objects and add unnecessary `.o3d` parsing and GPU buffer-upload overhead during tile streaming.
- **Smaller `.o3d` file size & cleaner `.sco` blocks** — removing redundant `0x26` material headers trims wasted bytes from every model without touching a single vertex or UV coordinate, and avoids repeating `[matl]` / `[matl_alpha]` entries for the same texture.

## Supported inputs

| Format | OMSI | Browser (native) | Supported | 
| ----- | ----- | ----- | ----- | 
| `.o3d` (v1–v7, encrypted or plain) | yes | no (custom parser) | **yes** | 
| `.sco` (UTF-8 / UTF-16 LE / CP1252) | yes | yes | **yes** | 
| PNG / JPG / JPEG / BMP | yes | yes | **yes** | 
| TGA (uncompressed & RLE) | yes | no (custom decoder) | **yes** | 
| DDS (DXT1 / DXT3 / DXT5 / RGBA) | yes | no (custom decoder) | **yes** | 
| WebP | no | yes | **yes** | 

## Running it

```
# Just open the file
index.html

```

That's it. Chrome / Edge / Firefox / Safari all work. If you prefer serving over `http` (for example, to avoid `file://` quirks in stricter browser setups):

```
# Optional, if you want a tiny static server
python -m http.server 8000
# then open http://localhost:8000/index.html

```

## File layout

This repo intentionally ships one self-contained app file. All state, all UI, all binary parsers, and the ZIP packager live in `index.html`. Open it, use it, archive it.

## Credits

OMSI O3D Optimize & SCO Tools V1.01 — Released by HC2105 @ [TJ x HC Studio](https://www.facebook.com/TJ9385omsistudio), 2026. Coded with Gemini.
