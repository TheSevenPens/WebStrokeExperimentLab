# WebStrokeExperimentLab

A test bed for how pen strokes are drawn on the web: **what the pen reports, how much of it a stroke uses, and what each stage of drawing does to it.**

Every stage is a control, so any two of them can be compared on the same hand movement. That is the whole purpose — not to draw well, but to make the difference between two ways of drawing visible.

**This is not the tablet tester.** [WebTabletTesterBasic](https://github.com/TheSevenPens/WebTabletTesterBasic) answers one question — *does my tablet work* — and is deliberately simple: no settings, one good way of drawing, a handful of readouts. It is what to send someone whose pen has stopped working. This repository is where alternatives get tried, and it is free to be as complicated as an experiment needs.

## Live

<https://thesevenpens.github.io/WebStrokeExperimentLab/>

## What it measures

| | |
|---|---|
| **Points/s** | two numbers: what the pen reports, and how many of those the stroke is built from |
| **Precision** | which grid positions land on — `CSS pixels`, `screen pixels`, or `sub-pixel` |
| **X, Y** | pointer position, always to two decimals |

## What it varies

| | |
|---|---|
| **Size** | which reading decides how big the brush is: `pressure` / `tilt altitude` / `fixed` |
| **Rotation** | which reading decides which way it points: `none` / `tilt azimuth` / `twist`. Anything but none makes the brush an oval |
| **Stroke** | `Stepped width` / `Taper (straight)` / `Taper (curved)`, which is the default |
| **Edge** | `Hard` / `Soft` |
| **Smoothing** | `Off` / `Light` / `Heavy` — streamlining, at perfect-freehand's and atrament's defaults |
| **Use all pen points** | draw from every reported position rather than one per screen refresh. On by default; untick it for what an ordinary web page draws |

Size and rotation are asked separately because they are separate facts about the pen. They used to be bundled into named modes — *Pressure to Size*, *Twist to Brush rotation* — which made every combination nobody had thought to name unavailable.

## What has been established here

- **Pen positions reach a web page snapped to whole screen pixels**, and display scaling disguises that as sub-pixel precision. Measured on Windows at 175% scaling: 359 consecutive samples, every one within 0.0001 of a whole screen pixel.
- **That quantisation is why slow strokes look rough** and fast ones do not. Slow strokes land samples a pixel or two apart, so snapping moves each one by a large fraction of the step and the direction between them lurches.
- **Smoothing fixes it by reconstructing between grid points**, not by hiding a shaky hand. Mean distance from a true straight line on a snapped diagonal: 0.141 px off, 0.082 light, 0.028 heavy.
- **Ruled out by experiment, each having been suspected first:** pressure noise, compositing of overlapping fills, capsules versus round dabs, brush size, and the input path — Krita through Windows Ink, the same API, is still clean.

Details, including the traps in measuring any of it, are in [USERMANUAL.md](./USERMANUAL.md).

## Running locally

Open `index.html` in a browser. No build step, no dependencies, no webserver needed.

## Scope

The opposite of the tester's. A control earns its place here if it makes a stage of drawing comparable against another; complexity is the point rather than the cost. Anything learned that the tester should benefit from goes there as fixed behaviour with no setting attached.

Traffic runs the other way too. Both apps grew from the same source, so a defect found in one is usually in the other: the fourteen from the tester's review ([TheSevenPens/WebTabletTesterBasic#9](https://github.com/TheSevenPens/WebTabletTesterBasic/issues/9)) were all here as well, and are fixed. A fix crosses over where it is about the pen being reported honestly; a *setting* does not cross over in either direction.
