---
layout: default
title: CHEK2 Daily News
---

<!--LATEST_REPORT-->
# CHEK2 Daily Report — 2026-11-03

## TL;DR

Two new items today, neither of which changes the established CHEK2 risk baseline. A Canadian study of rapid genetic testing at breast cancer diagnosis (panel included CHEK2) found decision regret is uncommon one year later — useful reassurance for carriers navigating testing at diagnosis — and a JAAD Case Reports case describes the first reported identification of a pathogenic CHEK2 c.1100delC variant following a subungual melanoma diagnosis, adding an anecdotal data point to the still-weak CHEK2–melanoma association.

## New Research Publications

**2026-11-01 — "Decision regret following rapid genetic testing at the time of breast cancer diagnosis" — *Cancer***

Among 871 Canadian women offered rapid multi-gene panel testing (BRCA1/2, CDH1, PALB2, PTEN, STK11, TP53, ATM, and CHEK2) at the time of breast cancer diagnosis, 9.5% carried a pathogenic variant, and only 7.9% reported any decision regret one year later (92.1% reported none). Regret was higher in women from racialized groups (13.4% vs 4.4% in White women) and in those with a positive BRCA1/BRCA2 result; regret was not specifically elevated for CHEK2-positive results. For CHEK2 carriers, this supports that learning one's status at diagnosis is generally well tolerated, while underscoring the value of follow-up counseling for higher-regret groups.
Link: https://pubmed.ncbi.nlm.nih.gov/42967224/ (PMID: 42967224, DOI: 10.1002/cncr.70663)

**2026-10 — "Subungual melanoma leading to identification of a pathogenic CHEK2 variant" — *JAAD Case Reports*** (case report; published online Oct 2026, not previously reported)

University of Michigan dermatologists report a 68-year-old woman diagnosed with subungual (under-the-nail) melanoma whose family history (sister with metastatic breast cancer at 44) prompted panel testing, which revealed the Northern European founder truncating variant CHEK2 c.1100delC (p.T367Mfs*15) — the first reported case of a pathogenic CHEK2 variant identified after a subungual melanoma diagnosis. This is a single case report and does not establish subungual melanoma as a CHEK2-associated cancer; the broader evidence for any CHEK2–melanoma link remains limited and inconsistent, so this should not change screening practice for carriers. It does illustrate the practical value of family-history-guided genetic evaluation after unusual cancer presentations.
Link: https://jaadcasereports.org/article/S2352-5126(26)00620-X/pdf
<!--END_LATEST_REPORT-->

## Report Archive

{% assign report_pages = site.pages | where_exp: "p", "p.name contains '20'" | sort: "name" | reverse %}
<ul>
{% for p in report_pages %}
  <li><a href="{{ p.url | relative_url }}">{{ p.name | remove: ".md" }}</a></li>
{% endfor %}
</ul>
