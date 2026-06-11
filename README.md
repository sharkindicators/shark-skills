# shark-skills

Official AI skills from [SharkIndicators](https://sharkindicators.com) for working with the SharkSignals platform.

An AI **skill** is a small package that teaches your AI agent how to perform a specific task. Skills are loaded by Claude or Codex, and trigger automatically when a user's request matches what the skill is for.

## Available skills

| Skill | Version | Download | Description |
|---|---|---|---|
| [`shark-signal`](shark-signal/) | 0.2.0 | [shark-signal.skill](shark-signal.skill) | Send a JSON POST to a SharkSignals signal endpoint (TradingView-style webhook) for testing the signal pipeline. |

Each skill is laid out as:

- A folder (e.g. `shark-signal/`) containing the source — `SKILL.md` and `CHANGELOG.md`.
- A `.skill` file at the repo root (e.g. `shark-signal.skill`) — the Claude installable, packaged archive.

## Installing a skill

The right install path depends on which agent you're using. 

### Claude or Codex

The easiest path is to ask your AI agent itself to install from this repo, e.g.:

> Install the shark-signal skill from https://github.com/sharkindicators/shark-skills

Or:

> Install all skills from https://github.com/sharkindicators/shark-skills

## Security

Only install skills from sources you trust — including ours. Skills can execute code and access data on whichever surface they're loaded on. Read the `SKILL.md` before installing anything, and prefer cloning this repo over downloading a `.skill` archive from an unknown mirror.

## Versioning

Skills use [semver](https://semver.org/): `MAJOR.MINOR.PATCH`. Each skill's version lives in two places:

- `metadata.version` in the skill's `SKILL.md` frontmatter — the canonical version.
- A `CHANGELOG.md` next to `SKILL.md` documenting what changed in each release.

Pre-1.0 (`0.x.y`) means the skill is still stabilizing and breaking changes may land in any minor release.

## License

This repo is licensed under [Apache License 2.0](LICENSE). Skills here are provided "as is" without warranty — see [LICENSE](LICENSE) and [NOTICE](NOTICE) for the full terms. This is especially relevant when a skill helps you send orders or signals to live trading systems: validate any generated request against a test environment before pointing it at production.

## Support

For bug reports, feature requests, and questions, please email [support@sharkindicators.com](mailto:support@sharkindicators.com).

When reporting a bug with a skill, please include the version line the skill announces. For `shark-signal`, that's the `shark-signal v0.X.Y` message it sends right before any prompts — paste that into your email so we know exactly which version you're running.
