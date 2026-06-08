# Changelog

All notable changes to the `shark-signal` skill. Versions follow [semver](https://semver.org/): MAJOR.MINOR.PATCH. Pre-1.0 means breaking changes are still on the table.

What counts as a breaking change here:

- A trigger phrase removed from the `description` (customers learned to say it; it stops working)
- A change to the user-facing flow (questions removed, reordered, or whose accepted values change)
- A change to the JSON body the skill POSTs (field renamed, removed, or type-changed) that the SharkSignals server no longer accepts from old skill versions

## 0.1.0 — 2026-06-04

Initial release.

- Prompts user for URL, action, quantity, symbol, and asset class.
- Reuses URL / quantity / symbol from prior session invocations when available.
- POSTs `{action, quantity, symbol, assetClass}` as JSON strings to the supplied signal URL.
- Requires TLS verification — `-k` / `--insecure` is explicitly forbidden.
- Announces `shark-signal v0.1.0` to the user on every invocation for support diagnostics.
