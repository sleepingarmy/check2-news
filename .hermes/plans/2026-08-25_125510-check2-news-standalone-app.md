# check2-news as a Standalone Application — Plan

**Goal:** Remove the dependency on the local Hermes cron agent. The repo itself should fetch CHEK2 news on a schedule, generate reports, and publish them — with an optional browsable site on GitHub Pages.

**Architecture:** Replace the Hermes cron job with GitHub Actions. A Python script in the repo does the data gathering (all sources have free, keyless APIs), reports stay as markdown, and GitHub Pages renders them either via Jekyll (zero code) or a small static site build. For AI-written summaries, optionally call an LLM API from the Action using a repo secret; otherwise fall back to template-based summaries from structured metadata.

**Key constraint discovered:** GitHub Actions scheduled workflows on *private* repos consume Actions minutes (free tier: 2,000 min/month — plenty). GitHub Pages from a private repo requires a paid plan (Pro/Team) OR making the repo public. Decision point below.

---

## Architecture Options

### Option A — Actions + template summaries, no LLM (simplest, zero cost, zero secrets)
The fetch script writes reports purely from structured metadata: title, journal, authors, date, abstract excerpt (first 2-3 sentences), source link, and auto-classification (germline vs somatic/CHIP, truncating vs missense via keyword rules).
- Pros: fully free, no API keys, deterministic, no hallucination risk, runs in <1 min
- Cons: "summaries" are abstract excerpts, not synthesized plain-English analysis; no cross-item TL;DR judgment

### Option B — Actions + LLM summaries (keeps current report quality)
Same as A, but after gathering new items the script calls an LLM API (OpenRouter, matching your current setup) to write the 2-3 sentence summaries and TL;DR, with the same CHEK2 baseline guardrails baked into the prompt.
- Pros: same quality as today's Hermes-generated reports
- Cons: needs an API key in repo secrets, small per-run cost (~a few cents/day), nondeterminism, needs a no-fabrication validation step (check every PMID/DOI/URL in the output exists in the fetched data)

### Option C — Hybrid (recommended)
Script does deterministic gathering + excerpt summaries (Option A). LLM is used ONLY for the TL;DR and importance flagging, with a hard validation pass. If the LLM call fails or key is absent, the report still generates with excerpts only. Degrades gracefully, never blocks the pipeline.

---

## Plan (assuming Option C, repo made public)

### Phase 1: Decouple gathering logic from the agent

**Task 1: Create `fetch_news.py`**
- Create: `scripts/fetch_news.py`
- Port the source queries from the cron prompt into plain Python (urllib, no deps):
  - `fetch_pubmed()` — esearch (reldate=2) → efetch XML → parse PMIDs, titles, abstracts, journals, dates, DOIs
  - `fetch_biorxiv()` — api.biorxiv.org details endpoint for yesterday/today, filter "CHEK2" in title/abstract
  - `fetch_trials()` — clinicaltrials.gov/api/v2/studies?query.term=CHEK2, filter lastUpdatePostDate within 7 days
  - `fetch_news_web()` — hardest to port: no keyless general-news API. Options: Google News RSS (`https://news.google.com/rss/search?q=CHEK2+mutation&hl=en-US&gl=US&ceid=US:en`) — free, parseable, works in Actions. Quality filter: domain allowlist/blocklist in `scripts/config.py`
  - `fetch_databases()` — Monday-only ClinVar (NCBI eutils db=clinvar works keyless) + OMIM note check (OMIM needs an API key for full access; fallback: skip with a note, or accept an `OMIM_API_KEY` secret)
- Each fetcher returns a list of dicts: `{id, source, title, date, url, abstract, doi_or_pmid}`
- Dedupe: load `.seen.json`, drop known ids, return only new items
- Run locally to verify: `python3 scripts/fetch_news.py --dry-run` prints new items without writing

**Task 2: Create `classify.py`**
- Create: `scripts/classify.py`
- Keyword-based tagging per item: `germline` vs `somatic/CHIP` (words: "clonal hematopoiesis", "CHIP", "somatic"); variant mentions extracted with regex (`c\.\d+[A-Za-z>delins]+`, `p\.[A-Z][a-z]{2}\d+[A-Za-z*]+`); variant class: truncating vs missense
- Baseline check: flag items containing risk-estimate language (OR, hazard ratio, lifetime risk + number) for the LLM/TL;DR to scrutinize
- Unit tests: `tests/test_classify.py` — feed known strings (e.g., "c.1100delC", "clonal hematopoiesis", "I157T") assert correct tags

**Task 3: Create `summarize.py`**
- Create: `scripts/summarize.py`
- If `OPENROUTER_API_KEY` env var present: send new items + the CHEK2 baseline block (same guardrails as the cron prompt) to the model, request JSON: `{id: {summary, why_it_matters, tldr_candidate}}`
- Validation: reject any summary whose id isn't in the fetched set; reject if it introduces a PMID/DOI not in the input; on any failure → excerpt fallback for that item
- If no key: excerpt mode (first 2-3 sentences of abstract)
- Tests: mock LLM response; test validation rejects fabricated ids

**Task 4: Create `build_report.py`**
- Create: `scripts/build_report.py`
- Renders `YYYY-MM-DD.md` in the exact current format (header, TL;DR, sections, "Sources checked with no new items" footer)
- TL;DR: LLM pick if available, else first item's title, else "No significant new findings today."
- Updates `.seen.json`, prunes entries older than 90 days
- Tests: given fixture items, assert report structure; given zero items, assert minimal report

