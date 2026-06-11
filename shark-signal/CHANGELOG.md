# Changelog

All notable changes to the `shark-signal` skill. Versions follow [semver](https://semver.org/): MAJOR.MINOR.PATCH. Pre-1.0 means breaking changes are still on the table.

What counts as a breaking change here:

- A trigger phrase removed from the `description` (customers learned to say it; it stops working)
- A change to the user-facing flow (questions removed, reordered, or whose accepted values change)
- A change to the JSON body the skill POSTs (field renamed, removed, or type-changed) that the SharkSignals server no longer accepts from old skill versions

## 0.2.0 — 2026-06-11

New step reporting POST outcome to the user. No change to the JSON body or to what triggers the skill.

- Added step 4: read the HTTP status from curl and tell the user the outcome. 2xx → "Your SharkSignal was successfully sent. Check https://app.sharksignals.com for the status of your trade." Anything else → the status code plus the structured envelope fields (`error`, `detail`, `errorId`) from the response body, or the raw body if unstructured, or the curl-level error if no response at all.
- Step 3's curl command (both POSIX and PowerShell variants) now appends `-w "\nHTTP %{http_code}\n"` so the status code is unambiguously visible as the last line of output for step 4 to read.

## 0.1.1 — 2026-06-08

Documentation clarification of Step 3. No change to the JSON body, the user flow, or what triggers the skill — old SKILL.md instructions remained valid against the same server contract.

- Split the curl example into POSIX (bash/zsh) and PowerShell variants. PowerShell needs `curl.exe` because `curl` is an alias for `Invoke-WebRequest` and rejects POSIX-style flags.
- Switch the bash example from `-d` to `--data-raw` for fully literal body handling (no special treatment of `@` or stripped newlines).
- Add an explicit warning that JSON property names and string values must remain double-quoted (`{action:buy,...}` is not valid JSON).

## 0.1.0 — 2026-06-04

Initial release.

- Prompts user for URL, action, quantity, symbol, and asset class.
- Reuses URL / quantity / symbol from prior session invocations when available.
- POSTs `{action, quantity, symbol, assetClass}` as JSON strings to the supplied signal URL.
- Requires TLS verification — `-k` / `--insecure` is explicitly forbidden.
- Announces `shark-signal v0.1.0` to the user on every invocation for support diagnostics.
