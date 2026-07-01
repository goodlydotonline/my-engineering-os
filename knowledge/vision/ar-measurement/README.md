# AR Measurement

## Purpose
Knowledge and experience around Augmented Reality based distance measurement, especially using ARCore on Android.

## What Belongs Here
- ARCore concepts, constraints, and device capability analysis.
- Measurement algorithm evolution and decision records.
- Best practices for user interaction, accuracy, and validation.
- Comparisons of different AR measurement approaches.

## What Does NOT Belong Here
- General computer vision theory not tied to AR measurement.
- FireCopilot project-specific active work (use `projects/`).
- Raw experiment logs (use `projects/` or `inbox/`).

## Lifecycle
1. Capture raw measurements and observations in `inbox/`.
2. Distill verified approaches and concepts here.
3. Update when ARCore versions, device support, or project findings change.

## Entries
- `arcore-concepts.md` — ARCore, Depth API, and Android AR hardware support explained.
- `approaches-evolution.md` — Six-stage evolution of FireCopilot distance measurement, from pixel estimation to plane-locked ray projection.
