---
layout: default
title: CHEK2 Daily News
---

<!--LATEST_REPORT-->
# CHEK2 Daily Report — 2026-10-15

## TL;DR

One clinical trial update today: the phase I/Ib trial of olaparib plus the oral DNMT inhibitor ASTX727 in homologous-recombination-repair–mutated solid tumors — which explicitly enrolls patients with germline or somatic CHEK2 mutations — posted an update on ClinicalTrials.gov on 2026-10-15 and is now listed as active, not recruiting. No new publications, preprints, or news items change the established CHEK2 risk baseline.

## Clinical Trials Updates

- **2026-10-15 — Olaparib and ASTX727 in BRCA1/2- and Homologous Recombination Deficient (HRD)-Mutated Tumors (NCT06177171).** This single-center phase I/Ib trial combines the PARP inhibitor olaparib with ASTX727 (oral decitabine/cedazuridine) in adults with advanced solid tumors carrying germline and/or somatic mutations in BRCA1, BRCA2, PALB2, ATM, or **CHEK2**; a phase Ib expansion cohort specifically includes germline CHEK2 carriers. The registry record was updated on 2026-10-15 with status "active, not recruiting," indicating enrollment is complete and the combination is being evaluated — relevant for CHEK2 carriers because it tests whether adding a DNA-hypomethylating agent can deepen PARP-inhibitor responses in CHEK2-mutated tumors. [ClinicalTrials.gov](https://clinicaltrials.gov/study/NCT06177171)
<!--END_LATEST_REPORT-->

## Report Archive

{% assign report_pages = site.pages | where_exp: "p", "p.name contains '20'" | sort: "name" | reverse %}
<ul>
{% for p in report_pages %}
  <li><a href="{{ p.url | relative_url }}">{{ p.name | remove: ".md" }}</a></li>
{% endfor %}
</ul>
