---
name: shark-signal
description: Sends a JSON POST via curl to a SharkSignals signal endpoint (`/signal/{route-id}/{signal-id}` URL used by TradingView-style webhooks). ALWAYS use this skill when the user says any variation of "send shark signal", "send sharksignal", "post sharksignal", "post shark signal", or asks to send/post a test signal, hit the `/signal/...` endpoint, trigger a SharkSignals webhook, or test the signal pipeline — even if they don't explicitly mention curl or this skill. Prompts the user via menus for action, quantity, symbol, and assetClass. Reuses the URL, Quantity and Symbol from earlier in the session if they were already used; otherwise asks for them. Do NOT use this for unrelated curl/HTTP requests that don't target a SharkSignals signal endpoint.
metadata:
  version: 1.0.1
---

# Send a SharkSignals signal

## Always announce the version first

Before any prompts, scans, or curl calls, send the user a single line:

> `shark-signal v1.0.1` — preparing to send a SharkSignals signal.

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

**Prefer a Unix-like shell (bash or zsh)** — including Git Bash or WSL on Windows. Fall back to PowerShell when no Unix-like shell is available. Either way, the JSON body must reach the server with its double quotes intact; how you guarantee that differs by shell, because some shells corrupt a quoted JSON string when it's passed as a command-line argument. That corruption risk is exactly why the Unix-like path is preferred — its single-quoting is the simplest thing that can't go wrong.

**Preferred — Unix-like shells (bash, zsh), including Git Bash / WSL on Windows: `curl` with single-quoted JSON.** Single quotes pass the body through literally, so the quotes survive.

```bash
curl -X POST "<url>" \
  -H "Content-Type: application/json" \
  --data-raw '{"action":"<action>","quantity":"<qty>","symbol":"<sym>","assetClass":"<class>"}' \
  -w "\nHTTP %{http_code}\n"
```

**Fallback — Windows PowerShell (only when no Unix-like shell is available): use `Invoke-WebRequest`, not curl.** PowerShell strips the double quotes off a JSON string when it hands it to a native exe like `curl.exe`, so the body arrives as `{action:buy,...}` — invalid JSON the server rejects with a 500. `Invoke-WebRequest` takes the body as a cmdlet parameter (`-Body`), which is in-process .NET binding and never crosses that argument-mangling layer, so the quotes survive.

```powershell
$body = '{"action":"<action>","quantity":"<qty>","symbol":"<sym>","assetClass":"<class>"}'
try {
    $r = Invoke-WebRequest -Uri "<url>" -Method Post `
        -ContentType 'application/json' -Body $body -UseBasicParsing
    "HTTP $([int]$r.StatusCode)"
    $r.Content
} catch {
    $resp = $_.Exception.Response
    if ($resp) {
        "HTTP $([int]$resp.StatusCode)"
        (New-Object System.IO.StreamReader($resp.GetResponseStream())).ReadToEnd()
    } else {
        "ERROR: $($_.Exception.Message)"
    }
}
```

Do **not** disable TLS verification (curl `-k` / `--insecure`, or PowerShell `-SkipCertificateCheck`). Calls must validate the server's certificate.

All four field values are sent as **JSON strings**, including quantity — that matches the payload shape the SharkSignals endpoint expects.

Both forms end by emitting a `HTTP <code>` line — curl via `-w`, PowerShell via the `try/catch` above — so step 4 has a uniform status line to read regardless of shell. Show that output (including the trailing line) to the user. If the call hangs with no output, surface that the SharkSignals server may have accepted the request but be slow to reply; the user might want to check logs.

### 4. Report success or failure to the user

Read the `HTTP <code>` line at the bottom of the command's output and tell the user the outcome:

- **2xx (200–299)** — send the user this exact line:

  > Your SharkSignal was successfully sent. Check https://app.sharksignals.com for the status of your trade.

- **Anything else** — tell the user the status code and the relevant details from the response body. SharkSignals returns a structured envelope on failure with `error`, `detail`, and `errorId` fields; quote those if present. Format:

  > The SharkSignals server returned HTTP `<code>`. error: "...", detail: "...", errorId: "...".

  If the response body has no recognizable structure, paste the raw response instead. If the call failed before getting a response at all (connection refused, DNS failure, TLS handshake error, etc. — no `HTTP <code>` line, or PowerShell printed an `ERROR:` line), report that directly without inventing a status code.

## Why these choices

- **Reuse-from-session, not from disk.** The signal URL contains a route ID and signal ID that change per signal config and per environment. Persisting it across sessions could cause the skill to silently POST to a stale or wrong endpoint. Re-asking once per session is cheap and safe.
- **Prior values as menu options, not auto-filled.** Showing the previous quantity/symbol as one option among several is faster than retyping but doesn't lock the user in — they can still pick a different value or type a new one via "Other". Auto-defaulting would surprise users running a different test than last time.
- **Two `AskUserQuestion` calls, not one.** The 4-question cap on `AskUserQuestion` makes a single batch impossible for five inputs. Splitting URL out is also natural because URL needs free-text mode when no prior URL exists.
- **No `-k`, even for local dev.** Skipping TLS verification hides real misconfigurations and trains the habit of ignoring cert errors. If the local cert isn't trusted, fix the trust store; don't bypass verification at the call site.
- **Each shell uses a body channel it can't corrupt.** Windows PowerShell strips embedded double quotes when it passes a string to a native exe, turning valid JSON into `{action:buy,...}` — which the server rejects. The fix can't be a prose reminder to "quote your JSON," because the model *does* produce correct JSON; the shell mangles it afterward. So each path keeps the JSON out of the argument list: POSIX single-quoting passes it literally, and PowerShell's `Invoke-WebRequest -Body` binds it in-process. Pick the form for the shell rather than trusting any shell to preserve a quoted argument.
