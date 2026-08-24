# Platform Adaptation

Use this optional generic reference only when the user asks for platform adaptation, a particular field format, length budget, or syntax style. The core prompt unchanged principle comes first: keep subject, composition, editing boundaries, visible text plan, and constraints stable while adapting wrapper format.

## Generic Adaptation Steps

1. Preserve the platform-neutral prompt as the source of truth.
2. Convert only the outer format: positive prompt, avoid/negative field, aspect ratio, seed/style/settings notes, or JSON-like fields if requested.
3. Keep editing tasks separated into Preserve / Edit / Avoid even if the platform uses a single prompt box.
4. Move long constraints into the platform's negative/avoid field only when that format supports it.
5. Do not add model names, provider names, private tool paths, or local workflow commands.

## Output Patterns

```text
Core prompt:
<platform-neutral production prompt>

Platform adaptation:
<requested syntax or field mapping>

Avoid / negative:
<compact risks if the platform supports this field>
```

## When Not To Adapt

Do not invent platform syntax when the user did not ask for it. Do not weaken the visible text policy, reference fidelity, exact panel counts, or edit boundaries to fit a platform shortcut. If a platform may struggle with exact text, use blank label boxes, numbered markers, or external metadata.