**Task 5: `main.py` entrypoint + local end-to-end test**
- Create: `scripts/main.py` wiring fetch → classify → summarize → build → (optionally) git commit
- Run locally with `OPENROUTER_API_KEY` unset and set; verify both modes produce valid reports

### Phase 2: GitHub Actions automation

**Task 6: Create `.github/workflows/daily-report.yml`**
```yaml
name: daily-report
on:
  schedule: [{ cron: "13 7 * * *" }]   # 07:13 UTC (avoid top-of-hour congestion); adjust tz as desired
  workflow_dispatch: {}
permissions:
  contents: write
jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - run: pip install -r requirements.txt   # or stdlib-only
      - run: python scripts/main.py
        env:
          OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
          OMIM_API_KEY: ${{ secrets.OMIM_API_KEY }}
      - name: commit and push
        run: |
          git config user.name "check2-news-bot"
          git config user.email "bot@users.noreply.github.com"
          git add -A
          git commit -m "daily report $(date +%F)" || echo "nothing to commit"
          git push
```
- Note: GitHub scheduled workflows can be delayed ~minutes under load and require the repo to have periodic activity (schedules are paused after 60 days of repo inactivity — commits by the workflow itself count, so this self-sustains, but worth documenting)

**Task 7: Add secrets and test**
- Repo Settings → Secrets → Actions: add `OPENROUTER_API_KEY` (Option B/C), optionally `OMIM_API_KEY`
- Run via `workflow_dispatch` manually; verify report committed; run again same day to verify dedupe ("nothing to commit" path)
- Verify failure path: temporarily unset key → excerpt-mode report still generates

### Phase 3: GitHub Pages site

**Task 8: Choose site approach**
- **8a (recommended, zero-build): Jekyll.** Add `_config.yml` (theme: minima or just-the-docs), `index.md` auto-generated by `build_report.py` listing reports newest-first, and set Pages source to "GitHub Actions" or "Deploy from branch: main /root". Jekyll renders each daily `.md` as a page automatically.
- **8b: Static HTML.** Small `scripts/build_site.py` (markdown → HTML via `markdown` lib, one template with your Y2K-cybercore CSS: #0060fa accents, light background, clean cards). More design control, one more build step in the Action.
- Add site build/deploy steps to the same workflow (actions/deploy-pages) so reports publish atomically with each commit

**Task 9: Pages config + verify**
- Settings → Pages → enable; verify https://sleepingarmy.github.io/check2-news/ renders the latest report and archive index
- Add a "Report archive" index page regenerated each run

### Phase 4: Decommission local cron

**Task 10:** Once the Actions workflow has run successfully for 2-3 consecutive days, delete the Hermes cron job (`cronjob action='remove'`, job d9488f5ae7ae) and note the migration in the README.

## Files to Create/Change

- Create: `scripts/fetch_news.py`, `scripts/classify.py`, `scripts/summarize.py`, `scripts/build_report.py`, `scripts/build_site.py` (if 8b), `scripts/main.py`, `scripts/config.py`
- Create: `.github/workflows/daily-report.yml`
- Create: `_config.yml`, `index.md` (Jekyll) or `site/` templates (static)
- Create: `tests/test_classify.py`, `tests/test_build_report.py`, `tests/fixtures/`
- Create: `requirements.txt` (stdlib-only if possible; else `requests`, `markdown`)
- Modify: `README.md` (architecture, self-run instructions, secrets setup)
- Keep: `.seen.json` (committed, shared dedupe state — Actions runner is ephemeral so this MUST stay in git)

## Risks / Tradeoffs / Open Questions

1. **Private vs public.** Pages from a private repo needs GitHub Pro. Making it public is free but exposes the reports (fine — they're public sources) and the workflow code. `.seen.json` is harmless either way. Decision needed.
2. **Google News RSS reliability.** It's unofficial and can rate-limit; quality-filter logic must be defensive. Worst case the news section degrades to "unavailable" for a run while PubMed/trials still work.
3. **OMIM API key.** Free but requires application/approval; Monday database section works without it via ClinVar alone, so treat OMIM as optional enhancement.
4. **LLM nondeterminism.** Mitigated by validation (Step in Task 3) + excerpt fallback. Never let an LLM failure fail the workflow.
5. **Schedule drift.** Actions cron isn't exact; reports may appear 07:05-07:30. Fine for a daily digest.
6. **60-day inactivity rule.** The workflow's own commits count as activity, but if the pipeline has a long silent stretch with zero commits, GitHub may pause the schedule. Mitigation: always commit something (even a `.heartbeat` timestamp) or accept manual re-enable.
7. **Open question:** keep summaries in English only, or is there value in tagging items by cancer type for future filtering on the site (breast/prostate/colorectal)? Cheap to add in classify.py now, hard to retrofit.

## Acceptance criteria

1. `workflow_dispatch` run produces a correct report with no local machine involved
2. Same-day second run produces no duplicate items
3. Site renders latest report + archive on GitHub Pages
4. Workflow succeeds with `OPENROUTER_API_KEY` absent (excerpt mode)
5. Local Hermes cron job removed after 2-3 clean scheduled runs
