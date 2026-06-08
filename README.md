# shark-skills

Official Claude skills from [SharkIndicators](https://sharkindicators.com) for working with the SharkSignals platform.

A Claude **skill** is a small package that teaches Claude how to perform a specific task. Skills are loaded by Claude Code (and other Claude clients that support them) and trigger automatically when a user's request matches what the skill is for.

## Available skills

| Skill | Version | Description |
|---|---|---|
| [`shark-signal`](shark-signal/) | 0.1.0 | Send a JSON POST to a SharkSignals signal endpoint (TradingView-style webhook) for testing the signal pipeline. |

## Installing a skill

Each skill ships as a `.skill` file (a packaged zip archive) inside its folder.

**Option A — drag-and-drop in Claude Code:** Settings → Skills → Install from file → pick the `.skill`.

**Option B — manual install:** Unzip the `.skill` archive (it's just a zip with a `.skill` extension) into your Claude skills directory:

- Windows: `%USERPROFILE%\.claude\skills\`
- macOS / Linux: `~/.claude/skills/`

After installing, restart Claude Code (or invoke `/reload-skills`) so the new skill shows up in the loaded list.

## Versioning

Skills use [semver](https://semver.org/): `MAJOR.MINOR.PATCH`. Each skill's version lives in two places:

- `metadata.version` in the skill's `SKILL.md` frontmatter — the canonical version.
- A `CHANGELOG.md` next to `SKILL.md` documenting what changed in each release.

Pre-1.0 (`0.x.y`) means the skill is still stabilizing and breaking changes may land in any minor release.

## Issues and feedback

Bug reports, feature requests, and questions: [open an issue](https://github.com/sharkindicators/shark-skills/issues).

When reporting a bug with a skill, please include the version line the skill announces. For `shark-signal`, that's the `shark-signal v0.X.Y` message it sends right before any prompts — paste that into the issue so we know exactly which version you're running.
