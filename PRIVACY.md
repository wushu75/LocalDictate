# Privacy Policy — LocalDictate

**Last updated: 23 July 2026**

LocalDictate is a Chrome extension that converts speech to text entirely on your own
device. This document describes exactly what it does with your data. It is short because
there is very little to describe.

## What LocalDictate collects

Nothing. There is no server, no account, no analytics library, no crash reporter and no
remote logging in this extension. The developer receives no data about you, including
whether you have installed it.

## What happens to your audio

1. Microphone audio is captured in an offscreen document inside the extension.
2. It is converted to 16 kHz mono and passed to a Web Worker in the same extension.
3. A Whisper model in that worker turns it into text.
4. The audio buffer is discarded. It is never written to disk and never transmitted.

## What happens to your transcripts

Transcribed text is inserted into the page you were typing in. If "Keep a local history"
is enabled — it is on by default — the last 100 transcripts are stored in
`chrome.storage.local`, which lives in your browser profile on your computer. You can
view, export or delete them at any time in Settings → Storage & privacy, and turning the
setting off stops any new ones being kept.

## Network requests

LocalDictate makes exactly one kind of network request: downloading a Whisper model from
`huggingface.co` the first time you select it. This is why `huggingface.co` appears in
the extension's host permissions. The download contains no identifying information beyond
what any browser sends when fetching a file (IP address, user agent), and is handled by
Hugging Face under [their privacy policy](https://huggingface.co/privacy).

Once a model is cached in IndexedDB, LocalDictate works with no network connection at all.

## Permissions and why they exist

| Permission | Why |
|---|---|
| `storage` | Saving your settings and local history. |
| `offscreen` | Hosting the microphone stream and inference worker, which a service worker cannot do. |
| `activeTab`, `scripting` | Knowing which tab to type into. |
| `clipboardWrite` | The paste fallback for editors that cannot be typed into directly. |
| `<all_urls>` content script | Dictation works on any site, so the focus tracker and insertion code must run on any site. It reads only the element you have focused, and sends nothing anywhere. |
| `huggingface.co` host access | The one-time model download. |
| `notifications` (optional) | Only requested if you enable notifications. |

## Children

LocalDictate is not directed at children and collects no data from anyone.

## Changes

Any change to this policy will appear in this file and in the repository's commit history,
which is public.

## Contact

Questions, or something in the code that contradicts this document:
https://github.com/wushu75/LocalDictate/issues
