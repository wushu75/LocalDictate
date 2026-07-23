# Store screenshots

Chrome Web Store requires at least one screenshot at **1280×800** or **640×400** PNG,
no alpha channel, no rounded corners, no drop shadow added by you. Up to five.

Capture these five, in this order — the sequence tells the product's story:

| # | Shot | What must be visible | Caption to overlay |
|---|---|---|---|
| 1 | Popup mid-utterance | Level meter lit, lamp amber, "Listening", WebGPU chip | Speak. It types. |
| 2 | Gmail compose with the HUD | Dictated sentence in the body, listening bar at the bottom | Works in any text field |
| 3 | Settings → Storage & privacy | The "What leaves this device" card in full | Nothing is uploaded, ever |
| 4 | Settings → Engine, downloading | Model cards, progress bar mid-download | Download once, then go offline |
| 5 | Popup in light theme | Transcript with text, Copy/Export buttons | Yours to keep or export |

## Capturing them

1. Load the extension unpacked and set the window to exactly 1280×800:
   ```js
   // paste into the DevTools console of the window you are capturing
   window.resizeTo(1280, 800 + (window.outerHeight - window.innerHeight));
   ```
2. Use a neutral page and dummy content — never real email addresses, names or
   message contents. Reviewers reject listings that show other people's data.
3. Capture with Chrome DevTools → Command menu → "Capture screenshot", or your OS
   tool, then confirm the PNG has no alpha:
   ```bash
   python3 -c "from PIL import Image; im=Image.open('1-popup.png'); print(im.size, im.mode)"
   # expect (1280, 800) RGB
   ```
4. Save here as `1-popup.png` … `5-light.png`, and copy the first two into
   `docs/screenshots/` as `popup.png` and `in-page.png` so the README renders them.

## Promo tiles

- Small tile, 440×280: the icon on the #151920 faceplate, wordmark in mono caps,
  tagline "Speak freely. Type instantly." No screenshot content, no gradients.
- Marquee, 1400×560: same treatment, wordmark left, level meter motif right.
