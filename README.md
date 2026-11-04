# DNS Log Analysis with Splunk: SOC Investigation

![Project](https://img.shields.io/badge/Project-SOC%20Investigation-173f5f) ![Platform](https://img.shields.io/badge/Platform-Splunk-2b6f56) ![Focus](https://img.shields.io/badge/Focus-DNS%20Threat%20Hunting-6b4c9a)

A hands-on SOC analyst (L1) project completed for the **Indian Cyber Club** programme. I investigated **427,935 DNS events** (Zeek `dns.log`, 16 to 17 March 2012) in **Splunk Enterprise**, built a baseline, hunted for anomalies and wrote an evidence-based investigation report.

> **Final assessment: suspicious and requires validation.** Internal host `192.168.202.79` shows behaviour consistent with DNS reconnaissance. Intent cannot be proven from DNS logs alone, so the activity is **not** classified as malicious.

**Full report:** [`report/DNS_Log_Analysis_SOC_Investigation_Report.pdf`](report/DNS_Log_Analysis_SOC_Investigation_Report.pdf) (40 pages, 30 figures, 28 SPL searches)

---

## Key findings

| # | Finding | Evidence |
|---|---|---|
| 1 | `192.168.202.79` sent **437 of 440 zone-transfer (AXFR) requests** to 10 DNS servers covering 7 `hec.net` zones. **415 received NOERROR** with internal hostnames and IPs in the answers. | [fig22](screenshots/fig22_axfr_by_client.png), [fig24](screenshots/fig24_axfr_answers.png), [fig25](screenshots/fig25_axfr_raw_events.png) |
| 2 | The same host sent **3,703 reverse (PTR) lookups** across nine subnets and queried **2,620 unique domains** with **2,557 unique failures**. | [fig26](screenshots/fig26_ptr_sweep_by_subnet.png), [fig19](screenshots/fig19_phase3c_nxdomain_by_client.png) |
| 3 | Spam-blocklist (DNSBL) lookups from `.79` and `.108`; `version.bind` fingerprinting from `.102` and `.115`. | [fig29](screenshots/fig29_dnsbl_lookups.png), [fig28](screenshots/fig28_version_bind.png) |
| 4 | `rootshell-security.net` requested 770 times by 7 clients, all NXDOMAIN. VirusTotal flags 4 of 91 vendors; URLhaus has no entry: **inconclusive**. | [fig21](screenshots/fig21_phase3e_rootshell_pivot.png), [fig30](screenshots/fig30_virustotal_rootshell.png) |
| 5 | Busiest client `10.10.117.210` (75,943 queries) repeats only 250 domains (Teredo, Google, Norton): **expected behaviour** pending asset confirmation. | [fig07](screenshots/fig07_client_10-10-117-210_profile.png), [fig08](screenshots/fig08_client_10-10-117-210_domains.png) |

Behaviour is *consistent with* MITRE ATT&CK reconnaissance and discovery techniques such as **T1590.002** (Gather Victim Network Information: DNS) and **T1018** (Remote System Discovery). This mapping is an assessment, not a confirmed attribution.

## What this project demonstrates

- Ingesting epoch-timestamped Zeek logs into Splunk (timestamp parsing, `MAX_DAYS_AGO`, search-time field extraction with `rex`)
- DNS baselining: top clients and domains, query types, response codes, activity timeline
- Threat hunting with SPL: volume versus unique-domain ratio, NXDOMAIN diversity, query bursts, zone-transfer and reverse-lookup sweeps
- Threat-intelligence validation (VirusTotal, URLhaus) with honest limitations
- Separating **facts from assumptions** and writing for a SOC audience

## Repository structure

```
dns-log-analysis-splunk/
├── README.md
├── report/
│   ├── DNS_Log_Analysis_SOC_Investigation_Report.pdf
│   └── DNS_Log_Analysis_SOC_Investigation_Report.docx
├── spl/
│   ├── spl_queries.md          # all 28 searches, mapped to report figures
│   └── dnsf_macro.txt          # field-extraction macro
├── screenshots/                # 30 evidence figures (fig01 to fig30, same numbering as the report)
└── docs/
    ├── splunk_ingestion_guide.md
    └── findings_summary.md
```

## How to reproduce

1. Obtain the dataset from the course (it is **not** included here; see the note below).
2. Follow [`docs/splunk_ingestion_guide.md`](docs/splunk_ingestion_guide.md) to upload it as sourcetype `dns_sample` into index `dns_lab`.
3. Create the `dnsf` macro from [`spl/dnsf_macro.txt`](spl/dnsf_macro.txt).
4. Run the searches in [`spl/spl_queries.md`](spl/spl_queries.md) with the time range set to **All time**. Splunk shows times in your local time zone; the data is UTC.

## Tools

Splunk Enterprise 10.6.0.5 · SPL · Zeek DNS logs · VirusTotal · URLhaus

## Limitations

DNS only (no connection volumes, user identities or DNS-server logs); the log is from 2012 while threat-intelligence results reflect October 2026; 372,201 events have no recorded reply; capture has a gap of about 13 hours.

## Note on data

The dataset is course training data and is not redistributed in this repository. All screenshots come from my own Splunk lab instance.

## Author

**[Replace with your name]** · SOC Analyst (trainee) · [Optional: LinkedIn] · [Optional: contact email]


## Publication checklist

Before making this repository public, personalise the report cover (name and batch),
replace any remaining draft/evidence placeholders in the DOCX, export a matching
final PDF, and update or remove the author links above. Do not present placeholder
content as completed evidence.
