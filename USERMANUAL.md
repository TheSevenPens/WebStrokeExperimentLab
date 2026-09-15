# Web Stroke Experiment Lab — User Manual

A simple web app for verifying that your drawing tablet and its pen are working, and for exploring what data the browser reports about each pen event. Useful when:

- You're troubleshooting a pen that "doesn't feel right" (no pressure, tilt missing, etc.).
- You're comparing two tablets or two drivers.
- You're a developer wanting to see what the Pointer Events API actually delivers on your hardware.

It is **not** a drawing app — there are no colors, brushes or layers. The strokes you make are only meant to show that the pen is reporting what you'd expect.

## Quick start

1. Open the [live app](https://thesevenpens.github.io/WebStrokeExperimentLab/).
2. Draw on the blue canvas with your pen, finger, or mouse.
3. Watch the toolbar readouts update as you draw — they show the raw values the browser is reporting for the current pointer event.
4. Use the **Size** and **Rotation** dropdowns to choose which pen readings drive the brush (see [What drives the brush](#what-drives-the-brush) below).
5. Press **Clear**, or **Delete** / **Backspace**, to clear the canvas.

## Controls reference

The window is in three parts. **Across the top**: Clear, Export, About, and under them the
readouts. **Down the left**, beside the canvas: the settings that decide how a stroke is drawn --
Size, Rotation, Stroke, Edge, Smoothing, Use all pen points and Pointer only. **The rest** is the canvas.

The settings sit beside the ink rather than above it because that is how they are used: read down
a list, changed one at a time, compared between strokes. The readouts are on their own row for the
opposite reason -- they change many times a second, and anything next to them would move as the
values change width.

| Control | What it shows / does |
| --- | --- |
| **Clear** | Wipes the canvas. |
| **Export…** | Save the current canvas as a PNG file, or copy it to the clipboard as an image (paste into chat, an image editor, etc.). The image is captured at your display's full pixel resolution, so stroke detail survives zooming in. Useful for sharing what your pen is producing when reporting a driver issue. |
| **Size** | Which pen reading decides how big the brush is — see [What drives the brush](#what-drives-the-brush). |
| **Rotation** | Which pen reading decides which way the brush points, and whether it is round or an oval — see [What drives the brush](#what-drives-the-brush). |
| **Smoothing** | Filters the positions before they are drawn, as every web drawing library does by default — see [Why slow strokes look rough](#why-slow-strokes-look-rough). Round brush only. |
| **Edge** | Whether the boundary of a stroke is crisp or feathered — see [Why slow strokes look rough](#why-slow-strokes-look-rough). Round brush only. |
| **Use all pen points** | Draw from every position the pen reported, instead of the one per screen refresh a browser hands over on its own — see [Report rate](#report-rate). On by default, whatever the brush, where the browser can supply them. |
| **Stroke** | How the ink between two pen samples is drawn — see [Stroke rendering](#stroke-rendering). Round brush only. |
| **Pointer only (no drawing)** | Shows a red crosshair that follows the reported pointer position and leaves no ink. The crosshair stays visible even while the pen is pressing down, when the system cursor would normally disappear. *Use it to check pointer tracking and latency, or to confirm events are arriving at all, without covering the canvas.* |
| **Type** | `pen`, `mouse`, or `touch` — what the browser thinks the input device is. |
| **X, Y** | Where the pointer is, in CSS pixels, relative to the window. Always to two decimals — see [Position precision](#position-precision). |
| **Precision** | Which grid those positions land on: `CSS pixels`, `screen pixels`, or `sub-pixel` — see [Position precision](#position-precision). |
| **Pressure** | 0.000 – 1.000. A mouse reports `0.5` while a button is held and `0` otherwise. |
| **Tilt X** | -90° to 90°. Left/right tilt of the pen. |
| **Tilt Y** | -90° to 90°. Forward/back tilt of the pen. |
| **Azimuth** | 0° – 360°. Compass direction the pen is leaning. |
| **Altitude** | 0° – 90°. 0° = pen flat on the tablet, 90° = perfectly upright. |
| **Twist** | 0° – 359°. Rotation around the pen's long axis (barrel rotation). |
| **Eraser** | `yes` when the eraser end of the pen is in contact, `no` otherwise. Detected via the eraser bit (32) of `PointerEvent.buttons`. Not all pens have an eraser end, and some drivers report the eraser as a normal tip contact — see [Known quirks](#known-quirks). |
| **Buttons** | The raw `PointerEvent.buttons` bitmask shown in binary (6 bits). From least significant: tip/primary, barrel/secondary, middle, X1, X2, eraser. Handy for spotting which buttons your driver reports. |
| **Points/s** | Two numbers: how many positions your pen reports each second, and how many of those the stroke is built from — see [Report rate](#report-rate). `n/a` means this browser cannot say. |
| **About** | Opens a dialog with Code and Docs links. |

If a value stays at `0` or `---` while you draw, your pen or driver isn't reporting that property.

## What drives the brush

A brush has a size and a direction, and the pen reports several things that could decide either.
**Size** and **Rotation** ask those two questions separately, so any answer to one can be tried
against any answer to the other.

**Size** — how big the brush is, as a number from 0 to 1:

- **Size from pressure** — press harder, draw wider. *Use this to verify pressure sensitivity.* A
  mouse reports 0.5 while a button is held, so it draws a mid-width stroke.
- **Size from tilt altitude** — upright is smallest, flat on the tablet is largest. *Use this to
  verify altitude reporting.* Held upright the brush is at its minimum and the stroke is a hairline;
  that is the reading, not a fault.
- **Fixed size** — half size, whatever the pen says. *Use this to take size out of a question
  entirely*, which is how you tell a shape problem from a pressure problem.

**Rotation** — which way the brush points:

- **No brush rotation** — a round brush, swept along the path as a continuous ribbon. This is the
  only setting where **Stroke**, **Edge** and **Smoothing** mean anything, because it is the only
  one that draws a ribbon; the others stamp.
- **Brush rotation from tilt azimuth** — an oval pointed in the pen's compass direction. *Use this
  to verify azimuth reporting.* The oval should turn as you lean the pen in different directions.
- **Brush rotation from twist** — an oval turned by the pen's barrel twist. *Use this to verify
  twist reporting* — only meaningful on pens that report it (some Wacom Art Pens). Most pens report
  twist as `0`.

Asking for any rotation is what makes the brush an oval: a round brush turned is a round brush, so
there would be nothing to see. The oval is deliberately long and thin so that a small change in the
driving angle is obvious, and **Size** drives its long axis while the short one stays put — which is
what a flattening nib does, and what makes the angle easy to read.

**The combinations are the point.** *Fixed size* with *rotation from twist* is the old Twist to
Brush rotation mode; *size from tilt altitude* with *rotation from tilt azimuth* is the old Tilt
Altitude to Brush size. But *size from pressure* with *rotation from twist* was not available at all
before, and it is the quickest way to see two readings at once.

## Stroke rendering

A pen reports samples, not a stroke. The **Stroke** dropdown picks what is drawn *between* two
samples, which is where a surprising amount of what a stroke looks like is decided. It applies to
the round brush; ask for a rotation and the brush is stamped as an oval, which has no line width to
ramp and no path of its own to fit.

- **Stepped width** — one width for the whole segment, taken from the size reading at its far end.
  Width therefore changes in a step at every sample rather than along the segment, and the edge of
  a stroke is a staircase. At tablet report rates that is everywhere. *This is what naive canvas
  code does, and it is here to be looked at rather than used.*
- **Taper (straight)** — the segment is the region swept between two circles, one at each sample,
  so the width ramps continuously. Consecutive segments share an endpoint **and** a width, so the
  ramp is continuous across the whole stroke. The path itself is still a chord from each sample to
  the next. *Use this to see where the samples actually are:* on anything drawn quickly the corners
  between chords are plainly visible, and you can count the report rate off them.
- **Taper (curved)** — the same taper, with a cubic fitted through the samples instead of chords.
  The corners go away. *Use this to see how much of a stroke's shape is interpolation rather than
  measurement.* **This is what the app opens with**, alongside **Use all pen points**: between them
  they are the closest this app comes to what good drawing software does, which makes them the
  useful thing to compare the other settings against.

The difference between the two taper options is entirely about the path, and it grows with the gap
between samples: a slow stroke on a high-reporting tablet looks the same either way, and a fast one
on a slow tablet does not.

**A note on what Curved costs.** The tangent at a sample is computed from its neighbours, so the
segment ending at a sample cannot be drawn until the next one arrives — the ink lags the pen by one
sample, and the last segment is painted when the pen lifts. That is the price of the curve meeting
its neighbours smoothly, and every application that fits curves to pen input pays it in some form.

The curve fitting is Krita's, by way of the C# implementation in
[PenDynamicsPaint](https://github.com/TheSevenPens/PenDynamicsPaint), and is kept close to that
version on purpose so the two can be compared.

## Report rate

Two numbers, and the gap between them is the interesting part.

**pen** — how many positions your pen sends every second. A mouse is usually around 125. Drawing
tablets are typically 130 to 250, and some are much faster. This is your hardware.

**used** — how many of those the stroke on screen is actually built from. This is normally about 60,
whatever the pen is doing, because it matches how often your screen redraws.

So a tablet reporting 200 times a second has roughly 140 of those readings a second discarded before
anything is drawn. That is not a fault in this app; it is how a web page ordinarily receives pen
input, and the same is true of most drawing done in a browser.

**Why the two numbers differ.** The browser does not hand a web page every reading as it arrives. It
waits until the screen is about to redraw and delivers everything that has happened since in one
bundle. An application that takes one position from each bundle — which is the usual thing to do,
and what this app does — gets the display's rate. The rest are still in the bundle, unopened, and
counting them is how **pen** is measured.

**What the values mean**

- **Numbers** — measured over the last second of movement. They settle after about a fifth of a
  second of drawing.
- **`---`** — nothing is moving, or not enough has happened yet to measure.
- **`n/a`** — this browser cannot report it. Chrome, Edge and Firefox have been able to for years;
  Safari only from **18.2**, so an older iPad or Mac says `n/a`. In those browsers a page cannot
  measure the pen's rate at all — only the display's.
- **pen and used the same** — nothing is being discarded, because the pen is not reporting faster
  than the screen refreshes. Normal for a mouse.

**What the difference costs you.** Every discarded reading is a small piece of the shape of your
stroke that no application ever saw: a change of direction, a moment of pressure. The
**Taper (curved)** option under [Stroke rendering](#stroke-rendering) guesses some of it back by
fitting a curve through the positions it did get, which is what most drawing software does. It is a
good guess, not the real thing.

**Use all pen points** stops discarding them, and it is on when the app opens: **used** sits close
to **pen**, because the stroke is built from every reading the browser had rather than one per
screen refresh.

**Untick it to see what an ordinary web page draws.** **used** drops to one per screen refresh,
where almost everything else on the web sits, and the two numbers separate. Two things are worth
watching as they do. The stroke stops following your hand as closely, most visibly on anything
drawn quickly — slow strokes look much the same either way, because the samples were already close
together. And the difference between **Taper (straight)** and **Taper (curved)** opens up, because
the gaps the curve is there to bridge are suddenly wide. That is the clearest demonstration in the
app of what interpolation is for.

It costs nothing in lag. The extra positions arrived in the same event as the one you were already
being given; they were simply going unopened.

## Position precision

**X, Y** shows where the pointer is to two decimals. **Precision** says something the decimals
cannot: which grid those positions are sitting on.

| | |
|---|---|
| **CSS pixels** | whole numbers as reported. The coarsest of the three — under display scaling one CSS pixel spans more than one screen pixel. A mouse normally reads this. |
| **screen pixels** | your display's own grid. Under scaling these arrive as decimals — `262.857` at 175% — and look like fine measurement while being nothing of the kind. |
| **sub-pixel** | positions between screen pixels, finer than your display can draw. |

**The decimals are a trap, which is why this readout exists.** At 175% scaling a whole screen pixel
is 0.571 CSS pixels, so a position quantised to the screen's grid arrives looking like `1043.428`.
Two decimal places of apparent precision, and not one of them earned. Multiply by your scaling
factor and the whole number underneath appears.

**This is a ceiling on what any web page can draw.** A drawing tablet measures in its own units,
often thousands per inch: a 24-inch display tablet is around 100,000 units across, against 2,560
screen pixels. Wintab hands a desktop application the tablet's own grid. A browser gets pixels.
There is no web API that does better — [Wacom's own web
demo](https://github.com/Wacom-Developer/wacom-device-kit-web) uses the same Pointer Events as
everything else.

Measured on Windows with a Wacom tablet at 175% scaling, this reads **screen pixels**: 359
consecutive samples, every one within 0.0001 of a whole screen pixel. That quantisation is what
makes slow strokes look rough — see [Why slow strokes look
rough](#why-slow-strokes-look-rough) — and **Smoothing** is what reconstructs a path between the
grid points.

## Taking pressure out of the picture

**Size: Fixed size** holds the brush at one width for the whole stroke, so nothing the pen says
about pressure reaches the ink. Position becomes the only thing that can vary.

It is a way of splitting a question in two. If a stroke looks rough and you want to know why, draw
it again with this ticked:

- **Still rough** — the roughness is in the *path*. Hand tremor, or the positions the pen is
  reporting.
- **Now smooth** — the roughness was in the *pressure*. Small changes between one reading and the
  next become changes in width, and a wide brush magnifies them: width is pressure times the
  maximum brush size, so a 1% wobble on a 50-pixel brush is half a pixel of edge, every sample.

The Pressure readout is deliberately left alone while this is on. It reports the pen, not the
brush, and it would be a poor readout that lied about its instrument because a drawing setting
changed.

## Why slow strokes look rough

Draw slowly and the edge of a stroke wobbles. Draw quickly and it comes out clean. The cause is not
your hand, and it is not this app adding anything.

**Your pen's positions are snapped to whole screen pixels before the page ever sees them.** Measured
on Windows with a Wacom tablet: 359 consecutive samples, every one of them landing within 0.0001 of
a whole physical pixel.

That quantisation is invisible at speed and obvious when slow, and the reason is the size of a step
compared to the size of the grid:

- **Slow** — samples land one or two pixels apart, so snapping each one to the grid moves it by a
  large fraction of the gap between it and the next. The direction from one sample to the next
  lurches, and a crisp edge shows every lurch.
- **Fast** — samples land twenty or thirty pixels apart, and the same snapping is a rounding error
  of a fraction of a degree.

**Display scaling hides this.** At 175% scaling a whole screen pixel is 0.571 CSS pixels, so the
positions arrive as `262.857`, `1043.428` — decimals that look like sub-pixel precision and are
nothing of the kind. To check your own machine, multiply **X** by your scaling factor: if the result
is always a whole number, your positions are on the grid.

**This is a ceiling, not a bug in the app.** A drawing tablet measures in its own units, often
thousands per inch. Wintab hands a desktop application the tablet's own grid; a browser gets whole
screen pixels. There is no web API that does better — [Wacom's own web
demo](https://github.com/Wacom-Developer/wacom-device-kit-web) uses the same Pointer Events
everything else does.

**Smoothing is the answer to it**, and not for the reason smoothing usually exists. It is not hiding
a shaky hand; it is reconstructing a path between grid points. Measured against a true straight
line, on a snapped diagonal:

| Off | Light | Heavy |
|---|---|---|
| 0.141 px from the line | 0.082 px | 0.028 px |

**Off** is the raw signal, which is what a testing tool should show by default and what almost
nothing else on the web shows you. **Light** matches the default of
[perfect-freehand](https://github.com/steveruizok/perfect-freehand), the library behind tldraw and
Excalidraw. **Heavy** matches [atrament](https://github.com/jakubfiala/atrament). Every drawing
library in this space filters by default, which is why strokes elsewhere look calmer than yours: you
have been comparing a raw signal against filtered ones.

The cost is honest: the filter is a running average, so the ink follows a little behind the pen, and
it is counted in samples rather than in distance — which means it filters a fast stroke over a
longer distance than a slow one. Krita's stabiliser weights by distance travelled instead and is the
better instrument; this is the one the web actually ships.

**Two things that change how visible the roughness is, neither of which is the cause:**

- **Edge.** A feathered rim has no crisp boundary for the lurching to land on, which is most of why
  a painting application's brush looks kinder than this one's.
- **Brush size.** Width is pressure times the maximum size, so a wide brush magnifies everything.

**And two things that are not involved at all,** both ruled out by experiment: pressure — hold it
constant with **Fixed size** and a slow stroke is still rough — and the choice of **Stroke**
rendering, since stepped, straight and curved all trace the same snapped path.

## OS & browser compatibility

| OS | Browser | Status |
| --- | --- | --- |
| Windows | Chrome, Edge, Firefox | Works. Requires **Windows Ink** enabled in your tablet driver settings. WinTab-only drivers will not report pressure. |
| macOS | Chrome | Works. |
| macOS | Safari | Works with [a known quirk](#known-quirks). |
| Linux | Chrome | Works. |
| Linux | Firefox (Wayland) | Works. |
| Linux | Firefox (X11) | Requires environment variable `MOZ_USE_XINPUT2=1`. |
| iPadOS | Safari | Works (Apple Pencil). |
| Android | Chrome | Works. |

## Known quirks

- **macOS Safari** — If the app loads while the pen is already in contact with the tablet, the pen may be treated as a mouse (no pressure, no tilt). **Workaround:** lift the pen away from the tablet and bring it back into range.
- **Windows, no pressure** — If pressure reads `0.000` or jumps straight to `1.000` with no in-between, the driver is most likely running in WinTab-only mode. Enable Windows Ink in your tablet's driver utility.
- **Apple Pencil twist** — Apple Pencil does not report barrel rotation; **Twist** will stay at `0°`.
- **Most pens, no twist** — Twist requires hardware support (e.g. Wacom Art Pen). Most styli will report `0°`.
- **Eraser detection is driver-dependent** — Pens with a physical eraser end (e.g. many Wacom pens) will set the eraser bit on Windows Chrome/Edge/Firefox with Windows Ink enabled. Apple Pencil has no eraser end. Some pens/drivers map the eraser to a normal tip contact plus a configurable button, so the **Eraser** readout stays `no` even when the eraser is touching the tablet.
- **Mouse / touch values** — A mouse reports pressure `0.5` while a button is held and `0` otherwise, with zero tilt and twist and an altitude of 90°. Touch typically reports no pressure or tilt either. These are not bugs — they reflect what the browser delivers.
- **Zero is a reading, not an absence.** A tilt or twist of `0` can mean the pen is upright and untwisted just as easily as it can mean the property is unsupported. Move the pen and watch whether the number changes; that is what distinguishes the two.
- **Azimuth and altitude on older browsers** — Safari only added these in **18.2**. Where they are missing they are worked out from Tilt X and Tilt Y, which every implementation reports, so the two tilt modes keep working and the numbers stay meaningful.
- **A second touch while drawing** — resting a palm on a display tablet, or a second finger, does not join or end the stroke in progress. The stroke belongs to the pointer that started it, and so do the readouts until it ends.

## Privacy

The Lab:

- Does **not** collect any data about you or your computer.
- Does **not** use cookies.
- Does **not** track your behavior.
- Does **not** record what you draw.

It is a static web page that runs entirely in your browser.

## Source code

The app is open source — review, fork, and modify freely:
<https://github.com/TheSevenPens/WebStrokeExperimentLab>

## Further reading

- MDN Pointer Events API: <https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events>
- Original docs page: <https://docs.sevenpens.com/drawtab/resources/sevenpens-tablet-tester>
