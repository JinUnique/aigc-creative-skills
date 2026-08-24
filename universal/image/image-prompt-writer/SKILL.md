---
name: image-prompt-writer
description: "Use for still-image prompt writing, rewriting, or editing briefs: generation, edits, reference preservation, product/prop, poster/cover/key art, portraits, character/environment sheets, storyboard panels, UI/infographic/diagram prompts, and photo/cine frames."
version: 0.4.0
metadata:
  hermes:
    tags: [image, prompt, generation, editing, product, poster, portrait, character, storyboard, keyframe, ui, annotation]
    related_skills: [video-prompt-writer, story_shot_planner]
---

# Image Prompt Writer

Turn a still-image generation, editing, or prompt-writing request into a production-ready prompt that is generic, platform-neutral, and usable without private workflow context.

## Trigger

Use this skill for static image tasks even when the user does not say "prompt": generate an image, edit a reference image, inpaint/outpaint, replace background or objects, preserve identity/product/reference details, create product or prop visuals, posters/covers/key art, portraits/headshots, character design sheets/model sheets/turnarounds, environment or production-design sheets, storyboard panels, UI mockups, infographics, diagrams, photography, cinematic frames, keyframes, and action stills. For continuous motion, duration, timeline, or video generation prompts, route to `video-prompt-writer`.

## Inputs

Collect or infer these slots:

- task, subject, use case, composition, action/pose, environment, camera/medium, lighting/color, material/texture, constraints.
- format: aspect ratio, size, crop, panel count, sheet type, reference images, and output language.
- editing boundaries: what to preserve, what to edit, what to avoid.
- visible-text intent: text-free, placeholder labels, readable labels/copy, annotations, legends, UI text, or typography.

Use the user's requested language for production prompts. If the user does not specify a language, match the user's language.

## Workflow

1. Classify the image job: new generation, edit, variation, reference lock, product/prop, poster/key art, portrait/photo frame, character sheet, environment sheet, storyboard, UI/infographic/diagram, or explicitly text-free image.
2. Fill concrete visual slots before style words: subject, use case, composition, action/pose, environment, camera/medium, lighting/color, material/texture, and constraints. Use `references/core-prompt-structure.md`.
3. For edits, separate `Preserve / Edit / Avoid` before writing the final prompt. Preserve identity, layout, product geometry, materials, viewpoint, and lighting unless the user changes them.
4. Treat visible text as a design element. Avoid random/unrequested text, filler brands, watermarks, and tiny unreadable prose; allow requested or useful labels, legends, panel numbers, UI copy, callouts, annotations, symbols, and diegetic prop text. Use `references/text-labels-annotations-policy.md`.
5. Route domain depth to references:
   - Character sheets/model sheets/turnarounds: `references/character-design-sheets.md`.
   - Environment concept or production-design sheets: `references/environment-concept-sheets.md`.
   - Prop and product design/presentation sheets: `references/prop-product-design-sheets.md`.
   - Storyboard sheets or panel sequences: `references/storyboard-panels.md`.
   - Posters, covers, and key art: `references/posters-covers-key-art.md`.
   - UI, infographic, and diagram prompts: `references/ui-infographic-diagram-prompts.md`.
   - Photography, portraits, cinematic frames, keyframes, action stills, and explicit text-free images: `references/photography-cinematic-frames.md`.
   - Platform-specific field formatting only when requested: `references/platform-adaptation.md`.
6. Quality-check for subject visibility, composition, reference fidelity, text intent, brand/IP boundaries, and downstream usability before final output.

## Output Contract

Default generation output:

```text
Task type: <image job>
Assumptions: <only if needed>
Prompt:
<production-ready prompt in the selected language>
Avoid:
<compact task-specific risks>
Continuity anchors:
<only when relevant>
```

Editing output:

```text
Preserve:
<unchanged identity, geometry, layout, materials, camera, lighting, or reference traits>

Edit:
<specific changes>

Avoid:
<drift, artifacts, random text/brands, wrong count/layout, and task-specific failures>
```

For sheets, boards, UI, diagrams, and annotated images, include a short `Visible text / annotation plan` when text, labels, legends, panel numbers, callouts, or blank label boxes matter.

## Quality Gate

- The requested subject, product, person, prop, panel, or interface is visible, inspectable, and not hidden by blur, effects, cropping, hands, reflections, or decorative atmosphere.
- The prompt specifies composition: aspect ratio or layout, crop, viewpoint, foreground/midground/background, focal hierarchy, and negative space when useful.
- Reference/edit tasks have explicit Preserve / Edit / Avoid boundaries and do not invent unauthorized identity, brand, logo, IP, or claims.
- Visible text is intentional: readable when requested or useful, placeholder/blank when exact text reliability matters, and absent when the user asks for an explicitly text-free image.
- Character sheets, environment sheets, storyboards, product/prop sheets, UI/infographic/diagram prompts, posters/covers/key art, and ordinary cinematic/photo frames each follow their relevant production conventions.
- No model, provider, local tool, personal workflow, private record system, or temporary project naming is required to understand or use the skill.

## References

Core: `references/core-prompt-structure.md`, `references/text-labels-annotations-policy.md`, `references/platform-adaptation.md`.

Specialized: `references/character-design-sheets.md`, `references/environment-concept-sheets.md`, `references/prop-product-design-sheets.md`, `references/storyboard-panels.md`, `references/posters-covers-key-art.md`, `references/ui-infographic-diagram-prompts.md`, `references/photography-cinematic-frames.md`.
