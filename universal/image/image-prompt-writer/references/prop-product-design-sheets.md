# Prop And Product Design Sheets

Use this reference for prop concepts, product renders, product listing images, packaging-free presentations, mechanism studies, material sheets, usage states, and design callouts.

## Product And Prop Slots

- Object identity: product or prop type, function, count, geometry, silhouette, size, and required visible faces.
- Use case: e-commerce hero, catalog image, concept sheet, pitch deck, instruction diagram, prop breakdown, packaging concept, or material study.
- Composition: single hero object, orthographic views, exploded view, scale reference, detail callouts, usage state, in-hand view, or environment context.
- Materials: plastic, metal, glass, ceramic, wood, fabric, rubber, paper, stone, organic material, finish, transparency, reflectivity, edge wear, texture, and manufacturing detail.
- Lighting: clean studio light, softbox, rim light, practical light, outdoor light, specular control, shadow grounding, and color fidelity.
- Constraints: exact count, no geometry drift, no extra parts, no unauthorized brand marks, no fake certifications, no unreadable safety text.

## Sheet Controls

For a product or prop design sheet, include front/back/side or 3/4 views, material callouts, scale reference, part labels or numbered markers, detail closeups, color/material swatches, and usage states. Keep callouts outside the object silhouette and away from critical edges.

For a product presentation image, prioritize product visibility, accurate shape, material finish, clean background, correct scale, readable silhouette, and enough negative space for later layout.

## Text And Branding

Use controlled typography only when the user supplies authorized copy, labels, or package text. Otherwise use blank label panels, generic icons, or numbered markers. Avoid random logos, fake brands, fake seals, fake medical/legal/financial claims, and tiny unreadable product text.

## Prompt Skeletons

```text
{product hero image}: {product type} for {use case}, exact geometry and required visible faces, clean composition, grounded shadow, material finish, controlled reflections, scale cue if useful, optional negative space for later copy, no random logos, no fake labels, no geometry drift.
```

```text
{prop/product design sheet}: hero 3/4 view, front/back/side views, exploded or cutaway detail if useful, material callouts, scale reference, usage states, color swatches, numbered markers or short labels in margins, clean background, no labels over critical silhouette.
```

```text
Preserve:
<product geometry, logo/copy if authorized, material, camera angle, color, packaging shape, and layout>

Edit:
<background, lighting, usage state, material variant, angle, or detail callout>

Avoid:
<warped geometry, changed count, fake branding, random text, extra parts, unreadable safety claims, reflection artifacts>
```
