# filmi-entry-assets

Static files for the Filmi Entry Google Flow tool, fetched at a pinned commit.

The tool reads them through esm.sh (`https://esm.sh/gh/ashokaitoolss-cell/filmi-entry-assets@<commit>/<path>?raw`), because Flow's tool sandbox blocks jsDelivr's `/gh/` path. jsDelivr (`https://cdn.jsdelivr.net/gh/...@<commit>/<path>`) is the fallback.

- `templates/index.json` lists the scenes in deck order. A playable scene points at its `template.json`; a coming-soon card has a title and an image.
- Each scene folder holds its `template.json`. Every path in it is relative to that folder.

## template.json

| Field | Meaning |
|---|---|
| `fps`, `width`, `height`, `totalFrames` | The finished scene: 24 fps, 9:16, exact frame count |
| `video` | The whole scene. It plays on the waiting screen and supplies every `pass` frame (frame N of the output is frame N of this file) |
| `audio` | The score. Sample 0 is frame 0, same length as the video |
| `thumbnail`, `title`, `lead`, `name`, `meta` | The card on the pick screen |
| `hero`, `keep` | Words for the face-swap prompt. Describe only what is in every swap shot: Omni adds anything named in `keep` that a shot doesn't have |
| `continuity` | The prompt's closing instruction. "One continuous shot, no cuts." for a single shot; for a clip with several shots, say to keep every hard cut on its frame |
| `endCard` | `startFrame`: where the name card starts. `bandHeight`: a solid black band at the bottom of the 1080x1920 frame that holds the name, in pixels. `fadeFrames`: how long the name takes to fade in. The picture is never darkened, so the face stays clear |
| `guardMinDb` | Optional. The repaint check's threshold in dB, default 16. A `swap` segment can set its own, which wins. Set it from clean renders: a shot with quiet edges scores about 30 dB when clean, so 16 would let a repaint through |
| `guardEdge` | Optional, on a `swap` segment: `left` or `right` scores only that edge of the frame, default both. Use it when the set's landmark is on one side and the other edge changes with the person's build |
| `segments` | Cover frames 0 to totalFrames-1 in order, no gaps. See below |

Segment modes:

- `pass`: frames come from `video`.
- `swap`: frames come from the edited version of clip `video` (`clipFrames` long; use 144 = 6.0 s. A 192-frame, 8.0 s clip broke a shot on 1 of 2 renders). A clip can start with lead-in frames that `clipStart` skips, so Omni's first frames aren't used. Frame N of the output is clip frame `clipStart` + (N - startFrame). Several segments can share one clip, so several shots cost one edit. Frames no segment uses are mirror padding and are dropped. If the edited clip comes back with a different frame count, read edited frame round(i x edited / clipFrames).
- `hold`: every frame shows frame `holdClipFrame` of the edited clip `holdClip`, with a slow push-in from 1.00 to `pushTo` around the height `pushOriginY` (0 = top). This is the title freeze: pick a frame where the face is big and clearly lit.
