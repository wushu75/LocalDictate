# Contributing to LocalDictate

Thanks for looking. LocalDictate is small on purpose, and the bar for a change is simply:
does it make dictation better without weakening the privacy guarantee?

## The one rule

**No data leaves the device.** A pull request that adds analytics, error reporting, a
remote config fetch, a font from a CDN, or any other network call beyond the model
download will be closed, however useful it is otherwise. If you need a runtime asset,
vendor it into `lib/` in `scripts/vendor.mjs`.

## Getting set up

```bash
git clone https://github.com/wushu75/LocalDictate.git
cd LocalDictate
npm install
npm run build
```

Then load the folder at `chrome://extensions` with **Load unpacked**. There is no watcher:
edit a file, hit the reload arrow on the extension card, and reload the page you are
testing on if you changed the content script.

### Where to look when debugging

| Layer | How to open its console |
|---|---|
| Service worker | `chrome://extensions` → LocalDictate → *service worker* |
| Offscreen document | Same card → *offscreen.html* (appears while dictation is active) |
| Inference worker | Inside the offscreen document's DevTools, under Sources → Threads |
| Popup / options | Right-click the popup → Inspect |
| Content script | The page's own DevTools |

## House style

- Plain ES modules, no bundler, no TypeScript, no framework. The code you read is the
  code that runs.
- Two-space indent, single quotes, semicolons.
- Comments explain *why*, not *what*. If a line needs a comment to say what it does,
  rewrite the line.
- UI copy: sentence case, active voice, no exclamation marks, no apologising in errors.
  Say what happened and what to do about it.
- Colours and type come from `src/common/theme.css`. Do not introduce new hex values in
  component stylesheets.

## Before you open a pull request

Test against this list — it covers the things that break most often:

- [ ] A plain `<textarea>` (this README's issue form will do)
- [ ] A single-line `<input>` mid-string, with the caret in the middle
- [ ] A rich contenteditable (Notion, Slack, a GitHub comment box)
- [ ] An iframe-based editor
- [ ] Toggle mode and push-to-talk
- [ ] WebGPU forced on, then WASM forced on, from Settings → Engine
- [ ] A cold start with no model cached, including the progress bar
- [ ] Dark and light themes
- [ ] Keyboard-only operation, and `prefers-reduced-motion`

Include in the PR: what you changed, why, which of the above you exercised, and a
screenshot or clip if it is visual.

## Commit messages

Conventional-ish and readable: `fix(content): keep caret position after insertion into
CodeMirror`. Squash noise before pushing.

## Reporting bugs

Open an issue with your Chrome version, OS, model, backend (the WebGPU/WASM chip in the
popup), the site you were dictating into, and what you expected to happen. Console output
from the relevant layer above is worth more than a paragraph of description.

## Security

If you find something that leaks data off the device, please open an issue marked
**security** rather than a silent PR, so users can be told quickly.

## License

Contributions are accepted under the [MIT License](LICENSE).
