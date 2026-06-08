---
name: shark-signal
description: Sends a JSON POST via curl to a SharkSignals signal endpoint (`/signal/{route-id}/{signal-id}` URL used by TradingView-style webhooks). ALWAYS use this skill when the user says any variation of "send shark signal", "send sharksignal", "post sharksignal", "post shark signal", or asks to send/post a test signal, hit the `/signal/...` endpoint, trigger a SharkSignals webhook, or test the signal pipeline — even if they don't explicitly mention curl or this skill. Prompts the user via menus for action, quantity, symbol, and assetClass. Reuses the URL, Quantity and Symbol from earlier in the session if they were already used; otherwise asks for them. Do NOT use this for unrelated curl/HTTP requests that don't target a SharkSignals signal endpoint.
metadata:
  version: 0.1.1
---

# Send a SharkSignals signal

## Always announce the version first

Before any prompts, scans, or curl calls, send the user a single line:

> `shark-signal v0.1.1` — preparing to send a SharkSignals signal.

This makes support bug reports trivially diagnosable — the customer can paste the line back. Keep it to one line. When the version in the frontmatter changes, update the literal string above to match.

Sends a `POST <url>` with this JSON body:

```json
{"action": "<action>", "quantity": "<qty>", "symbol": "<sym>", "assetClass": "<class>"}
```

The signal ID lives in the URL path and the endpoint reads it from there — don't add it to the body.

## Steps

### 1. Scan the current conversation for prior uses of this skill

Before prompting the user, look back through the **current conversation only** for earlier `curl ... /signal/...` calls you sent via this skill. From those, collect the distinct values that were used for:

- **URL** — the full `https://.../signal/<route-id>/<signal-id>` URL
- **Quantity** — the value sent in the JSON body
- **Symbol** — the value sent in the JSON body

Keep these in memory for step 2. Most-recent-first if there are several. Do not pull values from memory files, prior chats, or training-time guesses — only this conversation.

If this is the first run in the session, all three lists are empty and that's fine.

### 2. Ask the user for the five inputs, in order: URL, Action, Quantity, Symbol, Asset Class

Because `AskUserQuestion` is capped at 4 questions per call and the URL needs special handling, split the prompts into **two calls**:

**Call A — URL only.**

- If you found **prior URL(s)** in step 1: use `AskUserQuestion` with one question (header: "URL"). Options = each prior URL (most recent first, up to 3) plus a final option `Use a different URL`. If the user picks `Use a different URL` (or the auto-provided "Other"), follow up by asking for the new URL as plain free-text.
- If you found **no prior URL**: don't use `AskUserQuestion` — just ask the user for the URL as plain free-text. A menu with no real options is noise.

**Call B — Action, Quantity, Symbol, Asset Class (in that order, single `AskUserQuestion` with four questions).**

- **Action** (header: "Action") — options: `flatten`, `buy`, `sell`, `close_long`. The 5th valid action `close_short` rides the auto-provided "Other".
- **Quantity** (header: "Quantity") — options: prior quantities from step 1 first (most recent first), then fill up to 4 total with common defaults (`1`, `5`, `10`, `100`) that aren't already in the list. "Other" handles anything else.
- **Symbol** (header: "Symbol") — options: prior symbols from step 1 first (most recent first), then fill up to 4 total with common defaults (`AAPL`,`ES`, `EURUSD`, `BTCUSD`) that aren't already in the list. "Other" handles anything else.
- **Asset class** (header: "Asset class") — options: `stock`, `futures`, `forex`, `crypto`.

If the user dismisses either call without answering, stop — don't invent values or substitute defaults.

### 3. Send the POST

Choose the command form for the active shell. The request body must reach the server as valid JSON with double-quoted property names and string values.

**Bash, zsh, or other POSIX shells: use curl with single-quoted JSON.**

```bash
curl -X POST "<url>" \
  -H "Content-Type: application/json" \
  --data-raw '{"action":"<action>","quantity":"<qty>","symbol":"<sym>","assetClass":"<class>"}'
```

**PowerShell on Windows: use `curl.exe`, not `curl`.** In PowerShell, `curl` may resolve to an alias instead of real curl. Put the JSON in a variable first, then pass that variable to `curl.exe`.

```powershell
$body = '{"action":"<action>","quantity":"<qty>","symbol":"<sym>","assetClass":"<class>"}'

curl.exe -X POST "<url>" `
  -H "Content-Type: application/json" `
  --data-raw $body
```

Never send JSON that looks like `{action:buy,...}`. That is not valid JSON; property names and string values must keep their double quotes.

Do **not** add `-k` / `--insecure`. Calls must validate the server's TLS certificate. 

All four field values are sent as **JSON strings**, including quantity — that matches the payload shape the SharkSignals endpoint expects.

Show the response back to the user. If curl hangs past its default timeout with no response body, surface that the SharkSignals server may have accepted the request but be slow to reply; the user might want to check logs.

## Why these choices

- **Reuse-from-session, not from disk.** The signal URL contains a route ID and signal ID that change per signal config and per environment. Persisting it across sessions could cause the skill to silently POST to a stale or wrong endpoint. Re-asking once per session is cheap and safe.
- **Prior values as menu options, not auto-filled.** Showing the previous quantity/symbol as one option among several is faster than retyping but doesn't lock the user in — they can still pick a different value or type a new one via "Other". Auto-defaulting would surprise users running a different test than last time.
- **Two `AskUserQuestion` calls, not one.** The 4-question cap on `AskUserQuestion` makes a single batch impossible for five inputs. Splitting URL out is also natural because URL needs free-text mode when no prior URL exists.
- **No `-k`, even for local dev.** Skipping TLS verification hides real misconfigurations and trains the habit of ignoring cert errors. If the local cert isn't trusted, fix the trust store; don't bypass verification at the call site.
