# DNS Log Analysis: SOC Investigation with Splunk

Threat-hunting investigation of **427,935 Zeek DNS events** using Splunk Enterprise.
Completed as part of the Indian Cyber Club SOC Analyst hands-on project (ICC EHX 2026).

## Key finding

Internal host **192.168.202.79** showed a combination of behaviours no other host showed:

- 437 of the 440 AXFR (zone transfer) requests in the dataset, across 7 zones and 10 DNS servers
- 3,703 reverse (PTR) lookups spread evenly across nine subnets
- 2,620 unique domains queried and 2,557 unique failed names

This matches network reconnaissance. The activity is classified as
**suspicious and requires validation**, not malicious, because DNS logs cannot show intent
and an approved vulnerability scan would look identical.

## Repository contents

| Folder | Contents |
|---|---|
| `report/` | Full investigation report (PDF) |
| `spl/` | All 28 SPL searches and the `dnsf` field-extraction macro |
| `screenshots/` | Evidence figures from Splunk |
| `data/` | Dataset information (the log file is not included) |

## Investigation phases

1. **Data validation:** ingestion, timestamp handling, field discovery
2. **Baseline (Tasks 01 to 10):** top domains and clients, query types, response codes, activity over time, rare domains, NXDOMAIN, keyword hunt
3. **Advanced hunting:** volume vs unique domains, NXDOMAIN by client, query bursts, domain pivot
4. **Threat-intelligence validation:** VirusTotal and URLhaus checks, with limitations documented

## Skills demonstrated

Splunk SPL, search macros, Zeek log analysis, DNS baselining, AXFR / PTR / NXDOMAIN hunting,
threat-intelligence validation, evidence-based SOC reporting.

## Tools

Splunk Enterprise 10.6.0.5 (local lab), Zeek `dns.log`, VirusTotal, URLhaus

## Dataset

The raw log is **not included** (third-party training dataset, about 62 MB). See [`data/README.md`](data/README.md).

## Disclaimer

Authorised training dataset analysed in an isolated lab environment for learning purposes only.
