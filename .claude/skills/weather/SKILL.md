---
name: weather
description: Fetch the current weather or a short forecast for Medellín, Colombia (the default location, or a named city), straight from the terminal via wttr.in — no API key needed. Use when the user asks about the weather, clima, temperatura, pronóstico, or wants to know what it's like outside right now.
---

# Weather

Fetches live weather from [wttr.in](https://wttr.in), a free plain-text weather service. No API key, no signup, no config file — one `curl` call.

## Steps

1. **Pick the location.** Default to `Medellin` unless the user names a different city — if they do, use that instead (URL-encode spaces as `+`, e.g. `Buenos+Aires`).

2. **Pick the format and run one `curl` command** via the Bash tool:

   | User wants | Command |
   |---|---|
   | Quick one-line answer | `curl -s "wttr.in/CITY?format=3"` |
   | Current conditions (a bit more detail) | `curl -s "wttr.in/CITY?format=%C+%t+(feels+%f)+humidity+%h+wind+%w"` |
   | Full report incl. 3-day forecast | `curl -s "wttr.in/CITY"` |
   | Machine-readable data | `curl -s "wttr.in/CITY?format=j1"` |

   e.g. `curl -s "wttr.in/Medellin?format=3"` for the default city's quick answer.

3. **Report the result in plain language** — don't just paste raw command output back at the user; summarize temperature, conditions, and anything else they asked for (feels-like, wind, forecast) in a sentence or two. Respond in whatever language the user is using.

## Notes

- `curl -s --max-time 8 ...` if the network is flaky — wttr.in occasionally times out; a failed request isn't worth retrying more than once.
- This depends on internet access to wttr.in. If the user truly wants offline/no-network weather, that's a different problem (a local sensor or cached data) — say so rather than forcing this tool to do it.
