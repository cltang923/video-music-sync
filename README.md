# Video Music Sync

A single-HTML-file mobile tool. If you have a video whose background music is drowned out by voices or ambient noise, and you separately have a clean audio file of that same music, this tool automatically finds the right offset and mixes them together — no more ear-matching segments by hand.

## How it works

- Pure front-end, no backend server, and no third-party engine to download. Open it in your phone's browser, pick two files from your camera roll (the video + the music file), and everything (audio matching, mixing, rendering) runs locally on-device using standard browser APIs — Web Audio API for decoding/mixing and MediaRecorder for producing the final video. Neither file is ever uploaded anywhere.
- An earlier version of this tool used ffmpeg.wasm, but it hung indefinitely during initialization on iOS Safari/Chrome (both share the WebKit engine) — a known ffmpeg.wasm compatibility issue on iOS. This version avoids that entirely by sticking to native Web APIs, which are well supported on iOS.

## Alignment logic

1. Extract a short sample clip from the video (default: starting at 20s, 25s long — pick a segment with just background music, no talking).
2. Decode both the video's audio track and the full music file via the Web Audio API, downsample to 200Hz mono, normalize, and run cross-correlation to find where the sample best matches within the music file.
3. Compute the offset needed to align the music track to the video's timeline, plus a ±5s manual fine-tune slider (auto-detection isn't always perfect, especially with a lot of background noise).
4. Preview the alignment first (plays both tracks in sync for 8 seconds using native browser playback) before committing to the full "Render mixed video" step.

## Rendering (real-time, not instant)

The "Render mixed video" step works by actually playing the video and music together in sync, in real time, through a Web Audio mixing graph (with independently adjustable volume for the original video audio vs. the music track), while `MediaRecorder` captures the result into a new video file. This means rendering a 3-minute video takes about 3 minutes — there's no way to render faster than real-time with this approach, unlike a native ffmpeg encode. For short clips (a few minutes) this is a fine trade-off in exchange for something that actually works reliably on iOS.

## Deploying to your phone (GitHub Pages)

The tool is just an `index.html`, so it needs to be served from a URL for a mobile browser to open reliably (opening it via `file://` directly can break file picking on some mobile browsers). The simplest free option is GitHub Pages:

1. Create a new public GitHub repo (can be done entirely from the GitHub mobile app or github.com in a mobile browser — no computer needed).
2. Upload `index.html` to the repo root.
3. Repo Settings → Pages → set Source to the `main` branch / root. Save, and you'll get a URL like `https://<your-username>.github.io/<repo-name>/`.
4. From then on, just open that URL in your phone's browser whenever you need it — no computer required.

> Note: on the GitHub Free plan, Pages only works from a public repo (Pages on a private repo requires a paid plan). If you want the source code private but the page public, use something like Vercel or Netlify instead — both support deploying a public site from a private repo on their free tier.

## Known limitations

- Rendering happens in real time (see above), so it takes as long as the video's own duration. Long videos also mean the browser tab holds more in memory while recording, which could in theory cause it to reload on very long clips — for videos in the few-minutes range this hasn't been an issue.
- Cross-correlation alignment confidence drops if there's a lot of background noise, or if the sample clip overlaps with talking over the music (the tool flags this as "low confidence"). If that happens, try a cleaner sample segment or just use the manual fine-tune slider.
- This currently only handles alignment + mixing, not trimming unwanted footage at the start/end.
- Nothing is ever written to your phone's storage except when you explicitly tap Download or Save/Share — all intermediate processing stays in the browser tab's memory and is discarded when the tab closes.
- How you obtain a clean copy of the music track is outside the scope of this tool — make sure you have the rights to use it.
