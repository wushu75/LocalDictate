# Architecture

LocalDictate is four contexts passing messages. Nothing else.

## The contexts

### 1. Content script — `src/content/content.js`

Runs on every page and in every frame. Three responsibilities:

- **Focus tracking.** Listens to `focusin` and `pointerdown` and remembers the last
  editable element, traversing shadow roots. This must happen *before* dictation starts,
  because opening the popup moves focus off the page.
- **The HUD.** Only in the top frame. A pill with a lamp, a level meter and the current
  partial transcript.
- **Insertion.** On `insert`, the frame checks whether it still owns the caret. Exactly
  one frame does, so there is no cross-frame coordination.

Insertion strategy, in order:

1. `<input>` / `<textarea>` → the native `value` setter from the prototype, then a
   synthetic `InputEvent`. Using the prototype setter is what makes React and Vue notice
   the change; assigning `el.value` directly does not.
2. `contenteditable` → `document.execCommand('insertText')`, which preserves the undo
   stack and fires the events rich editors listen for. Falls back to a `Range` splice.
3. Anything else, or a thrown error → clipboard, with a toast telling the user to paste.

### 2. Service worker — `src/background/service-worker.js`

The router. Owns the hotkeys, creates and tears down the offscreen document, tracks which
tab dictation started in, applies post-processing to finished text, and writes history.
It never sees audio.

MV3 service workers are killed aggressively. Nothing important is kept in module scope
that cannot be reconstructed — `session` is a cache, not a source of truth, and the
offscreen document survives independently.

### 3. Offscreen document — `src/offscreen/offscreen.js`

Exists because a service worker cannot hold a `MediaStream` or an `AudioContext`.

```
getUserMedia (16 kHz mono)
  → MediaStreamSource
  → AudioWorkletNode (pcm-processor)   1024-sample frames, 64 ms
  → VAD                                RMS vs adaptive noise floor
  → utterance buffer                   grows while speech is open
  → Worker.postMessage(Float32Array)   transferred, not copied
```

Segmentation rules:

| Event | Condition | Result |
|---|---|---|
| Utterance opens | ≥160 ms above the gate | State becomes `speaking` |
| Partial decode | every `partialIntervalMs` while open, buffer > 900 ms | non-blocking preview |
| Utterance closes | `silenceMs` below the gate | final decode, then insertion |
| Forced close | buffer reaches `maxUtteranceMs` | final decode, buffer resets |
| Discarded | buffer < 350 ms | ignored — a door, not a sentence |

Only one decode runs at a time. Partials that arrive while the worker is busy are dropped
(they are disposable). A final that arrives while the worker is busy is queued in a
single slot, so the newest utterance always wins and the queue cannot grow.

### 4. Inference worker — `src/worker/whisper-worker.js`

Holds the Transformers.js pipeline. Receives `Float32Array` at 16 kHz, returns text.

- Device selection: probe `navigator.gpu.requestAdapter()`, build on WebGPU, and rebuild
  on WASM if construction throws. The user is told which one won.
- dtype: `fp32` encoder on WebGPU (`q4` decoder for Small), `q8` everywhere on WASM.
- Cancellation: there is no reliable abort in the middle of a generation, so each request
  carries a generation number and stale results are dropped instead of interrupted.
- Warm-up: one second of silence is decoded after load so the first real utterance is not
  paying for shader compilation.

## Model caching

`src/common/idb-cache.js` implements the two methods Transformers.js needs from a cache —
`match(request)` and `put(request, response)` — over IndexedDB, and is installed via
`env.useCustomCache`. IndexedDB rather than Cache Storage so that:

- clearing "cached images and files" in Chrome does not silently delete 500 MB the user
  will have to re-download;
- the extension can report its own footprint precisely in Settings;
- deleting models is one `clear()`, not a guess at cache key names.

## Message protocol

Every message carries `to` (`background` | `offscreen` | `popup` | `content`) and `type`
(from `MSG` in `src/common/constants.js`). Handlers return early when `to` does not match
them, which is what keeps four `onMessage` listeners in one runtime from stepping on each
other.

Downstream flow for one spoken sentence:

```
offscreen  FINAL  →  background
background         postProcess() + history
background INSERT  →  content (target tab)
background FINAL   →  popup (display)
```

## Deliberate limitations

- **No cross-frame arbitration.** If two frames somehow both believe they hold the caret,
  both insert. In practice `document.activeElement` makes this impossible.
- **No speaker diarisation, no timestamps.** Whisper can do both; dictation does not need
  either, and both cost latency.
- **No streaming *during* speech.** Partials are re-decodes of the open buffer, not an
  incremental decoder. A true streaming decoder is the most valuable open contribution.
