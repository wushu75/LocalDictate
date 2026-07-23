# Testing LocalDictate

Two halves: what a machine can check, and what only you and a microphone can.

---

## 1. Automated checks

```bash
npm install
npm run build      # vendor transformers.js + ORT into lib/
npm run check      # validate + unit tests
```

`npm run validate` is a pre-flight over the package itself — every manifest path,
every relative import, every HTML asset reference, the CSP, the command count, the
vendored runtime. It catches the class of bug that produces a silently blank
extension. 54 checks, and it runs automatically before `npm run package`.

`npm test` covers the two pieces of pure logic: post-processing and voice activity
detection. 21 tests, no browser needed. If you change a regex in
`src/common/postprocess.js` or a constant in `src/common/vad.js`, these are what tell
you whether you broke dictation for everyone.

Neither of these can test audio capture, model loading, WebGPU or text insertion.
That is the rest of this document.

---

## 2. Loading it

```bash
npm run build
```

1. `chrome://extensions` → **Developer mode** on → **Load unpacked** → select the repo folder.
2. A welcome tab opens. Grant the microphone.
3. Open the popup, choose **Tiny** for your first run — it downloads in under a minute.
4. Wait for the progress bar to finish and the chip to read `WebGPU` or `WASM`.

**Reloading after a change:** hit the reload arrow on the extension card. If you changed
the content script, also reload the page you are testing on — the old copy is still
injected there. If you changed the offscreen document or the worker, stop and restart
dictation, since both are recreated on start.

---

## 3. The five-minute smoke test

Do these in order. If any fails, stop and fix it before going further.

| # | Action | Expected |
|---|---|---|
| 1 | Open the popup | Lamp is grey, status "Idle" or "Ready", device chip populated |
| 2 | Click into this README's GitHub comment box, press `Ctrl+Shift+D` | HUD appears bottom-centre, lamp amber, status "Listening" |
| 3 | Say nothing for 5 seconds | Meter stays dark, nothing is typed, no hallucinated text |
| 4 | Say "hello world" | Meter lights while you speak, text appears in the box after you stop |
| 5 | Press `Ctrl+Shift+D` again | HUD fades, badge clears |
| 6 | Reopen the popup | Your sentence is in the transcript panel, Copy/Export enabled |

Step 3 is the one people skip and the one that matters most. Whisper will confabulate
entire sentences out of silence — "Thanks for watching!" is the classic — and the VAD gate
plus `isNoiseOnly()` are what stop that reaching your document.

---

## 4. Where the consoles are

Four contexts, four separate DevTools. Most "nothing happened" reports are an error
sitting in a console the reporter never opened.

| Layer | How to open it |
|---|---|
| Service worker | `chrome://extensions` → LocalDictate → **service worker** |
| Offscreen document | Same card → **offscreen.html** (only listed while dictation is active) |
| Inference worker | Offscreen DevTools → Sources → Threads → `whisper-worker.js` |
| Popup | Right-click inside the popup → Inspect |
| Options | Normal page DevTools |
| Content script | The DevTools of the page you are dictating into |

Useful one-liner, pasted into the **service worker** console:

```js
chrome.runtime.sendMessage({ to: 'background', type: 'get-state' }).then(console.log)
```

---

## 5. Insertion matrix

Text insertion is where this extension will actually break, because every editor is
different. Work down this list; the first four are the ones that must never regress.

| Target | Type | Expected |
|---|---|---|
| A plain `<textarea>` | native | Types at the caret, undo works |
| `<input type="text">`, caret mid-string | native | Inserts at the caret, does not append |
| GitHub comment box | contenteditable | Types at the caret, markdown preview updates |
| Gmail compose | contenteditable | Types at the caret, draft autosaves |
| Slack message box | contenteditable (Quill) | Types at the caret |
| Notion page | contenteditable | Types at the caret |
| Google Docs | canvas | **Clipboard fallback** + toast telling you to paste |
| A React-controlled input | native + framework | Text persists — does not vanish on the next keystroke |
| An iframe-embedded editor | either | Only the focused frame inserts, exactly once |
| A page with two iframes | either | Text lands in one frame only |

