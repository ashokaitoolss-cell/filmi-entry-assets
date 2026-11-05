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
| `thumbnail` | The card image on the pick screen |
| `hero`, `keep` | Words for the face-swap prompt. Describe only what is in every swap shot: Omni adds anything named in `keep` that a shot doesn't have |
| `segments` | Cover frames 0 to totalFrames-1 in order, no gaps. `pass` frames come from `video`. A `swap` segment has its own clip of `clipFrames` frames (144 = 6.0 s, the length proven frame-exact through Omni's edit). That clip goes to the face swap. Its first endFrame-startFrame+1 frames are the shot and land on startFrame onward; the rest is mirror padding and is dropped. If the edited clip comes back with a different frame count, read edited frame round(i × edited / clipFrames). A `hold` segment repeats finished frame `holdFrame` (before the end card is drawn) with a slow 6% push-in. The smoke walk ends on a hold of the last hero frame, so "Introducing [name]" sits on the hero |
