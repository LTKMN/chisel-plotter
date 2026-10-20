# chisel-plotter

Browser-based GRBL pen-plotter controller over Web Serial. Single-file app: everything
lives in `index.html` (HTML + CSS + JS, no build step). Forked from ChiselLaser
(`C:\brennan\mesa\ChiselLaser`) — the serial/streaming/tracer core is shared, so fixes
there may apply here too.

## Project facts

- **The pen is the Z axis.** Pen heights are *work* Z: pen-down = 0 (set by RUN) and
  pen-up is a lift above it. No M3/M5/S anywhere —
  never emit spindle/laser commands.
- A feed move previews as drawn when Z < midpoint of up/down (or when the file never
  sets Z). Go-to-zero, job end and STOP lift first; outline never touches Z.
- **Jobs start where the pen is.** No zero buttons. OUTLINE (pen hovering) sends
  `G10 L20 P1 X0 Y0` and never moves Z. RUN (pen on the paper) sends
  `G10 L20 P1 X0 Y0 Z0`, so pen-down = the height it was pressed from.
- Safety lifts (STOP, GO TO ZERO, alarm unlock) use `liftIfLow` — upward only, since
  a stale Z zero could put "up Z" below the pen.
- G10 writes EEPROM; GRBL drops serial RX during the write, so `pump()` holds the
  queue until an EEPROM-writing line is acked (`EEPROM_RE`).
- **No true machine homing.** Treat "home" / "return home" as **work zero** —
  `G90 G0 X0 Y0` (same target as the `◎` goto-zero button) — never a `$H` cycle.
- Target machine: a Cetus 3D printer converted to GRBL 1.1h (Uno + CNC shield,
  bed-slinger: X carriage, Y bed, belt Z; 80 steps/mm on all axes, `$3=1`).
