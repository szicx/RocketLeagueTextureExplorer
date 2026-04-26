<div align="center">

<img src="https://img.shields.io/badge/Rocket%20League-Textures%20Explorer-003EFF?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0xMiAyQzYuNDggMiAyIDYuNDggMiAxMnM0LjQ4IDEwIDEwIDEwIDEwLTQuNDggMTAtMTBTMTcuNTIgMiAxMiAyem0tMSAxNy45M1Y0LjA3YzMuOTQuNDkgNyAzLjg1IDcgNy45M3MtMy4wNiA3LjQ0LTcgNy45M3oiLz48L3N2Zz4="/>

# RocketLeague Textures Explorer

**A real-time DirectX 11 texture inspector and exporter for Rocket League.**

Browse, preview, and export every texture loaded by the game.

<img src="https://raw.githubusercontent.com/szicx/RocketLeagueTextureExplorer/main/preview.png" alt="Preview" width="100%"/> </div>

***

## Overview

RocketLeague Textures Explorer injects into the game process and hooks directly into the **DirectX 11 rendering pipeline**.

It intercepts every texture in real time, and exposes them through a ImGui overlay, without touching any UE3 assets or game files.

***

## How It Works

The mod hooks two functions in the D3D11 pipeline via MinHook:

### `IDXGISwapChain::Present`
Called every frame when the game submits a rendered frame to the display.

Used to initialize the ImGui overlay on the game's own D3D11 device and render the UI without any separate window.

### `ID3D11DeviceContext::PSSetShaderResources`
Called every time the engine binds a texture to the pixel shader stage. For each new shader resource view (SRV) intercepted, the mod:

1. Retrieves the underlying `ID3D11Texture2D` from the SRV
2. Creates a CPU-readable staging texture and performs a **GPU → CPU readback** of mip level 0
3. Converts the raw data to **RGBA8**, handling all common DXGI formats (BC1–BC7, UNORM, SRGB, etc.)
4. Computes a **SHA-256 hash** of the pixel data as a stable, content-based unique identifier
5. Stores everything in an in-memory hash map — exact duplicates are silently dropped

Captures are capped per frame to avoid hitches. A pointer-based pre-check short-circuits repeat visits to already-seen GPU resources.

***

## Features

| Feature | Description |
|---|---|
| 🔍 **Real-time detection** | Textures appear as the engine renders them — no need to restart |
| 🖼️ **Texture preview** | View any texture data with a transparency checkerboard |
| 🔎 **Search by hash** | Filter the list instantly by typing a partial or full SHA-256 hash |
| 📋 **Copy Hash** | One-click copy of the full 64-character SHA-256 hash |
| 💾 **Export PNG** | Save as RGBA PNG via a native file dialog |
| 💾 **Export DDS** | Save as uncompressed RGBA8 DDS, compatible with most texture tools |
| 🗑️ **Clear** | Reset the captured texture list without restarting |
| ⌨️ **F1** | Toggle the overlay on/off |
| ⌨️ **F2** | Safely unload the DLL and restore the game state |

***

## Usage

The mod is distributed as a `.dll` that must be injected into the running Rocket League process.

### Recommended Injector

**[Extreme Injector](https://github.com/master131/extremeinjector)** — simple, free, and reliable.

### Steps

1. Download **`RLTexturesExplorer.dll`** from the [latest release](https://github.com/szicx/RocketLeagueTextureExplorer/releases/latest)
2. Launch **Rocket League** and wait until you reach the main menu
3. Open **Extreme Injector** as **Administrator**
4. Click **Select Process** → choose `RocketLeague.exe`
5. Click **Add DLL** → select `RLTexturesExplorer.dll`
6. Click **Inject**
7. The overlay will appear in-game

> **To unload:** press **F2** inside the overlay, or simply close the game. All hooks and GPU resources are released cleanly.

### Populating the list

Detection is **passive** — a texture only appears once it has been drawn at least once in the current session. To capture more textures:

- Navigate through the main menu, garage, game maps ect...

***

## Notes

- All data is **in-memory only** — nothing is written to disk unless you explicitly export a texture

***

## Disclaimer

This tool is intended for **modding, research, and educational purposes only**.  
Use it responsibly and in accordance with [Psyonix's terms of service](https://www.psyonix.com/eula/).  
The author is not responsible for any misuse.

***
