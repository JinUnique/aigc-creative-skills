# Core Prompt Structure

Use this reference for every still-image prompt before selecting a specialized sheet or genre reference. It keeps the prompt standalone, generic, and independent of any model or platform.

## Core Slots

Fill these slots in this order when they are relevant: task, subject, use case, composition, action/pose, environment, camera/medium, lighting/color, material/texture, constraints.

- task: generation, edit, variation, inpaint, outpaint, reference lock, design sheet, storyboard, UI/diagram, poster, portrait, product, prop, or explicit text-free image.
- subject: who or what must be visible, identity anchors, product geometry, scale, count, and relationship between subjects.
- use case: avatar, product listing, concept sheet, pitch deck, storyboard, cover, poster, diagram, annotation sheet, reference image, or background plate.
- composition: aspect ratio, crop, viewpoint, focal hierarchy, foreground/midground/background, negative space, margins, gutters, or panel layout.
- action/pose: static pose, gesture, expression, contact point, force direction, usage state, interaction, or one clear action beat.
- environment: location type, spatial layout, surface, props, weather/time, scale cues, and what must stay uncluttered.
- camera/medium: photograph, illustration, concept art, diagram, orthographic sheet, cinematic frame, lens, angle, depth of field, rendering medium.
- lighting/color: motivated source, direction, contrast, palette, value hierarchy, color script, and color restrictions.
- material/texture: fabric, metal, glass, plastic, paper, skin, terrain, product finish, wear, reflection, transparency, and tactile detail.
- constraints: visible text policy, brand/IP limits, safety, no random filler, anatomy/product/layout risks, exact counts, and reference fidelity.

## Task Routing

- Use character-design sheets when the output must make a reusable character, not just a portrait.
- Use environment sheets when spatial layout, paths, scale, entrances, zones, materials, or production-design readability matter.
- Use prop/product sheets when geometry, use state, scale, material finish, manufacturing detail, or selling clarity matters.
- Use storyboard panels when multiple still panels communicate shot order, continuity, action beats, or animatic planning.
- Use UI/infographic/diagram prompts when information structure, labels, axes, legends, interface copy, or annotation logic matters.
- Use photography/cinematic frames for ordinary single frame prompts, portraits, landscapes, action stills, and text-free images.

## Editing Boundaries

For editing/reference tasks, write boundaries before prose:

```text
Preserve:
<identity, product geometry, layout, pose, environment, camera, lighting, material, text, or style traits that must remain>

Edit:
<specific change, location of change, intensity, replacement, added element, removal, or extension>

Avoid:
<identity drift, geometry drift, changed layout, random text, unauthorized brands, extra subjects, artifacts, wrong count, or over-editing>
```

Only preserve what the user needs. Do not lock irrelevant details that would fight the requested edit.

## Assembly Pattern

```text
{task} for {use case}: {subject and required anchors}. Composition: {aspect ratio/layout/crop/viewpoint/focal hierarchy}. Action/pose: {one readable beat or neutral design pose}. Environment: {space, surfaces, scale cues}. Camera/medium: {photographic, illustrated, concept, diagram, sheet, or board language}. Lighting/color: {source, contrast, palette}. Material/texture: {specific surfaces}. Visible text: {text-free / placeholder / readable text plan}. Constraints: {avoid block}.
```

## Prompt Density

- Prefer one clear image goal over multiple competing scenes.
- Use concrete nouns and production controls instead of generic quality adjectives.
- Keep exact counts explicit for panels, views, products, callouts, and labels.
- Keep text requirements short and high-value.
- Put risky constraints in `Avoid`, not scattered through the prompt.

## Self-Check

The prompt is ready when another image system or human artist can identify the subject, use case, layout, camera/medium, lighting/color, material/texture, visible text plan, and constraints without knowing any private workflow.
