# Chisel Plotter

A GRBL pen-plotter controller that runs entirely in the browser over the Web Serial API.
A fork of [Chisel Laser](https://laser.brennan.computer) with the laser swapped for a
pen on the Z axis.

---

## Running it

```sh
# Web Serial works from file:// in Chrome, so just open index.html locally
# or serve it:
python3 -m http.server 8000   # then http://localhost:8000
```

**Requires Chrome, Edge, or another Chromium browser.** Safari and Firefox do not
implement Web Serial.

Everything is client-side. Nothing is uploaded; images you trace never leave your machine.

---

## Using it

1. **CONNECT**, pick the plotter's USB port. On connect it sends `$$` and learns
   `$130/$131` (work area, drawn as the dashed envelope in the preview).
2. **Set the pen height:** in the Pen panel, nudge Z down (Z▼ or PgDn, step 0.1 for the
   last bit) until the pen just touches the paper, then **ZERO Z**. Pen-down is now `0`; pen-up defaults to `3` mm.
3. Jog to where the drawing's origin (bottom-left for traced images) should be. There's
   no XY zero to set: **OUTLINE** and **RUN** both zero XY at the pen's current position
   before they move.
4. Drop a `.gcode`/`.nc` file **or an image** onto the preview. Images open the vector tracer.
5. **▢ OUTLINE** walks the job's bounding box with the pen up, looping until you click it
   again — use it to position the paper.
6. **▶ RUN** streams the job. The browser must stay open; it feeds the machine in realtime.

### Pen

| Setting | Meaning |
|---|---|
| up Z | work Z when lifted (travel moves) |
| down Z | work Z touching the paper — `0` after ZERO Z |
| z feed | how fast the pen lowers, mm/min (lifts are rapids) |

**PEN UP / PEN DOWN** move there directly; the lit button shows the current state.
Outline, GO TO ZERO / `◎`, job end and **STOP** all lift the pen first so it never drags.

G-code from other tools works if it lifts and lowers the pen with Z. Files with no Z
at all preview every feed move as drawn.

### Controls

| Control | Notes |
|---|---|
| Jog pad | XY `$J=` jog commands. Arrow keys work too; Esc = jog cancel |
| Z▲ / Z▼ | In the Pen panel, with their own step (0.1–5 mm) at the pen z feed. PgUp/PgDn too |
| GO TO ZERO | Pen up, then `G90 G0 X0 Y0` — back to where the last outline/run started (same as `◎` in the jog pad) |
| ZERO Z | `G10 L20 P1 Z0` — current Z becomes pen-down |
| Alarm popup | Appears only while GRBL is in ALARM. UNLOCK + PEN UP sends `$X` then lifts; LATER hides it (click the blinking badge to reopen) |
| HOLD / RESUME | Realtime `!` / `~` |
| STOP | Big button pinned atop the left column. While jogging: jog cancel (no reset, no alarm). Otherwise: feed hold, soft reset 400 ms later, then pen up |
| Overrides | Live feed-rate trim while running |
| Console | Full history (↑/↓). Error and alarm codes are decoded to plain English |

### Vector tracer

Drop an image → threshold it → trace contours → generate G-code.

- **Threshold** — auto-set per image via Otsu's method, then adjustable live.
- **Invert** — trace light areas instead of dark.
- **Despeckle** — drop contours below N mm².
- **Simplify** — Douglas-Peucker tolerance in mm.
- **Smoothing** — Chaikin corner rounding, capped so straight edges stay straight.
- **Mode** — outline, hatch fill, or both. Set fill spacing to roughly your pen's line width.
- **Width / draw feed / passes** live in the left sidebar and stay adjustable after the
  tracer closes — changing them (or the pen heights) regenerates the job in place.

Each path is: travel pen-up → lower → draw → lift. Paths are ordered greedy-nearest and
each closed loop starts nearest the previous path's end, to cut travel time.
