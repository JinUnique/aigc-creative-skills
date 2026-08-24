# Creative Skills

Reusable agent skills for writing production-ready image and video generation prompts.

## Included skills

- `image-prompt-writer` — still-image generation and editing prompts, including products, posters, portraits, design sheets, storyboards, UI, infographics, diagrams, and cinematic frames.
- `video-prompt-writer` — single-clip and multi-segment video prompts with timelines, cinematography, sound, continuity, and platform-capability checks.

## Layout

```text
universal/
├── image/image-prompt-writer/
└── video/video-prompt-writer/
```

Each skill is self-contained: start with its `SKILL.md`, then load only the referenced files needed for the task.

## Install

Copy the desired skill directory into the skills directory used by your agent runtime. Keep the entire directory together so relative links under `references/` continue to work.

Example:

```bash
cp -R universal/image/image-prompt-writer ~/.hermes/skills/
cp -R universal/video/video-prompt-writer ~/.hermes/skills/
```

Some `related_skills` metadata entries refer to optional companion skills. The two skills published here remain usable on their own.

## Scope

This repository contains reusable capability definitions and reference material only. It does not include private production state, provider credentials, generated outputs, or personal workflow data.

This initial public release uses a clean Git history intentionally; private development history is not part of the distribution.

## License

MIT — see [LICENSE](LICENSE).
