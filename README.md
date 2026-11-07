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
| `remove` | Optional. A sentence for the prompt, naming what must come off the hero so the face shows, such as sunglasses. Describe `hero` and `keep` without that item |
| `continuity` | The prompt's closing instruction. "One continuous shot, no cuts." for a single shot; for a clip with several shots, say to keep every hard cut on its frame |
| `endCard` | `startFrame`: where the name card starts. `bandHeight`: a solid black band at the bottom of the 1080x1920 frame that holds the name, in pixels. `fadeFrames`: how long the name takes to fade in. The picture is never darkened, so the face stays clear |
| `guardMinDb` | Optional. The repaint check's threshold in dB, default 16. A `swap` segment can set its own, which wins. Set it from clean renders: a shot with quiet edges scores about 30 dB when clean, so 16 would let a repaint through |
| `guardEdge` | Optional, on a `swap` segment: `left` or `right` scores only that edge of the frame, default both. Use it when the set's landmark is on one side and the other edge changes with the person's build |
| `segments` | Cover frames 0 to totalFrames-1 in order, no gaps. See below |

Segment modes:

- `pass`: frames come from `video`.
- A `swap` segment can set its own `keep`, which wins over the scene's. Use it when a shot lacks something the scene's `keep` names: naming smoke on a shot with none repainted the background on most test runs.
- `swap`: frames come from the edited version of clip `video` (`clipFrames` long; use 144 = 6.0 s. A 192-frame, 8.0 s clip broke a shot on 1 of 2 renders). A clip can start with lead-in frames that `clipStart` skips, so Omni's first frames aren't used. Frame N of the output is clip frame `clipStart` + (N - startFrame). Several segments can share one clip, so several shots cost one edit. Frames no segment uses are mirror padding and are dropped. If the edited clip comes back with a different frame count, read edited frame round(i x edited / clipFrames).
- `hold`: every frame shows frame `holdClipFrame` of the edited clip `holdClip`, with a slow push-in from 1.00 to `pushTo` around the height `pushOriginY` (0 = top). This is the title freeze: pick a frame where the face is big and clearly lit.

## Fields a `swap` segment can add (v1.24)

These turn on the keyframe and paste method for that shot. A segment without `head` works the old way.

| Field | Meaning |
|---|---|
| `crop` | A clip file. Omni edits this enlarged view of the head instead of `video`. `video` stays the template's own footage |
| `still` | One frame of the shot as a JPEG, the base for the keyframe stills. If the segment has `crop`, it is a frame of the crop |
| `stillHead` | `[x, y, r]`: where the head sits in `still`. `x` and `y` are fractions of its width and height, `r` a fraction of its width |
| `stills` | How many chosen stills Omni gets, 1 or 2 |
| `head` | `{cx, cy, ch, fh}`: arrays with one number per clip frame, in 720 x 1280 pixels. The head's centre, the height of the crop window, the face height |
| `oval` | `[a, b, shift]`: the pasted head's soft oval. Radii are `fh * a` and `fh * b`, centre `(cx, cy + fh * shift)`. Lower `b` or a more negative `shift` keeps more of the template's shirt |
| `tint` | Optional `[r, g, b]`: multipliers on the pasted head, fading out on bright pixels, to match the shot's light. The same for every face, so differences in skin between people stay. Not yet seen on a darker face |

`headSameDb` on the scene: a shot fails its check if the head inside the oval differs from the template by less than this, averaged over 8 frames, meaning Omni left it about unchanged. For these segments `guardMinDb` and `guardEdge` no longer apply, because everything outside the oval is the template's own pixels. It catches nothing else, so a wrong-looking head is left to the visitor's eye and Redo.

## Fields for one edit with several shots

Every field is optional. A template without them works as before.

| Field | Meaning |
|---|---|
| `omniModel` | The Omni model name for the edit |
| `sameSentence` | A sentence added to the prompt after the hair sentence, for a clip with several shots |
| `clipCuts` | The clip frames where a shared clip has a hard cut. After the edit, the tool finds where each cut landed within 3 frames and repeats the neighbouring frame, so a frame from the wrong shot is never pasted |
| `key` | `{a, b, shift, lo, hi}`: below the cheeks, inside this larger oval, pixels the edit changed by more than `lo` to `hi` also paste, computed at 180 x 320. Long hair crosses the head oval this way, while small drift on the collar stays the template's |
| `relight` | On a `swap` segment: `true` matches the edit's broad light to the template's before the paste, so the pasted oval doesn't show when the edit changed the shot's light |
| `waitSec`, `redoWaitSec` | The waiting screen's countdown for a new render and for a redo, in seconds |

Swap segments that share one `crop` are one edit: one clip, one run. `head` arrays are indexed by clip frame, and `clipStart` is the clip frame where the segment starts, so `pass` segments can sit between them. `pass` frames come from `video`, where frame N is output frame N. A segment without `still` makes no stills, and only the photos go to the edit as references. Omni has returned at most 192 frames (8.0 s), so keep such a clip to 192.

`lab/` holds test clips. The app doesn't read it.
