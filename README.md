# Fantasy League Dashboard (template)

A static site for your fantasy football league, built from ESPN's own fantasy
API. Click **Use this template** (green button above) to get your own copy —
see **[SETUP.md](SETUP.md)** for the full walkthrough, entirely doable from
GitHub's website, no local install required.

**Design principle:** this is not a second copy of the ESPN app. Live scores,
current rosters and this week's matchups all already exist in ESPN's own app
and aren't worth rebuilding. This site is for the data ESPN collects but never
surfaces — cross-season records, career head-to-head, luck vs. skill, draft
return on investment, and the long-memory stuff a league argues about.

Two surfaces: a **draft-day splash page** (countdown, draft order, dues) and
an **all-time analysis** section behind it (records, rivalries, luck, lineup
efficiency, draft history, rosters by season).

## How it works

```
ESPN v3 API  ->  data/raw/{year}/{view}.json.gz   (archived, committed)
             ->  site/data/*.json                 (generated, committed)
             ->  site/                            (static HTML/CSS/JS, no build)
```

`site/` is entirely self-contained, so GitHub Pages publishes that one
directory and nothing needs a build step.

The raw archive is committed on purpose. ESPN's historical data has a habit of
becoming unavailable, and once a season is in `data/raw/` this site never
depends on ESPN to hand it back again.

A GitHub Action re-runs the pipeline weekly during the season and commits any
changes; GitHub Pages serves `site/`. See `.github/workflows/refresh.yml`.

## Privacy

ESPN identifies managers by SWID (a GUID tied to their ESPN account) and
ships their real names alongside it. Neither belongs in a repo you might make
public, so `pipeline/identity.py` replaces every SWID with a deterministic
`mgr_<hash>` id and strips name fields **before** anything is written to
`data/raw/` — the raw archive never carries anyone's real identity.

Real names stay in `config/identities.local.json`, which is gitignored and
never leaves whichever machine fetched them (or, if you only ever run the
pipeline through GitHub Actions, never leaves GitHub's own temporary runner
at all). The only names that reach the repo are the display strings you (or
the pipeline's own suggestions) write into `config/owners.ini` — those are
rendered on the site, so keep them to whatever you're comfortable showing
your league.

## Configuration

Everything you can change lives in `config/` — see each file's own comments,
or the full walkthrough in **[SETUP.md](SETUP.md)**.

| File | What it holds |
|---|---|
| `config/league.ini` | League id and season range. |
| `config/owners.ini` | Public display names. Auto-created on your first run; edit afterward. |
| `config/draft.ini` | Everything on the splash page except the draft date, which comes from ESPN. |
| `.env` | ESPN auth cookies, for running the pipeline from your own machine. Gitignored. Not needed if you only ever run it through GitHub Actions. |

## Pipeline

Stdlib only — no pip install, no venv. All of this also runs from the
Actions tab with no local setup; see SETUP.md.

Check what ESPN will actually return for your league, season by season,
before trusting it:

```bash
python3 pipeline/probe.py --league-id YOUR_ID --from 2015
```

Pull and archive every view for every season. Resumable — files already
archived are skipped, so re-running is cheap and a failed backfill picks up
where it left off:

```bash
python3 pipeline/fetch_raw.py
```

Rebuild the site's JSON from the archive and config (no network):

```bash
python3 pipeline/build_site_data.py
```

Set a passphrase for the League Home button, if you want one (prompts
hidden, prints a hash to paste into `config/draft.ini`):

```bash
python3 pipeline/hash_passphrase.py
```

## Local preview

The site fetches its JSON, so `file://` will not work. Serve it:

```bash
python3 -m http.server 8017 --directory site
```

Then open http://localhost:8017.