The React case is the subtle one. If a framework-controlled input reverts the moment you
type, the native prototype setter in `insertIntoField()` has been broken — assigning
`el.value` directly does not notify React.

Test the caret explicitly: type `Hello world`, click between the two words, then dictate.
The text must land in the middle, and `autoSpace` must not add a space before a comma.

---

## 6. Forcing the failure modes

Every one of these is a real user's Tuesday. Each should degrade gracefully with a
message that says what to do next.

**WASM fallback** — Settings → Engine → Backend → *WebAssembly (CPU) only*, then reload the
model. The chip should read `WASM`, transcription should still work, and it should be
visibly slower. Also test *WebGPU only* on a machine without it: expect a clear notice,
not a crash.

**Microphone denied** — click the padlock in the address bar on the extension's own page
and block the mic, then try to dictate. Expect the "microphone is blocked" message and a
button that opens the permission page. Then re-grant and confirm recovery without a
browser restart.

**Fully offline** — load a model, then DevTools → Network → *Offline*, or turn off Wi-Fi.
Dictation must work exactly as before. This is the core promise; test it properly.

**Cold cache** — Settings → Storage & privacy → **Delete downloaded models**, then dictate.
Expect a download with a progress bar, not a silent hang.

**Interrupted download** — start Small, kill the network mid-download, restore it, retry.
Expect a usable error and a working retry.

**Cancel mid-sentence** — start a long sentence and press `Alt+Shift+X` while the model is
decoding. Nothing should be inserted, and the next utterance should work normally.

**Service worker death** — MV3 kills it after ~30 s idle. Wait a minute after loading the
extension, then press the hotkey. It must wake and work. This is the single most common
source of "it stopped working after a while" reports.

**Restricted pages** — press the hotkey on `chrome://extensions` or the Web Store. Expect a
graceful no-op; the text should land in history rather than disappearing.

**Two tabs** — start dictating in tab A, switch to tab B, keep talking. Text goes to tab A,
where you started. That is deliberate.

---

## 7. Accessibility and appearance

- Tab through the popup and options: every control reachable, focus ring always visible.
- Toggle Settings → Theme through system / dark / light; check both popup and options.
- Turn on **Reduce motion** in your OS. The lamp should stop pulsing and the HUD should
  stop sliding. The level meter still updates — it is information, not decoration.
- Zoom the page to 200%. The HUD must not cover the text field you are dictating into.
- Check the HUD against a light page and a dark page.

---

## 8. Measuring performance

In the **offscreen** console:

```js
// Latency for the last utterance is reported on every final result.
chrome.runtime.onMessage.addListener(m => {
  if (m.type === 'final') console.log(`${m.seconds}s audio → ${m.ms}ms decode on ${m.device}`);
});
```

A useful sanity ratio is decode time ÷ audio duration. Under 0.3 feels instant; over 1.0
means you are transcribing slower than you can talk, and you should drop a model size or
check that WebGPU is actually engaged.

Ignore the first utterance after every load — that one is paying for shader compilation.

---

## 9. Before you tag a release

- [ ] `npm run check` passes
- [ ] Smoke test passes on a clean profile with no cached model
- [ ] The first four rows of the insertion matrix pass
- [ ] Offline dictation works after a model download
- [ ] Both themes and reduced-motion look right
- [ ] `npm run package` produces a zip that loads unpacked from a fresh unzip
- [ ] Version bumped in `manifest.json` **and** `package.json`, and `CHANGELOG.md` updated

---

## 10. What is deliberately not tested

- **Transcription accuracy.** That is Whisper's, not ours. If quality is poor, the
  variables are model size, microphone and background noise — not this codebase.
- **Every language.** The pipeline is language-agnostic; spot-check the ones you use.
- **End-to-end automation.** Puppeteer can load an MV3 extension but cannot grant a real
  microphone or exercise WebGPU meaningfully. A fake audio device
  (`--use-fake-device-for-media-stream --use-file-for-fake-audio-capture=speech.wav`)
  gets you partway, and would be a genuinely valuable contribution — but it tests the
  plumbing, not the experience.
