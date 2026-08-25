# check2-news

Automated daily digest of research, clinical news, and database updates related to the **CHEK2** gene (checkpoint kinase 2, 22q12.1) and its pathogenic variants.

Reports are generated daily at 07:00 by an automated agent job, committed here, and pushed to this repo.

## Report format

Each `YYYY-MM-DD.md` file contains:

- **TL;DR** — the day's most important finding, or "No significant new findings today."
- **New Research Publications** — PubMed-indexed papers with plain-English summaries
- **Preprints** — bioRxiv / medRxiv
- **Clinical Trials Updates** — ClinicalTrials.gov studies updated in the last 7 days
- **News & Clinical Guidance** — press coverage, guideline changes (quality-filtered)
- **Database & Classification Changes** — ClinVar / OMIM checks (Mondays)

## Reference baseline

Risk estimates the reports compare against (GeneReviews NBK615090; OMIM 604373/609265):

| Finding | Baseline |
|---|---|
| Truncating variants (e.g. c.1100delC) | ~2-2.7x breast cancer OR; 20-30% lifetime female breast risk |
| Ile157Thr / Ser428Phe (missense) | Low-penetrance, OR ~1.3 |
| Contralateral breast cancer | HR ~2.0-2.25 |
| Male breast cancer | OR ~3.1 |
| Prostate cancer | ~2x risk |
| Li-Fraumeni syndrome | NOT caused by CHEK2 (early reports refuted) |

`.seen.json` tracks already-reported items (DOI/PMID/NCT/URL) so nothing is reported twice.
