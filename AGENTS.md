# treg-cli

JSON client for https://treg.to. Treg is "OpenRouter for agent tools": one token reaches about 3,800 paid data endpoints from about 110 vendors (people and company enrichment, emails, phones, SEO and SERP, TikTok, Instagram, YouTube, X, LinkedIn, Reddit, Spotify, scraping, image and video generation). Each call is billed from a prepaid team balance at the vendor's price. No vendor signup.

## Commands

```bash
treg-cli whoami                               # email, team, role
treg-cli balance [-n N]                       # balance_usd + recent ledger
treg-cli platforms                            # catalog shelves, endpoint counts, providers
treg-cli search "find a work email" [-n 10]   # endpoints for a job: id, usd, ok_rate, p50_ms, example call
treg-cli search --new                         # endpoints added in the last 30 days
treg-cli get <endpoint-id>                    # params, cost block, sibling providers, reliability
treg-cli call <endpoint-id> -q k=v [-q k=v]   # GET call
treg-cli call <endpoint-id> -d '{"k":"v"}'    # POST call (body: JSON, @file, or - for stdin)
treg-cli calls [-n 20] [--days N]             # audit log
treg-cli result <call-id>                     # fetch a stored answer again, free
treg-cli tools                                # the team's own registered tools
treg-cli request "podcast transcripts"        # ask treg to add a missing capability
treg-cli raw GET /catalog/platforms/youtube   # any other API path
```

`call` options: `--max-cost USD` (default 0.10, sent as `X-Treg-Route-Max-Cost`; treg refuses with 402 and charges nothing above it), `-H 'Name: value'`, `-X METHOD`, `--no-cache`.

## Output

```json
{"ok": true, "data": {"result": <vendor body, verbatim>, "treg": {"http_status": 200, "call_id": "...", "cost_usd": 0.00188, "served_by": "...", "cache": "hit"}}}
{"ok": false, "error": "HTTP 402 from ...", "code": "HTTP_402", "result": {...}, "treg": {...}}
```

`cost_usd` comes from the `X-Treg-Cost-Micro` header. That is the real charge. Ignore cost numbers inside the vendor body.

Routed calls (`treg.*`) answer `{output, raw, _treg}`. The row fields in `output` depend on the vendor in `served_by` (for example icypeas people rows use `firstname`, `lastname`, `lastJobTitle`, `profileUrl`). Check one row before you parse many. Routed answers are not stored, so `result <call-id>` cannot fetch them again.

## Examples

```bash
treg-cli call scrapecreators.x.v1-youtube-video-transcript -q url=https://www.youtube.com/watch?v=dQw4w9WgXcQ
treg-cli call scrapecreators.x.v1-spotify-search -q 'query=my first million'
treg-cli call treg.people.search -d '{"q":"founders of AI software consultancies in Dallas Texas"}' --max-cost 0.05
treg-cli call treg.people.email.find -d '{"full_name":"Jason Zhou","domain":"superdesign.dev"}' -H 'X-Treg-Route-Verify: true'
```

## Rules

- Pick an endpoint with `search`, then `get`: match the inputs you have first, then `ok_rate` with a real sample size, then price.
- `treg.<capability>` ids are routed. Treg tries providers in order until one answers. Use them for common jobs (`treg.web.extract`, `treg.web.search`, `treg.google.serp.organic`, `treg.people.email.find`, `treg.people.search`).
- On 429, 5xx, or a timeout, try the next provider for the same job. Do not retry a 4xx on another provider. It is usually your parameters.
- A found email or phone is a guess until verified. Run `treg.people.email.verify` before outreach. Cold outreach still needs Christian's approval of the message (`~/AGENTS.md`).
- Top-ups move money. Confirm with Christian first. Out of balance returns 402 with a `topup_url`.

## Account

- Team `gxb-vc` (owner `christian@gen.co`), created 2026-10-08 with $1.00 free credit. Token: `TREG_TOKEN` in `.env` (see `.env.example`). Do not print it.
- The official Python CLI (`treg`, PyPI `tools-registry`, at `~/.local/bin/treg`) is also installed. It is signed in as `christian@gxb.vc` on a different team, `gxb`, with its own $1.00. `treg-cli` does not use it.
- We did not run `treg mcp install` or `treg skill install`. They write MCP config and skills into every agent on the machine. Ask Christian first.
- Source of the service: https://github.com/superdesigndev/treg (Jason Zhou, Superdesign).
