# Dataset information

The raw dataset is **not included** in this repository because it is a third-party training
dataset and is large (about 62 MB).

| Item | Detail |
|---|---|
| Type | Zeek `dns.log` (tab-separated, no header row) |
| Events | 427,935 |
| Time range (UTC) | 16 Mar 2012 12:30:05 to 17 Mar 2012 20:59:51 |
| Provided by | Indian Cyber Club, ICC EHX 2026 batch (SOC Analyst project) |
| Splunk index / sourcetype | `dns_lab` / `dns_sample` |

To reproduce the analysis, obtain the dataset from the course, upload it with
**Settings > Add Data > Upload**, apply the sourcetype settings in
[`spl/dnsf_macro.md`](../spl/dnsf_macro.md), then run the searches in
[`spl/spl_searches.md`](../spl/spl_searches.md).
