# Setup

Everything below can be done from GitHub's website — no terminal, no Python
installed on your own computer required, even for a private league. If you'd
rather work locally, everything here also works as `git clone` + edit files +
`git push`; the pipeline is stdlib-only Python, nothing to install.

Total time: usually 15–20 minutes, most of it waiting on the first archive
to download.

## Before you start

- You need to be a member of the ESPN league you're pointing this at.
- You need a (free) GitHub account of your own. **ESSENTIAL**: On the main fantasy-football-dashboard-template page, click
  **Use this template** (green button, top of the repo page) to get your own
  independent copy first. Your copy shares no data and no credentials with
  this original one or with anyone else's copy.
- If your league is **private** (most are — check by opening it in an
  incognito window while logged out of ESPN; if it asks you to sign in,
  it's private), you'll need two cookie values from your own ESPN login.
  Getting those is the one genuinely technical-feeling step below, and
  there's no way around it — see step 3.

## Checklist

- [ ] **1. Use this template.** Click the green **Use this template** button
      at the top of this repo (on the main page, not this one), choose your own account, name it whatever you
      want. This creates a brand-new, independent repo — not a fork, no
      shared history, and nothing you do in it ever touches this one.

- [ ] **2. Find your league id.** Open your league on ESPN's website. The
      URL looks like `fantasy.espn.com/football/team?leagueId=1234567&...` —
      the number after `leagueId=` is what you need.

- [ ] **3. If your league is private, get your two cookie values.**
      Sign in to fantasy.espn.com in a real browser first — the cookies
      only exist once you're logged in. Then find two cookies named `SWID`
      and `espn_s2` under `https://fantasy.espn.com` using your browser's
      dev tools. Not sure which browser you're on? Check its icon in your
      dock/taskbar rather than guessing — the steps below don't overlap
      between browsers, and following the wrong one is the most common way
      to get stuck here.

      **Chrome or Edge:**
      1. Press `F12` (Windows) or `Cmd+Option+I` (Mac) to open DevTools.
      2. Click the **Application** tab along the top of the DevTools panel.
         If you don't see it, click the `»` overflow arrow to find it.
      3. In the left sidebar under **Storage**, expand **Cookies** and
         click `https://fantasy.espn.com`.
      4. Find the rows named `SWID` and `espn_s2`. Click a row's **Value**
         cell, select all the text in it, and copy — the column is often
         too narrow to show the full value, so widen it or double-click
         into the cell first rather than copying what's visible.

      **Firefox:**
      1. Press `F12` (Windows) or `Cmd+Option+I` (Mac) to open DevTools.
      2. Click the **Storage** tab (Firefox doesn't call this Application).
      3. In the left sidebar, expand **Cookies** and click
         `https://fantasy.espn.com`.
      4. Same as above: click into each value cell for `SWID` and `espn_s2`
         and copy the full value, not the truncated display.

      **Safari (Mac only) — this is the one people get stuck on:**
      Safari hides its dev tools by default, and its layout doesn't match
      Chrome's at all, so don't try to follow the Chrome steps here.
      1. Enable the Develop menu, if you haven't already: Safari menu →
         **Settings** (**Preferences** on older macOS) → **Advanced** tab →
         check **Show features for web developers** / **Show Develop menu
         in menu bar** (wording varies by macOS version).
      2. Back on fantasy.espn.com, open **Develop → Show Web Inspector**
         (or `Cmd+Option+I`).
      3. Click the **Storage** tab in the panel that opens.
      4. Expand **Cookies** in the sidebar → click `https://fantasy.espn.com`.
      5. Click the `SWID` row, then read the full value from the detail
         pane below the table — Safari truncates long values in the row
         itself and won't let you select from there. Copy from the detail
         pane. Do the same for `espn_s2`.
      6. Paste into a plain text field (like the notes app) before pasting
         into GitHub, and check there's no stray space or line break at the
         start or end — Safari copies have occasionally carried one, and a
         copy that looks identical but isn't is the hardest kind to
         troubleshoot later.

      Skip this step entirely if your league is public.

- [ ] **4. Add your cookies as repo secrets** (private leagues only). In
      **your new repo** (not this one): Settings → Secrets and variables →
      Actions → New repository secret. Add two:
        - `SWID` — paste the value exactly as copied, including the curly
          braces `{...}` if it had them.
        - `ESPN_S2` — paste the value as copied.

      These never appear in your code, your commits, or anywhere visible —
      GitHub stores them encrypted and only your own workflows can use them.

- [ ] **5. Set your league id.** Edit `config/league.ini` (pencil icon on
      GitHub, top right of the file) and fill in `id`. Leave
      `first_season` blank for now — the next step finds it for you. Set
      `current_season` to the season you're playing/drafting now. Commit
      the change (green "Commit changes" button).

- [ ] **6. Find out how far back your league's data actually goes.** Go to
      the **Actions** tab → **Probe league history** (left sidebar) →
      **Run workflow**. Enter your league id and a starting guess like
      `2010`. Run it, wait ~30 seconds, click into the finished run, and
      read the log: it prints one line per season, `ok` or `FAIL`. The
      earliest year that comes back `ok` is your real `first_season` — ESPN
      doesn't go back further than that for any league, and the cutoff is
      different for every one. Put that number into `config/league.ini`'s
      `first_season` and commit.

- [ ] **7. Fill in `config/draft.ini`.** Every field has a comment above it
      explaining what it does and whether you actually need to set it — most
      of them are optional and safe to leave blank until you know the
      answer. The draft date itself isn't one of these fields: it's read
      straight from ESPN once you schedule it there, automatically.

- [ ] **8. Turn on GitHub Pages.** Settings → Pages → under "Build and
      deployment", set **Source** to **GitHub Actions**. (Not "Deploy from
      a branch" — that's the wrong option and won't work with this repo.)

- [ ] **9. Run the real thing.** Actions tab → **Refresh league data** →
      **Run workflow**. This pulls your full history from ESPN, checks it
      against ESPN's own records, builds the site, and publishes it — all
      in one run, usually 1–3 minutes depending on how many seasons your
      league has. Watch it in the Actions tab; if it fails, the failed step
      tells you why (almost always a wrong league id or a cookie that
      doesn't match).

- [ ] **10. Find your URL and check it.** Settings → Pages will show your
      live URL once the first deploy finishes (also linked from the
      finished Actions run). Open it. If any manager names look wrong or
      generic (ESPN's own suggestion, first name + last initial), that's
      the next step below.

- [ ] **11. Fix manager display names.** The first run created
      `config/owners.ini` for you, with one entry per manager it found and
      a suggested display name. Edit any you don't like, save, and
      re-run **Refresh league data** (step 9) to rebuild with the new
      names. If the same real person played under two different ESPN
      accounts over the years, `owners.ini`'s own comments explain how to
      merge them with `merge_into` so their career stats combine into one.

That's it — from here it runs itself. The scheduled refresh (see
`.github/workflows/refresh.yml`) pulls new results and redeploys weekly
during the season automatically; nothing further to do unless you want to
change dues, the draft order, or anything else in `config/`.

## If something goes wrong

- **Refresh workflow fails immediately on the fetch step** — almost always a
  wrong `id` in `league.ini`, or (for a private league) a cookie value that
  got truncated or mistyped when pasted into the secret. If you copied the
  cookies from Safari, this is the first thing to double-check — see the
  note in step 3.
- **Every season fails, even ones that should exist** — check `SWID` still
  has its curly braces; ESPN wants them.
- **Validation step fails** — this is the pipeline refusing to publish data
  that doesn't match what ESPN itself has on record. Rare, and worth
  reading the actual error rather than re-running blind.
- **Site loads but shows no data** — check the Pages source is set to
  "GitHub Actions" (step 8), not "Deploy from a branch".

## What you'll never need to touch

The `pipeline/` code, the `site/` code, and the two workflow files are the
same ones this repo was built and tested against — they read everything
they need from `config/` and from ESPN itself. The only files you should
ever need to edit are inside `config/`.
