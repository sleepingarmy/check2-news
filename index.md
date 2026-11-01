---
layout: default
title: CHEK2 Daily News
---

<!--LATEST_REPORT-->
# CHEK2 Daily Report — 2026-11-01
## TL;DR
Two publications surfaced today, neither of which changes the established CHEK2 risk baseline. A large Saudi whole-exome study confirms CHEK2 is among the most variant-rich breast cancer predisposition genes in that population while c.1100delC remains rare outside Northern Europe, and a Spanish hereditary polyposis panel study adds to evidence that CHEK2 variants occasionally appear in colorectal polyposis cohorts.

## New Research Publications

**2026-10-27 — "Germline variation across breast cancer predisposition genes: whole-exome evidence" — *Scientific Reports*** (published Oct 27; missed by prior runs, not previously reported)

Researchers whole-exome sequenced 11,093 Saudi individuals without a cancer diagnosis and classified protein-altering variants across 20 breast cancer–associated genes. CHEK2 carried the second-highest variant burden (26 variants, behind ATM's 31), with missense variants predominating (13 of 26), and many variants were absent from ClinVar — highlighting gaps in population diversity of variant databases. Notably, the Northern European founder variant c.1100delC was *not* observed at elevated frequency, consistent with its known ethnic specificity. This reinforces, rather than changes, the baseline: CHEK2 risk variants are common in the general population (~1–2%), but the truncating variants that drive the ~2–2.7x breast cancer risk are population-specific, so risk estimates should not be transferred uncritically across ancestries.
Link: https://www.nature.com/articles/s41598-026-73559-8 (DOI: 10.1038/s41598-026-73559-8)

**2026-11-01 — "Diagnostic Yield of Expanded Multigene Testing in Hereditary Polyposis Syndromes: Functional Characterization of Clinically Relevant Variants" — *Cancer Genomics & Proteomics***

A Spanish group tested 50 patients with suspected hereditary gastrointestinal polyposis using a 101-gene panel, finding pathogenic/likely pathogenic variants in 28%. Beyond the established polyposis genes (APC, MUTYH, SMAD4, PTEN), variants appeared in four broader cancer-predisposition genes — ATM, CDKN2A, **CHEK2**, and FANCM. This is a single-variant observation for CHEK2 in a small polyposis cohort, so it does not establish a new CHEK2–polyposis association, but it adds to the ongoing debate about whether CHEK2 carriers warrant enhanced colorectal surveillance (current evidence for a colorectal link remains weaker than for breast and prostate).
Link: https://pubmed.ncbi.nlm.nih.gov/42966627/ (PMID: 42966627, DOI: 10.21873/cgp.20619)
<!--END_LATEST_REPORT-->

## Report Archive

{% assign report_pages = site.pages | where_exp: "p", "p.name contains '20'" | sort: "name" | reverse %}
<ul>
{% for p in report_pages %}
  <li><a href="{{ p.url | relative_url }}">{{ p.name | remove: ".md" }}</a></li>
{% endfor %}
</ul>
