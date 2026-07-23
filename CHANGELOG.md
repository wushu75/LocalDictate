# Changelog

All notable changes to LocalDictate are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project uses
[semantic versioning](https://semver.org/).

## [1.0.0] — 2026-07-23

First public release.

### Added
- On-device speech recognition with Whisper Tiny, Base and Small.
- WebGPU inference with an automatic WebAssembly fallback.
- Model download with per-file progress and IndexedDB caching.
- Toggle hotkey (`Ctrl+Shift+D`) and hold-to-talk on a configurable key.
- Automatic insertion into inputs, textareas and contenteditable regions,
  with a clipboard fallback for canvas editors such as Google Docs.
- Energy-based voice activity detection with an adaptive noise floor.
- Post-processing: filler removal, sentence capitalisation, punctuation
  spacing, spoken punctuation commands and user-defined replacements.
- In-page listening HUD with a live level meter and partial transcript.
- Local transcript history with export, plus dark and light themes.
