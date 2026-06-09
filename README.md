[Clinical Trials Fda Scraper](https://apify.com/constructive_calm/clinical-trials-fda-scraper?fpr=data)

Unified clinical-trial + FDA intelligence in one actor. Search ClinicalTrials.gov, fetch FDA drug approvals, 510(k) and PMA device clearances, adverse events, recalls, and active drug shortages — and roll them all up into a single sponsor-pipeline view that no incumbent ships.

Built on **two official, public, no-auth APIs**: ClinicalTrials.gov v2 and OpenFDA. Every output row carries a `verifyUrl` that links to the canonical public page so you can audit the data with one click.

## What does this actor do?

It pulls structured biomedical regulatory data and unifies it into one normalized output shape. Ten modes cover the full surface:

| Mode | Records returned | Buyer use case |
| --- | --- | --- |
| `trials` | NCT records with phase, status, conditions, comparators, locations, primary endpoints | Search trials by indication / sponsor / phase / status |
| `trial_details` | Full study record incl. eligibility criteria, secondary endpoints, sponsors/collaborators | Deep dives on specific NCTs |
| `drug_approvals` | NDA / ANDA / BLA submissions with all status changes, products, NDCs | Approval timeline + commercial reality |
| `device_clearances` | 510(k) + PMA records with decision date, decision, product code | Med-device pipeline tracking |
| `adverse_events` | PII-stripped FAERS reports — reactions, seriousness, demographics | Pharmacovigilance signals |
| `recalls` | FDA enforcement (Class I / II / III) — reason, distribution, status | Compliance & QA monitoring |
| `shortages` | Active or resolved drug shortages with availability + reason | Hospital / pharmacy planning |
| `pipeline` | **One row per sponsor** merging trials + approvals + clearances + recalls + shortages + next catalyst | Alt-data hedge fund, biotech VC, pharma BD |
| `monitor` | Incremental change events vs. prior runs — new trials, status changes, new approvals, recalls | Scheduled daily / weekly diffs |
| `ai_summary` | Gemini 2.5 Flash structured summary — TL;DR, risk flags, milestone class, outcome prior | Premium analyst layer |

**Free trial:** the first 10 chargeable events of every run are free. Pay only on the 11th event onward.

## Why use this actor?

- **Biotech VCs / hedge funds** — build a catalyst calendar from `pipeline.nextCatalystDate` and `upcomingMilestones[]`. Every trial readout, PDUFA-class submission, and 510(k) decision is normalized and ticker-tagged when the sponsor maps to an SEC filer.
- **Pharma BD / corp dev** — find Phase 2 oncology assets at small biotechs, see their FDA submission history, then pivot to drug labels for MOA inference. Sponsor resolver bridges to SEC tickers when public.
- **CROs / clinical operations** — query for `RECRUITING` Phase 3 trials in a target country, get full `locations[]` with city + facility + status.
- **Insurance underwriters / pharmacovigilance teams** — pull adverse-event trends per drug with `serious=true` breakdowns. PII is stripped (no names, no contact info, no narrative free text) before the row is written.
- **Hospital pharmacy / supply chain** — `shortages` mode lists every currently-active FDA drug shortage with availability, generic name, and affected NDCs. Schedule daily and pipe alerts.
- **Compliance / QA** — `recalls` mode returns Class I / II / III enforcement actions with classification, recall reason, and distribution pattern. Combine with `monitor` for new-recall alerts.
- **AI / LLM training** — every record has a stable PK (NCT, ANDA, k_number, recall_number) and a public `verifyUrl`. Perfect for grounded biomedical knowledge graphs.
- **Public-health researchers** — trial geography by condition, sponsor class breakdowns, no API key required.

## How to use this actor

1. **Pick a mode** — see the table above. The most common starting points are `trials` (search) and `pipeline` (sponsor rollup).
2. **Set your filters** — conditions, phases, sponsor identifiers (ticker / name / domain / CIK), date range, etc.
3. **Run it** — first 10 chargeable events per run are free; you only pay on the 11th and beyond. Set `maxItems: -1` for unlimited bulk extraction (the actor uses `nextPageToken` and `search_after` pagination automatically).
4. **Verify any record** — click the `verifyUrl` field on any output row to land on the source's canonical page (clinicaltrials.gov, accessdata.fda.gov, etc.).
5. **Export** as JSON / CSV / Excel from the dataset, or hit the dataset API directly to chain into a downstream workflow.

## Input

### Search Phase 3 lung-cancer trials currently recruiting

```
{
    "mode": "trials",
    "conditions": ["non-small cell lung cancer"],
    "phases": ["PHASE3"],
    "statuses": ["RECRUITING"],
    "maxItems": 100
}
```

### Full trial details by NCT ID

```
{
    "mode": "trial_details",
    "nctIds": ["NCT04368728", "NCT05636956"],
    "maxItems": 10
}
```

### Pfizer's recent FDA drug approvals

```
{
    "mode": "drug_approvals",
    "sponsorIdentifiers": ["PFE"],
    "dateFrom": "2024-01-01",
    "maxItems": 50
}
```

### Medtronic's 510(k) cardiac-monitor clearances

```
{
    "mode": "device_clearances",
    "applicantName": "Medtronic",
    "productCode": "DXN",
    "devicePathways": ["510k"],
    "maxItems": 25
}
```

### Serious Ozempic adverse events

```
{
    "mode": "adverse_events",
    "drugName": "Ozempic",
    "seriousOnly": true,
    "maxItems": 200
}
```

### Class I drug recalls in last 90 days

```
{
    "mode": "recalls",
    "recallClassifications": ["Class I"],
    "recallDomains": ["drug"],
    "dateFrom": "2026-02-01",
    "maxItems": 50
}
```

### Active drug shortages

```
{
    "mode": "shortages",
    "shortageStatus": "currently_in_shortage",
    "maxItems": 100
}
```

### Sponsor pipeline rollup (the killer mode)

```
{
    "mode": "pipeline",
    "sponsorIdentifiers": ["MRNA", "PFE", "LLY"],
    "maxItemsPerSource": 1000
}
```

### Monitor for changes (run on a schedule)

```
{
    "mode": "monitor",
    "monitorTargets": ["NCT04368728", "MRNA", "PFE"],
    "maxItems": 100
}
```

### AI-powered structured summary

```
{
    "mode": "ai_summary",
    "nctIds": ["NCT04368728"],
    "enableAiSummary": true
}
```

## Output

### Trial record (sample)

```
{
    "type": "trial",
    "id": "NCT06840782",
    "nctId": "NCT06840782",
    "briefTitle": "First-line Immunotherapy-based Standard of Care and Local Ablative Treatments for Oligometastatic NSCLC",
    "conditions": ["Oligometastatic Non-small Cell Lung Cancer (NSCLC)"],
    "interventions": [
        { "type": "EXPERIMENTAL", "name": "Radiation: Radical local treatment", "description": "..." }
    ],
    "comparators": [
        { "type": "ACTIVE_COMPARATOR", "name": "Drug: SoC-based immunotherapy", "description": "..." }
    ],
    "phase": "PHASE3",
    "status": "RECRUITING",
    "enrollmentTarget": 124,
    "actualEnrollment": 124,
    "studyDesign": {
        "allocation": "RANDOMIZED",
        "interventionModel": "PARALLEL",
        "masking": "NONE",
        "primaryPurpose": "TREATMENT"
    },
    "primaryEndpoints": [
        {
            "measure": "Overall Survival (OS)",
            "timeFrame": "From randomization up to two years",
            "description": "Time from randomization to documented death from any cause"
        }
    ],
    "sponsorClass": "OTHER",
    "locations": [
        { "facility": "Gustave Roussy", "city": "Villejuif", "country": "France", "status": "RECRUITING" }
    ],
    "countries": ["France"],
    "primaryCompletionDate": "2030-02",
    "hasResults": false,
    "lastUpdateSubmitDate": "2025-09-12",
    "verifyUrl": "https://clinicaltrials.gov/study/NCT06840782",
    "scrapedAt": "2026-05-04T11:34:45.860Z"
}
```

### Pipeline record (sample, Moderna abridged)

```
{
    "type": "pipeline",
    "id": "0001682852",
    "ticker": "MRNA",
    "cik": "0001682852",
    "trialCountTotal": 111,
    "trialCountByPhase": { "PHASE1": 48, "PHASE2": 22, "PHASE3": 18, "PHASE4": 4 },
    "trialCountByStatus": { "COMPLETED": 71, "RECRUITING": 12, "ACTIVE_NOT_RECRUITING": 19 },
    "trialCountActiveRecruiting": 31,
    "drugApprovalCount": 1,
    "topConditions": [
        { "condition": "SARS-CoV-2", "trialCount": 31 },
        { "condition": "Influenza", "trialCount": 13 },
        { "condition": "Respiratory Syncytial Virus", "trialCount": 11 }
    ],
    "upcomingMilestones": [
        { "nctId": "NCT07089706", "milestone": "primary_completion", "date": "2026-08", "daysUntil": 92 }
    ],
    "nextCatalystDate": "2026-08",
    "nextCatalystType": "primary_completion",
    "nextCatalystId": "NCT07089706",
    "regulatorySignal": 0,
    "verifyUrl": "https://clinicaltrials.gov/search?spons=Moderna%20Inc"
}
```

### Verify any record

| Record type | URL pattern |
| --- | --- |
| Trial | `https://clinicaltrials.gov/study/<NCT_ID>` |
| Drug approval | `https://www.accessdata.fda.gov/scripts/cder/daf/index.cfm?event=overview.process&ApplNo=<#>` |
| 510(k) clearance | `https://www.accessdata.fda.gov/scripts/cdrh/cfdocs/cfpmn/pmn.cfm?ID=<K_NUMBER>` |
| PMA approval | `https://www.accessdata.fda.gov/scripts/cdrh/cfdocs/cfpma/pma.cfm?id=<P_NUMBER>` |
| Recall | `https://www.accessdata.fda.gov/scripts/ires/index.cfm?Recall_Number=<#>` |

## How much does it cost?

Pay-per-event pricing (the first 10 events of any run are free):

| Event | Price | What you get |
| --- | --- | --- |
| `trial-fetched` | $0.0003 | One CT.gov trial record |
| `trial-details-fetched` | $0.0006 | Full trial detail (eligibility + endpoints + results) |
| `drug-approval-fetched` | $0.0004 | One FDA drug application with all submissions |
| `device-clearance-fetched` | $0.0004 | One 510(k) or PMA clearance |
| `adverse-event-fetched` | $0.00015 | One PII-stripped FAERS report |
| `recall-fetched` | $0.0004 | One FDA enforcement action |
| `shortage-fetched` | $0.0004 | One drug-shortage record |
| `pipeline-aggregated` | **$0.02** | One sponsor pipeline rollup (synthesizes 5+ source queries) |
| `change-detected` | $0.0003 | One change event in monitor mode |
| `ai-summary-generated` | $0.05 | One Gemini 2.5 Flash structured summary |

### Typical run costs

- **100-trial NSCLC search:** ~$0.027
- **Pfizer drug approval scan (250 records):** ~$0.10
- **Daily class-I-recall watch (5 records / day):** ~$0.002 / day
- **Weekly pipeline scan, top 10 biotech sponsors:** ~$0.20 / week
- **Sponsor pipeline + AI summary (1 sponsor):** ~$0.07

Compare to enterprise alternatives:

- Citeline / Datamonitor: $10K+/yr/seat
- Phesi: $5K+/yr/seat
- BioPharmaCatalyst: $99/mo
- GlobalData Pharma: enterprise pricing

## Tips & advanced options

- **Bulk extraction** — set `maxItems: -1` and the actor will use `search_after` (Link header `rel="next"`) on OpenFDA and `nextPageToken` on CT.gov to scroll through the entire dataset. There is no built-in cap.
- **OpenFDA API key** — anonymous tier is 1,000 req/day per IP. For bulk users, set the optional `openFdaApiKey` input (free at [https://open.fda.gov/apis/authentication](https://open.fda.gov/apis/authentication), 120K req/day).
- **Sponsor resolution** — every output row has `sponsor.matchConfidence` ∈ `{exact, alias, fuzzy, raw}`. `exact` = SEC ticker bridge, `alias` = hand-curated biopharma index (Wyeth → Pfizer, Genentech → Roche, etc.), `fuzzy` = token-set match, `raw` = passthrough. Audit ambiguous matches by checking the field.
- **Monitor mode** — run on a schedule (Apify integrations → Schedule daily). The first run seeds fingerprints and emits 0 changes; subsequent runs emit only new or changed rows.
- **Pipeline mode runtime** — for busy sponsors (Pfizer ≈ 600+ trials, ≈ 200 approvals), the parallel fan-out takes 30–60 seconds. Tighten with `maxItemsPerSource: 500` to reduce wall time.
- **Combining mode + AI** — set `enableAiSummary: true` in `pipeline` mode to attach a Gemini structured summary to every sponsor row. Adds $0.05 per sponsor.
- **Out of scope (v1)** — EU CTIS, WHO ICTRP, Health Canada, FDA orphan / breakthrough / fast-track designations. These are scoped for a future v2.

## Legal disclaimer

This actor consumes only **public data from official US-government APIs** (ClinicalTrials.gov v2, OpenFDA). Both APIs are open, free, and explicitly distributed for re-use under federal open-data policy. The actor does not bypass authentication, does not scrape content from third-party sites, and does not violate the terms of service of either source.

Adverse-event records are PII-stripped at emit time (`utils/piiStrip.ts`): emails, phone numbers, and SSN-shaped values are redacted. Patient identifiers are not present in the source data.

This data is provided for research, analytical, and informational purposes only. **Do not use for medical decisions.** OpenFDA's own disclaimer states: "Do not rely on openFDA to make decisions regarding medical care."

## FAQ

**Q: My sponsor returned `matchConfidence: "raw"`. Why?**
A: Raw passthrough means the sponsor name didn't match an SEC ticker, didn't match a hand-curated biopharma alias, and didn't fuzzy-match within threshold. The actor still ran the query as-is. Either submit an SEC ticker / canonical name, or add the sponsor to your custom alias list (open a feature request).

**Q: My pipeline mode showed `drugApprovalCount: 0` for a company that obviously has approvals.**
A: OpenFDA tracks legal entities by sponsor_name. Subsidiaries (e.g. ModernaTX, Inc.) may file under names that don't match the parent (Moderna Inc). Try passing the subsidiary string directly: `sponsorIdentifiers: ["ModernaTX, Inc."]`. Future v2 will widen the alias index.

**Q: How fresh is the data?**
A: ClinicalTrials.gov v2 reflects the current live database. OpenFDA datasets update on different cadences — drug approvals weekly, adverse events quarterly, recalls daily, shortages weekly. Each row's `scrapedAt` reflects when this actor fetched it; OpenFDA's last-update date is in `meta.last_updated` of the underlying API.

**Q: Can I get EU trials?**
A: Not in v1. EU CTIS uses an undocumented POST search endpoint and EUDRA-CT requires HTML scraping. Add as a feature request.

**Q: Does the AI summary hallucinate?**
A: Gemini 2.5 Flash with `thinkingBudget: 0` and `temperature: 0.2` is constrained to JSON output and only sees the structured record (no external context). It does not invent NCT IDs or sponsor names. Outcome priors are calibrated to BIO/Biomedtracker industry base rates (Phase 1→2 ~63%, Phase 2→3 ~31%, Phase 3→NDA ~58%, NDA→approval ~85%).

**Q: How do I report a bug?**
A: Open an issue on the Apify actor page or reach out via the developer's email ([omar.eldeeb@remotegrowthpartners.com](mailto:omar.eldeeb@remotegrowthpartners.com)).