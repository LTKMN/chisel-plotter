# chisel-plotter

Browser-based GRBL pen-plotter controller over Web Serial. Single-file app: everything
lives in `index.html` (HTML + CSS + JS, no build step). Forked from ChiselLaser
(`C:\brennan\mesa\ChiselLaser`) — the serial/streaming/tracer core is shared, so fixes
there may apply here too.

## Project facts

- **The pen is the Z axis.** Pen heights are *work* Z: jog the pen onto the paper,
  ZERO Z, so pen-down = 0 and pen-up is a lift above it. No M3/M5/S anywhere —
  never emit spindle/laser commands.
- A feed move previews as drawn when Z < midpoint of up/down (or when the file never
  sets Z). Anything that travels (outline, go-to-zero, job end, STOP) lifts first.
- **Jobs start where the pen is.** OUTLINE and RUN send `G10 L20 P1 X0 Y0` first, so
  XY work zero is always the position they were pressed from — there is no manual
  XY zero button. Z is never auto-zeroed (pen-down comes from ZERO Z on the paper).
- **No true machine homing.** Treat "home" / "return home" as **work zero** —
  `G90 G0 X0 Y0` (same target as the `◎` goto-zero button) — never a `$H` cycle.
- Target machine: a Cetus 3D printer converted to GRBL 1.1h (Uno + CNC shield,
  bed-slinger: X carriage, Y bed, belt Z; 80 steps/mm on all axes, `$3=1`).
