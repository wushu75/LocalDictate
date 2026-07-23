# Chrome Web Store listing — LocalDictate

Everything below is ready to paste into the Developer Dashboard, field by field.

---

## Item name (45 characters max)

```
LocalDictate — Private Offline Voice Typing
```
*43 characters.*

---

## Short description (132 characters max)

```
Voice typing that never leaves your device. Whisper runs in your browser — press a hotkey, speak, and text lands in any field.
```
*125 characters.*

---

## Category

Productivity → Workflow & Planning

## Language

English (United States)

---

## Detailed description (16,000 characters max)

```
Speak freely. Type instantly. 100% private and offline.

LocalDictate turns your voice into text in any text field on any website — Gmail, Notion, Slack, GitHub, Jira, a search box, an old web form. Put your caret where you want the words, press Ctrl+Shift+D, and talk.

The part that makes it different: the speech recognition runs inside your browser. OpenAI's Whisper model is downloaded once and then executes on your own hardware through WebGPU. Your microphone audio is never uploaded, because there is nowhere to upload it to. There is no server, no account and no subscription.


WHY THIS INSTEAD OF THE OTHERS

Most dictation extensions stream your microphone to a company's servers. That means your half-finished emails, your medical notes, your client work and your private messages are transcribed on someone else's computer, under someone else's retention policy.

LocalDictate has no backend at all. Turn off your Wi-Fi after the first model download and it keeps working perfectly. The entire source is on GitHub under the MIT licence, so you do not have to take our word for any of this.


FEATURES

• Whisper on your device — Tiny, Base and Small models, optimised for the browser
• WebGPU acceleration with an automatic WebAssembly fallback, and it tells you which one is running
• Near real-time: it transcribes as you pause, with a live preview while you speak
• Types straight into inputs, textareas and rich editors; copies to the clipboard when a site uses a canvas editor
• Voice activity detection so silence is never transcribed and never costs you compute
• Automatic clean-up: fillers like "um" and "uh" removed, sentences capitalised, punctuation spaced
• Spoken commands: say "new paragraph", "comma", "question mark"
• Custom replacements for names and acronyms the model keeps mishearing
• Hold-to-talk or press-to-toggle, with rebindable keys
• Works in 90+ languages, or detects the language automatically
• Local transcript history you can export or switch off entirely
• Dark and light themes
• Free, open source, MIT licensed, no ads, no upsell


HOW TO START

1. Install and grant microphone access — Chrome asks once
2. Choose a model. Base is the right first choice
3. Wait for the one-time download
4. Click into any text field and press Ctrl+Shift+D

After that you are offline-capable forever.


PRIVACY IN ONE PARAGRAPH

LocalDictate collects nothing. No audio, no transcripts, no usage statistics, no crash reports, no identifiers. The only network request it ever makes is downloading the Whisper model from huggingface.co the first time you select it. Transcript history, if you leave it on, is stored in your browser profile on your computer and can be deleted with one click. Full policy: https://github.com/wushu75/LocalDictate/blob/main/PRIVACY.md


REQUIREMENTS

Chrome 116 or newer. WebGPU is strongly recommended for the larger models but is not required — LocalDictate falls back to WebAssembly on the CPU and still works well with Tiny and Base. The first download is 78 MB to 490 MB depending on the model you choose.


KNOWN LIMITATIONS

Google Docs draws text on a canvas with no editable DOM, so no extension can type into it directly. LocalDictate copies the text to your clipboard and prompts you to paste. Everything else that behaves like a normal text field works.


OPEN SOURCE

Source, issues and roadmap: https://github.com/wushu75/LocalDictate
Licence: MIT
```

---

## Single purpose (required justification)

```
LocalDictate has one purpose: converting the user's speech into text and inserting that text into the field the user has focused. Every permission, script and UI surface serves that single function.
```

---

## Permission justifications

**storage**
```
Stores the user's settings (chosen model, hotkey, language, clean-up preferences) and, if the user leaves it enabled, a local transcript history. All of it stays in the browser profile.
```

**offscreen**
```
Manifest V3 service workers cannot hold a MediaStream or an AudioContext. The offscreen document hosts the microphone capture graph and the speech recognition worker. Without it, on-device transcription is not possible.
```

**activeTab**
```
Identifies which tab the user was typing in so the transcribed text is inserted in the right place.
```

**scripting**
```
Used to ensure the insertion helper is present in the tab receiving dictated text.
```

**clipboardWrite**
```
Fallback path for editors that cannot be typed into programmatically, such as canvas-based Google Docs. The transcript is copied so the user can paste it.
```

**Host permission: huggingface.co**
```
Downloads the Whisper model files once, on first use of each model. Files are cached locally in IndexedDB; no further requests are made and the extension works fully offline afterwards.
```

**Host permission: <all_urls> (content script)**
```
Dictation is meant to work in any text field on any website, so the focus-tracking and text-insertion script must be able to run on any site. It reads only the currently focused editable element in order to place the caret's text, and it transmits nothing anywhere.
```

**Remote code**
```
No remote code is executed. Transformers.js and ONNX Runtime Web are bundled in the extension package. The only remote files fetched are model weight files (.onnx/.json), which are data, not executable script.
```

---

## Data disclosure answers

| Question | Answer |
|---|---|
| Does this item collect personally identifiable information? | No |
| Health information? | No |
| Financial and payment information? | No |
| Authentication information? | No |
| Personal communications? | No |
| Location? | No |
| Web history? | No |
| User activity? | No |
| Website content? | No |

Certifications — tick all three:
- I do not sell or transfer user data to third parties, outside of the approved use cases
- I do not use or transfer user data for purposes that are unrelated to my item's single purpose
- I do not use or transfer user data to determine creditworthiness or for lending purposes

---

## Privacy policy URL

```
https://github.com/wushu75/LocalDictate/blob/main/PRIVACY.md
```

## Homepage URL

```
https://github.com/wushu75/LocalDictate
```

## Support URL

```
https://github.com/wushu75/LocalDictate/issues
```

---

## Assets checklist

| Asset | Size | Status |
|---|---|---|
| Store icon | 128×128 PNG | `icons/icon128.png` ✅ |
| Screenshot 1 — popup, listening | 1280×800 PNG | see `store/screenshots/README.md` |
| Screenshot 2 — dictating into Gmail | 1280×800 PNG | pending capture |
| Screenshot 3 — settings, privacy panel | 1280×800 PNG | pending capture |
| Screenshot 4 — model picker with progress | 1280×800 PNG | pending capture |
| Screenshot 5 — WASM fallback notice | 1280×800 PNG | optional |
| Small promo tile | 440×280 PNG | pending |
| Marquee promo tile | 1400×560 PNG | optional |

---

## Submission notes for the reviewer

```
LocalDictate performs speech recognition entirely client-side using Transformers.js and
ONNX Runtime Web, both bundled in the package under lib/. No remote scripts are loaded and
no user data is transmitted. The only outbound requests are model weight downloads from
huggingface.co, declared in host_permissions and cached in IndexedDB after first use.

The <all_urls> content script is required because dictation is intended to work in text
fields on arbitrary websites. It tracks the focused editable element and inserts text; it
does not read or exfiltrate page content. Full source: https://github.com/wushu75/LocalDictate
```
