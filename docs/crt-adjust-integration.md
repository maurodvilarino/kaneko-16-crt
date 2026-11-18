# CRT Adjust integration — Kaneko 16

This branch adds the core-side `crt_adjust.sv` geometry engine from
rmonic79/MiSTer-CRT-Adjust to the Kaneko 16 video output.

## OSD controls

- CRT Adjust: Off / On (default Off)
- H-Size: -16..+15; positive widens, negative narrows
- H-Position: -48..+48 pixels; positive moves right. The 7-bit OSD field uses the upstream wrap encoding: 0..48 for 0..+48 and 80..127 (two's complement) for -48..-1.
- V-Shift: -16..+15 lines; positive moves down

The controls occupy status bits 38..55. The original options use bits only
through 37, so these fields do not overlap existing settings.

## Integration details

The insertion is after the existing two-clock video alignment pipeline in
`Kaneko16.sv`, where palette RGB and HS/VS/DE are already aligned. True VBlank
is delayed by the same two clocks and connected separately from combined
blanking.

The Kaneko pixel clock is 6 MHz and `clk_sys` is 48 MHz (8 system clocks per
pixel). The CRT read-rate accumulator therefore uses a neutral period of
32 quarter-cycles, not the 64-quarter value from the 96 MHz reference example.
Its phase is reset from `crt_adjust.hs_ref_out`.

The module is built with `VTOTAL=264`, `HTOTAL=384` and
`HPOS_MODE=0` (HSync-shift mode), initially chosen as the conservative mode
for the narrow 256-pixel game geometry.

## Important validation

This is an RTL integration, not a Quartus-compiled bitstream. Run the project's
Verilator lint and test targets, then compile the Quartus project and test on
the actual VGA CRT. In particular verify all games at H-Position extremes,
H-Size extremes, and V-Shift extremes. If a 320-pixel game shows edge clipping,
evaluate `HPOS_MODE=1` for that geometry.

For the analog raw-video path, keep rotation/framebuffer scaling out of the way
and use the core's Direct Video mode as appropriate for the MiSTer setup.
The core-side adjustment is inserted in the emu video stream, so downstream
video paths that consume that stream may also see the adjusted geometry.

The source module retains its GPL-3.0-or-later license.
