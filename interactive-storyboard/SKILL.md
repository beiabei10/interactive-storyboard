---
name: interactive-storyboard
description: Turn a drawn storyboard (PDF or images) into an interactive visual story on one HTML page: guided gestures, synthesized or recorded sound, a chosen look, multilingual text.
---

# Interactive storyboard

Turn a finished storyboard into an **interactive visual story**: a web page where the viewer moves the story on by clicking, holding and dragging on the drawings themselves, with sound, captions/dialogue and transitions that respect the film's rhythm. It is for people who already know what a storyboard is (boards, shots, slugs, notes); it is not an animatic generator.

Example cases by the author of this skill, Ding Yilin: *The Joy of the White Room* and *Fahrenheit 451* (scenes 1, 8, 9) — see [`examples/`](../examples/) and the live versions on [dingyilin.netlify.app](https://dingyilin.netlify.app).

## 0. Ask before building (one round, then go)

Ask only what changes the build; take sensible defaults for the rest and say which.

1. **Which scenes / boards** (a long storyboard: pilot one scene first, then do the others the same way).
2. **Look**: one of the presets below, or "keep my drawings as they are".
3. **Text**: dialogue only / narration / both / none. Use the storyboard's own lines; never invent dialogue or narration.
4. **Sound**: the user's recordings (ask for the files and which board each belongs to) or synthesized sound.
5. **Languages**: always ask which languages the piece needs; there is no default set. Build only those. Every on-screen string (titles, slugs, hints, balloons, credits) lives in a per-language table and is switchable live; with more than one language the starting one can follow a host page (`?lang=`, a postMessage, or `<html lang>`). Translations you make are drafts the user should check.
6. **Where it will live**: a shareable page only, or also embedded in the user's own website (then ask for the site package and edit *their* copy).
7. **Credit line**: use exactly the wording the user gives (e.g. "Made by …"); do not add roles they did not list.

## 1. Read the storyboard

- **Storyboarder / PDF exports**: `pdfimages -png` gives each board at full size including the NOTE area; find the drawing frame once (scan rows/columns for the frame lines) and crop every board the same way to the drawing area (keep a 16:9 window if the stage is 16:9). Render the pages with `pdftoppm -r 150` and *read* the notes, slugs (EXT./INT. … – DAY/NIGHT), durations and dialogue; `pdftotext -layout` gets the slugs.
- Make a contact sheet and a gridded view (lines every 0.1 of width/height) of each board: every later coordinate (where a figure stands, where to tap, where a screen is) is measured on these, in **image fractions**, never guessed.
- Write a beat list: board → what happens → the one gesture that would make the viewer *do* that action → sound → text.

## 2. Shots before effects

- **Find the long takes.** Consecutive boards with the same setup and moving subjects (A→B→C of people walking) are one continuous shot: build one clean background (patch the subject out using the other boards), cut the subject out and move it, camera drifting with it. Do not cross-dissolve whole boards there.
- **Close shot → wide shot of the same subjects** (or the reverse) is one move: frame the wide board zoomed so its subjects have the same size and position as in the close board (compute zoom and centre from the measured head/feet positions), finish the cross-fade at that matched framing, *then* pull back. Check the matched frame in a screenshot; ghosts mean the numbers are off.
- Boards that must read as one move share one palette (process them with the same preset colour).
- When the user says a board shows the *back* of a TV, a door seen from behind, etc., draw nothing that the board does not show: light and sound can suggest what is off-screen.

## 3. Look presets

All presets keep the user's line drawing; they never redraw it.

- **Film / riso** (the White Room look): grade per scene (bright saturated colour against gloom), soft paper texture, gentle grain; occasional — not constant — gate weave, flicker and faint scratches; a registration shadow. Effects are rare and quiet: constant flicker or shake is tiring.
- **American comic**: bilateral smoothing; an ink layer from a dark threshold *plus* black-hat so thin pencil strokes get inked; a flat spot colour in the drawn areas, 45° Ben-Day dots in the midtones and a light screen over the whole panel, denser at the edges; the colour plate offset 1–2 px from the ink plate; output ~1.5×. One spot colour per location/time (warm day, blue night, green CCTV, red alarm…). Lettering: yellow caption boxes for slugs, comic SFX words in the picture, speech balloons for dialogue.
- **Pixel**: downscale to a small grid with a limited palette and ordered dithering, upscale nearest-neighbour; text in a pixel font.

Page chrome follows the preset (screening room, newsprint page, game screen). Web fonts load with a 2–3 s fallback race so the story never waits on them.

## 4. Build: one HTML page, one canvas

- A 16:9 stage, a canvas, a subtitle/hint area under it, a language switch (if more than one language) and a sound toggle.
- **Scene state machine**: each scene is an object with `enter / render(t) / tap / drag / down / up / hover / key`; `go(scene)` switches; `st()` is time in scene. A camera helper (`setCam(img, zoom, cx, cy)`, `toScreen`, `toImg`) so every hit area and effect is placed in image fractions and follows the camera.
- **Every scene has a way out**: a keyboard key and a slow automatic fallback (10–20 s) so nobody is ever stuck, plus a "skip" link on the long ones.
- Wrap every sound call so a missing audio context or a failed decode can never break the story.
- Load images with `<img>` (works on `file://` too); audio via `fetch` + `decodeAudioData`, falling back to `<audio>` elements when fetch is blocked.

## 5. Interaction design

- **The gesture is the story action.** Run = rapid clicks; hold your breath = don't touch; lean out to look = hold; climb = repeated drags up; hand something over = carry it; turn a page = drag across the book; switch it off = hold on the hand on the button. Prefer these over generic "click to continue"; keep plain taps for dialogue and for beats that are pure rhythm.
- **Carry the user's own drawing, never an invented prop.** Cut the figure/object out along its outline (polygon traced on the gridded zoom, soft edge), add a thin white border like a paper cut-out, lift it with a small shadow and tilt while dragging, leave the bare paper where it was, snap back if dropped wrong. Check the cut against the board: do not include stairs, furniture or another character's hand by mistake.
- Effects added on top of a board (a screen lighting up, radio waves, a viewfinder) must follow the drawing's geometry — clip a screen glow to the screen's *perspective quad*, not a rectangle — and should be pointed out to the user as additions.
- **Hints say where and how**: "Hold on the fish and scribble back and forth", not "interact". Bigger than body text, under the picture.
- **On-picture guides, quiet by default** (think *What Remains of Edith Finch*): a small sign on the touch point that already says the action — a pulse for tap, a filling ring for hold, chevrons flowing in the drag direction, a turning arc for circling (in comic style: ink "look here" ticks). A hand demonstrates the gesture twice when the step starts, then steps back; it returns only after ~3 s of hesitation. Hide everything while the finger is down. Never cover the picture with long dashed paths.

## 6. Transitions

- Match the tone: soft by default (dissolves where the new panel settles from slightly larger, ink-out/ink-in, a light bloom for a TV coming on or a fire catching). Keep one hard cut (negative frame, speed lines, shake) for the single shock of the scene. All hard cuts = stiff.
- A continuous push across to the next space is one long move: whatever lies between (a wall) takes its colour and texture from the destination panel and melts into it; no gutters in the middle of a move.
- Never show the black edges of a rotated or zoomed image: clamp the camera.

## 7. Sound

- Recorded sound: trim to the action (a door opening synced to the drawn handle), level everything, loop ambiences with fades.
- Synthesized sound (WebAudio, no files): steps, paper, doors, mechanical hounds, alarms, heartbeats, TV murmur… Randomize every hit (timing, pitch, envelope) and shape envelopes unevenly so nothing sounds machine-repeated; avoid clicks and dry beeps in quiet scenes.
- Use **sound bridges** for meaning (a washing machine slowly becoming the sea of the dream; a sound continuing over the title). Ask before adding sounds the storyboard does not imply.

## 8. Text

- Dialogue: speech balloons (comic) or subtitles (film), tail pointing at the speaker's head, kept inside the panel, wrapped by measuring, never starting a line with closing punctuation. Fill each balloon's body and tail separately so no stroke crosses the text.
- Slugs as caption boxes in the corner; keep them off important drawing (skip the caption on a board where it would cover the subject).
- Quotations you add (e.g. what characters recite) must be public domain and flagged to the user.

## 9. Performance

- Cap the canvas at ~1600 px across (`DPR = max(1, min(devicePixelRatio, 2, 1600 / cssWidth))`).
- Pre-render static overlays (paper texture, vignette) once per size and lay them on each frame; never rebuild gradients or patterns per frame.
- For dissolves keep the previous frame at half size, not a full copy every frame.
- Lazy-load the story on the host page (poster + "Enter the story" button); each scene page loads only its own images.

## 10. Test like a viewer

- A headless-browser run (e.g. Playwright) with real mouse gestures (clicks, holds, drags measured from `toScreen`) through the whole story, in every language, failing on any page error. Expose a test-only `__go(scene, state)` to jump to a beat.
- Screenshots of every key frame *and of the middle of each transition/match*; look at them.
- Syntax-check the script after edits (`node --check`). Do not put `//` comments in the middle of a one-line statement (it silently comments out the rest of the line and the page stays blank).
- Measure frame rate before/after heavy changes. Say plainly that sound and feel were checked by measurement and screenshots, not by playing it.

## 11. Deliver

- A shareable hosted page for review; for the user's website, a zip of *their* site package with the story in its own folder (e.g. `story/<project>/`), a lazy-loading embed block, and nothing else changed. If the user has adapted the story inside their site, always edit that copy (3-way merge your changes onto it), never overwrite it with the original.
- Several scenes: one page per scene sharing one assets folder, chained (end of scene → next scene, language kept; the last returns to the first). Each scene's cover needs one click to unlock audio after navigation.
- Never deploy anywhere on the user's behalf; hand over the zip.
- A single file opened on its own will be black: the page needs its `a/` assets folder next to it. Say so when handing over files.
