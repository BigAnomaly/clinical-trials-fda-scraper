[Clinical Trials Fda Scraper](https://apify.com/constant_quadruped/clinical-trials-fda-scraper?fpr=data)

# Clinical Trials & FDA Data Scraper

Extract clinical trial data from **ClinicalTrials.gov** and FDA data from **openFDA** in one unified actor. Perfect for pharma intelligence, clinical research, regulatory monitoring, and biotech investment analysis.

## Data Sources

| Source | Data Type | Records Available |
| --- | --- | --- |
| **ClinicalTrials.gov** | Clinical trials | 400,000+ studies |
| **openFDA - FAERS** | Drug adverse events | Millions of reports |
| **openFDA - SPL** | Drug labels/prescribing info | 100,000+ labels |
| **openFDA - Enforcement** | Drug & device recalls | All FDA recalls |
| **openFDA - Drugs@FDA** | Drug approvals | 26,000+ drugs |
| **openFDA - MAUDE** | Device adverse events | Millions of reports |
| **openFDA - 510(k)** | Device clearances | 200,000+ clearances |

## Features

- **Multi-source search**: Query clinical trials and FDA data in a single run
- **Unified schema**: Consistent output format across all data types
- **AI summaries** (optional): GPT-powered trial analysis and adverse event pattern detection (falls back to rule-based summaries when no API key provided)
- **No authentication required**: All APIs are public and free
- **Rate-limit compliant**: Built-in throttling respects API limits
- **Public domain data**: All data is CC0/public domain, legal to use commercially

## Use Cases

- **Pharma competitive intelligence**: Monitor competitor clinical trials and FDA submissions
- **Clinical researchers**: Find trials for systematic reviews, identify research gaps
- **Biotech investors**: Track drug pipeline progress, FDA approval timelines
- **Regulatory teams**: Monitor recalls, adverse events, compliance issues
- **Market access teams**: Evidence generation, investigator discovery

## Example Inputs

### Search clinical trials for a condition

```
{
  "dataSources": ["clinicaltrials"],
  "condition": "breast cancer",
  "phase": ["PHASE3"],
  "status": ["RECRUITING"],
  "maxResults": 100
}
```

### Get FDA data for a specific drug

```
{
  "dataSources": ["openfda"],
  "drugName": "Keytruda",
  "includeAdverseEvents": true,
  "includeDrugLabels": true,
  "includeRecalls": true,
  "maxResults": 500
}
```

### Combined search for a drug

```
{
  "dataSources": ["clinicaltrials", "openfda"],
  "intervention": "pembrolizumab",
  "drugName": "Keytruda",
  "genericName": "pembrolizumab",
  "phase": ["PHASE2", "PHASE3"],
  "maxResults": 200
}
```

### Search for device recalls

```
{
  "dataSources": ["openfda"],
  "deviceName": "pacemaker",
  "recallClass": "I",
  "includeRecalls": true,
  "includeDevice510k": true,
  "dateFrom": "2023-01-01"
}
```

## Input Parameters

### Data Source Selection

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `dataSources` | array | `["clinicaltrials"]` | Data sources to query: `"clinicaltrials"`, `"openfda"` |

### ClinicalTrials.gov Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| `condition` | string | Disease or condition to search |
| `intervention` | string | Drug, device, or other intervention |
| `searchTerm` | string | General search term |
| `sponsor` | string | Sponsor name |
| `location` | string | Geographic location |
| `status` | array | Trial statuses: `RECRUITING`, `COMPLETED`, `ACTIVE_NOT_RECRUITING`, etc. |
| `phase` | array | Trial phases: `PHASE1`, `PHASE2`, `PHASE3`, `PHASE4` |
| `studyType` | string | Study type: `INTERVENTIONAL`, `OBSERVATIONAL` |
| `nctIds` | array | Specific NCT IDs to retrieve |

### openFDA Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| `drugName` | string | Brand name of drug |
| `genericName` | string | Generic name of drug |
| `deviceName` | string | Device name for device searches |
| `manufacturerName` | string | Manufacturer name |
| `recallClass` | string | Recall classification: `I`, `II`, `III` |
| `productDescription` | string | Product description text |

### Shared Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `dateFrom` | string | - | Start date (YYYY-MM-DD) |
| `dateTo` | string | - | End date (YYYY-MM-DD) |
| `maxResults` | number | `100` | Maximum records to return (openFDA adverse events limited to 25,000) |

### Feature Flags

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `includeAISummary` | boolean | `true` | Generate AI summaries (requires `openaiApiKey`) |
| `includeAdverseEvents` | boolean | `true` | Include FAERS/MAUDE adverse events |
| `includeDrugLabels` | boolean | `true` | Include SPL drug labels |
| `includeRecalls` | boolean | `true` | Include enforcement/recalls |
| `includeDrugApprovals` | boolean | `true` | Include Drugs@FDA approvals |
| `includeDevice510k` | boolean | `true` | Include 510(k) clearances |

### API Keys (Optional)

| Parameter | Type | Description |
| --- | --- | --- |
| `openfdaApiKey` | string | openFDA API key for higher rate limits (240 vs 40 req/min). Get free at open.fda.gov |
| `openaiApiKey` | string | OpenAI API key for AI-powered summaries. Without this, rule-based summaries are generated |

## Output Schema

Each record includes a `dataType` field indicating the data source:

- `clinical_trial` - ClinicalTrials.gov study
- `drug_adverse_event` - FAERS adverse event report
- `drug_label` - SPL drug label
- `drug_recall` - Drug enforcement/recall
- `drug_approval` - Drugs@FDA approval
- `device_adverse_event` - MAUDE device event
- `device_recall` - Device enforcement/recall
- `device_510k` - 510(k) clearance
- `adverse_event_summary` - AI-generated summary (if enabled)

### Clinical Trial Fields

| Field | Description |
| --- | --- |
| `nctId` | NCT identifier |
| `briefTitle` | Study title |
| `overallStatus` | Current status (RECRUITING, COMPLETED, etc.) |
| `phases` | Trial phases |
| `conditions` | Target conditions/diseases |
| `interventions` | Treatments being studied |
| `leadSponsor` | Sponsor name and class |
| `enrollmentCount` | Target enrollment |
| `locations` | Study sites with geo coordinates |
| `primaryOutcomes` | Primary outcome measures |
| `eligibilityCriteria` | Inclusion/exclusion criteria |
| `url` | Direct link to trial |

### FDA Adverse Event Fields

| Field | Description |
| --- | --- |
| `safetyReportId` | FAERS report ID |
| `serious` | Whether event was serious |
| `drugs` | Drugs involved with characterization |
| `reactions` | Reported adverse reactions |
| `patientAge`, `patientSex` | Patient demographics |
| `seriousnessDeath` | Fatal outcome flag |

## API Rate Limits

| API | Rate Limit | Notes |
| --- | --- | --- |
| ClinicalTrials.gov | ~50 req/min | No API key needed |
| openFDA (no key) | 40 req/min | Free, limited |
| openFDA (with key) | 240 req/min | Free key at open.fda.gov |

The actor automatically respects these limits with built-in throttling.

## Pricing

This actor uses standard Apify compute units. A typical run:

- 100 clinical trials: ~0.05 CU
- 1,000 adverse events: ~0.1 CU
- Full drug profile (all data types): ~0.2 CU

## Key-Value Store Schema

The actor stores the following data in the key-value store:

### `RUN_STATS`

```
{
  "clinicalTrials": 100,
  "drugAdverseEvents": 500,
  "drugLabels": 5,
  "drugRecalls": 2,
  "drugApprovals": 1,
  "deviceAdverseEvents": 0,
  "deviceRecalls": 0,
  "device510k": 0
}
```

## Legal & Data License

- **ClinicalTrials.gov**: U.S. government work, no restrictions on use
- **openFDA**: CC0 1.0 Universal (Public Domain Dedication)
- All data is legal to use, modify, and distribute commercially

## Support

For issues or feature requests, please contact the author through the Apify platform.

---

Built by JCD | Data from ClinicalTrials.gov & openFDA