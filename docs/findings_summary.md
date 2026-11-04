# Findings Summary

All times UTC. Figure numbers refer to the files in [`../screenshots`](../screenshots) and to the report.

## Dataset
427,935 DNS events, 253 clients, 16 Mar 2012 12:30:05 to 17 Mar 2012 20:59:51.

## Baseline
- Top domains are routine services: `teredo.ipv6.microsoft.com` (39,273), `tools.google.com`, `www.apple.com`, `time.apple.com`, `safebrowsing.clients.google.com`.
- Top query types: A 212,477; NB 77,199; AAAA 54,519; PTR 53,017. Uncommon: **AXFR 440**.
- Response codes: `-` 372,201 (no reply logged); NXDOMAIN 50,330; NOERROR 2,665; REFUSED 1,797.
- Activity peaks 16 Mar 16:00 to 19:00 (73,300 events at 18:00), driven by `10.10.117.210`. No events from about 23:00 on 16 Mar to 12:00 on 17 Mar.

## Primary case: 192.168.202.79

| Indicator | Value | Figure |
|---|---|---|
| AXFR requests | 437 of 440 (7 zones, 10 DNS servers) | 22, 23 |
| AXFR answered NOERROR with internal records | 415 (16 no reply, 4 NOTAUTH, 2 REFUSED) | 24, 25 |
| PTR lookups | 3,703 across subnets 21 to 28 and 229 | 26 |
| Unique domains / unique failed names | 2,620 / 2,557 | 17, 19 |
| Blocklist (DNSBL) lookups | 84 events, 77 distinct | 29 |
| AXFR period | 16 Mar 15:57 to 17 Mar 15:50 | 22 |

## Related hosts
`192.168.202.102` (515 `version.bind` queries to 95 servers), `.115` (84 to 76), `.108` (240 DNSBL events), `.138` and `.140` (about 2,200 unique failed names each).

## Benign explanations considered
- `10.10.117.210`: 250 unique domains, no failures; gateway, proxy or automated software.
- `192.168.202.103` and `.83`: thousands of NXDOMAIN on only 62 and 18 names (retries).
- An approved vulnerability scan or penetration test would reproduce the primary pattern.

## Threat intelligence (4 Oct 2026)
`rootshell-security.net`: 770 queries, 7 clients, all NXDOMAIN. VirusTotal 4 of 91 vendors; URLhaus no entry. **Inconclusive** (domain registered about 2 years ago; the log is from 2012).

## Assessment
**Suspicious and requires validation.** Next actions: identify the host owner and confirm authorisation; check firewall/`conn.log` and DNS-server logs for what was transferred; escalate if unapproved and restrict AXFR to approved secondary servers.
