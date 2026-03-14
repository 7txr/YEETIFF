<div align="center">

# YEETIFF

**Yet Even Extremely Expressier Transcoded Image File Format**

[![Version](https://img.shields.io/badge/version-2.0-blue?style=flat-square)](https://github.com/7txr/YEETIFF/releases)
[![Rust](https://img.shields.io/badge/Rust-1.70%2B-orange?style=flat-square&logo=rust&logoColor=white)](https://www.rust-lang.org/)
[![Python](https://img.shields.io/badge/Python-3.x-3670A0?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)
[![Build](https://img.shields.io/badge/build-passing-brightgreen?style=flat-square)](https://github.com/7txr/YEETIFF)

An open, well-documented image format built from scratch to explore binary encoding, compression algorithms, and file format design at the byte level.

[Specification](#format-specification) · [Quick Start](#quick-start) · [Docs](docs/) · [Releases](https://github.com/7txr/YEETIFF/releases)

</div>

---

## Overview

YEETIFF is a custom image container format and toolchain written in **Rust**, with a **Python**-based Windows installer. It was built as a deep-dive into how image formats actually work — headers, compression pipelines, metadata schemas, and binary encoding — rather than relying on existing format internals as a black box.

The project ships three format generations in parallel: a stable v2 implementation, a legacy v1 compatibility layer, and an experimental v3 branch exploring ICC color profiles and multi-frame animation.

**Capabilities (v2 stable):**

- Full **RGBA** transparency — true alpha channel, not bitmask
- **zlib compression** — 40–80% size reduction depending on image content
- Dual encoding modes — human-readable hex or compact binary
- **JSON metadata** block — author, timestamp, and arbitrary key-value pairs
- Batch conversion of entire directories in a single command
- **OpenGL-accelerated GUI viewer** built on `egui` / `eframe`
- Windows installer with file association registration via `pywin32`
- Backward-compatible v1 reader

---

## Project Structure

```
YEETIFF/
├── yeet-core/          # Stable v2 implementation (production) · Rust
│   └── src/main.rs     # 570+ lines — encoder, decoder, GUI viewer, CLI
│
├── yeet-v3/            # Experimental v3 (alpha) · Rust
│   └── src/main.rs     # ICC profiles, animation, Brotli/Zstd — in progress
│
├── yeet-legacy/        # v1 backward compatibility layer · Rust
│
├── yeet-installer/     # Windows installer · Python
│   ├── installer_gui.py     # Tkinter installation wizard
│   └── build_installer.py   # PyInstaller packaging script
│
├── docs/
│   ├── SPEC_v2.md      # Complete v2 byte-level format specification (650+ lines)
│   ├── SPEC_v3.md      # v3 feature roadmap and specification (200+ lines)
│   └── ARCHITECTURE.md # Code organisation and data flow (500+ lines)
│
├── examples/           # Sample .yeet files
├── Cargo.toml          # Workspace manifest
└── BUILD.md            # Build instructions
```

---

## Format Specification

### v2 File Structure

Every `.yeet` file begins with a fixed header, followed by a variable-length metadata block and the pixel data payload:

```
Offset   Size     Field              Description
──────   ──────   ─────              ───────────────────────────────────────────
0x00     4 B      Magic              ASCII "YEET" — format identifier
0x04     1 B      Version            0x02 for v2
0x05     1 B      Flags              Bitfield: compression, alpha, binary mode
0x06     4 B      Width              Image width in pixels (u32, little-endian)
0x0A     4 B      Height             Image height in pixels (u32, little-endian)
0x0E     2 B      Metadata Length    Byte length of JSON block (u16, little-endian)
0x10     var      Metadata           UTF-8 JSON string (author, timestamp, etc.)
var      4 B      Data Length        Byte length of pixel data (u32, little-endian)
var      var      Pixel Data         Raw RGBA bytes, optionally zlib-compressed
```

**Flags byte breakdown:**

| Bit | Meaning |
|-----|---------|
| `0` | Compression enabled (`flate2` / zlib deflate) |
| `1` | Alpha channel present (RGBA vs RGB) |
| `2` | Binary mode (raw bytes) vs hex-text mode |
| `3–7` | Reserved |

### Compression Performance

Real-world measurements on 1920×1080 images:

| Image type | Uncompressed | Compressed | Reduction |
|------------|-------------|------------|-----------|
| Photos | 6.2 MB | ~2.5 MB | ~60% |
| Vector / flat graphics | 6.2 MB | ~1.9 MB | ~70% |
| Text / UI screenshots | 6.2 MB | ~1.2 MB | ~80% |

### Format Evolution

```
v1 (legacy)           v2 (stable)              v3 (experimental)
──────────────        ────────────────          ─────────────────────────
RGB only              RGBA transparency         RGBA + HDR (16-bit)
No compression        zlib (flate2)             zlib · Brotli · Zstd
Text / hex only       Text + binary modes       Extended encoding
8-byte header         20+ byte header           Extended header + EXIF
No metadata           JSON metadata block       ICC color profiles
                                                Multi-frame animation
```

Full byte-level documentation: [`docs/SPEC_v2.md`](docs/SPEC_v2.md)

---

## Quick Start

### Install (Windows)

Download `YeetInstaller.exe` from [Releases](https://github.com/7txr/YEETIFF/releases). The installer registers `.yeet` file associations automatically.

### Build from Source

```bash
git clone https://github.com/7txr/YEETIFF.git
cd YEETIFF

# Build stable v2
cargo build --release -p yeet-core

# Run
./target/release/yeet --help
```

Requirements: Rust 1.70+, Cargo.

### Basic Usage

```bash
# Open a .yeet file in the GUI viewer
yeet image.yeet

# Convert PNG → YEET (compressed binary, recommended)
yeet compile photo.png --compress --binary

# Batch convert a directory
yeet batch ./photos --compress --binary

# Convert without compression (human-readable hex, useful for debugging)
yeet compile photo.png
```

---

## Dependencies

**Rust (yeet-core):**

| Crate | Purpose |
|-------|---------|
| `image` | PNG / JPEG I/O for conversion |
| `eframe` + `egui` | Immediate-mode OpenGL GUI |
| `egui_extras` | Image display widgets |
| `flate2` | zlib deflate/inflate (compression) |

**Python (yeet-installer):**

| Package | Purpose |
|---------|---------|
| `tkinter` | Installer GUI (stdlib) |
| `Pillow` | Image preview in wizard |
| `pywin32` | Windows registry — file associations |
| `PyInstaller` | Bundle to standalone `.exe` |

---

## Roadmap

**v2.1 (in progress)**
- Unit test coverage across encoder/decoder
- Cross-platform CI via GitHub Actions
- Performance benchmarks against PNG / WebP

**v3.0 (planned)**
- ICC color profile embedding (accurate color reproduction)
- Multi-frame animation (APNG-style sequences)
- Brotli and Zstd compression backends
- 16-bit per channel HDR support
- Extended EXIF metadata

**Future**
- `yeet-lib` — library API for embedding in other projects
- WASM viewer for browser-based rendering
- Package manager distribution (`cargo install yeet`)

---

## Contributing

Contributions are welcome. YEETIFF is intentionally kept simple to be a useful learning resource — keep that in mind when proposing changes.

```bash
# Fork, clone, then:
cargo build --workspace
cargo test --workspace
cargo fmt --all
cargo clippy --workspace
```

See [`docs/CONTRIBUTING.md`](docs/CONTRIBUTING.md) for guidelines on pull requests, code style, and testing requirements.

---

## License

MIT — see [LICENSE](LICENSE) for details.

---

<div align="center">
  <sub>Built by <a href="https://github.com/7txr">7txr</a> · Part of the <a href="https://github.com/ordnary-com">Ordnary</a> ecosystem</sub>
</div>
