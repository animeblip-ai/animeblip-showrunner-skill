# Animeblip Showrunner — quick reference

Producer/reviewer for Animeblip episodes — call the per-op script directly
(`python3 scripts/<name>.py …`). Add `--dry-run` to any command to simulate with no spend.
You drive by polling + sending messages; there is no managed loop.

## Env

| Var | Required | Default |
|---|---|---|
| `ANIMEBLIP_API_KEY` | yes (ab_…) | — |
| `ANIMEBLIP_BASE_URL` | no | `https://studio.animeblip.com` |

## Commands

| Script | Purpose |
|---|---|
| `create_project.py --title T [--idea ..] [--aspect 9:16] [--lang en]` | New **project (series)** + its **episode 1 (story)** → storyId. Art style is fixed (**Anime6**) — change it on the website, not here |
| `send_message.py --story S --message "…"` | Send a message **verbatim** — full end-to-end brief, or one scoped step ("Write the screenplay only, then stop") |
| `query_session.py --story S [--after-seq N]` | One status snapshot (state / progress / failures / final video / new messages) — the **poll channel** |
| `get_project_state.py --story S` | Review channel — scenes + series characters[]/elements[] ({ key, name, imageUrl }) |
| `create_episode.py --from S` | Next episode in the SAME series (reuses characters) |
| `list_projects.py [--limit N] [--cursor C]` | List projects (series) + their episodes ({order, storyId}) — find one to resume |

## Per-episode flow

```
create_project → send_message (full brief + end-to-end line) → poll query_session → deliver
            → [optional human review] → create_episode --from S → …
```

## Triage (apply on every poll)

- content/policy/moderation → **rewrite** that shot (English first; if still flagged, **Chinese** re-send) — see docs/triage-rules.md
- other failure → **retry** just the affected scene(s)
- `agent_processing` / `batch_running` → keep polling (~60s, no faster)
- final video URL present → deliver (surface for human sign-off if the user asked)
- new `messages[]` (agent asked something) → answer via `send_message.py`

## Keep spend bounded

- Stop after ~**2** retries per shot / ~**6** fix-sends per episode → surface to the human (no retry spiral).

Full detail: **SKILL.md** (canonical) and **docs/**.
