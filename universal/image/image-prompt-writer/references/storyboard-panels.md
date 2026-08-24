# Storyboard Panels

Use this reference for a static storyboard sheet, shot board, panel sequence, animatic planning still, comic-like production board, or single storyboard key panel.

## Core Principle

A storyboard prompt is a still-image prompt for shot planning. It should specify panel count, layout, order, shot role, continuity, and note strategy with no provider dependency.

## Required Controls

- Exact panel count: state the requested count or choose 6 panels for an unspecified full board.
- Layout: 3x2 for 6 panels, 4x2 or 2x4 for 8 panels, 4x3 or 3x4 for 12 panels, unless the user requests another layout.
- Panel numbers: use large simple `PANEL 01`, `PANEL 02` labels in margins/headers, or blank label boxes plus row-major order when exact text reliability matters.
- Per-panel beat: one visual beat per panel with shot size, camera angle, action/blocking, composition focus, mood, and transition when useful.
- Camera movement cues: allowed as compact storyboard planning notes such as pan, tilt, push-in, track, zoom, handheld, or static; do not write durations or continuous video directions.
- Continuity: subject identity, wardrobe, prop state, environment geography, screen direction, eyelines, light direction, and color palette.
- Note bands: keep dialogue, VO, SFX, timing notes, or action notes short and outside image action areas.

## Label Policy

Panel labels, panel numbers, note bands, arrows, blank label boxes, and numbered markers are useful. Keep them in margins, headers, gutters, or note bands. Do not cover faces, hands, props, staging, action contact points, or key silhouettes. Use short labels; avoid long generated prose.

## Prompt Skeletons

```text
{static storyboard sheet}: exact {6/8/12} panels in {layout} uniform grid, clean gutters, row-major order, panel numbers using large simple PANEL 01 labels or blank label boxes, sequence purpose: {setup/change/result}, each panel has one visual beat with shot size, camera angle, Camera movement cues, action/blocking, composition focus, mood, and transition, consistent identity/wardrobe/prop state/environment/screen direction/eyelines/light direction, no extra panels, no merged panels, no timecodes.
```

```text
{single storyboard key panel}: one decisive beat, shot role, subject/object relation, clear silhouette and contact point if action, environment anchor, screen direction or eyeline anchor if part of a sequence, static framing, optional margin label or blank note band, no long captions.
```

```text
Preserve:
<reference panel style, subject identity, wardrobe, prop state, layout, lighting, screen direction, and panel order>

Edit:
<add/remove/reorder panel, clarify beat, fix eyeline, simplify density, add note bands, change panel count>

Avoid:
<wrong panel count, extra panels, merged panels, inconsistent identity, broken screen direction, hidden action, long text, timecodes, continuous video directions>
```
