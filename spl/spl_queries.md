# SPL Searches

All searches use index `dns_lab`, sourcetype `dns_sample` and time range **All time**.
The macro `dnsf` extracts the DNS fields (see [dnsf_macro.txt](dnsf_macro.txt)).
Splunk displays time in the user time zone (IST in this lab); the data itself is UTC.

| # | Purpose | Report figure |
|---|---|---|
| SPL 1 | Validation | Figure 1 |
| SPL 2 | Field summary | Figure 2 |
| SPL 3 | DNS fields only | Figure 3 |
| SPL 4 | Task 01, event count and time range | Figure 4 |
| SPL 5 | Task 02, top domains | Figure 5 |
| SPL 6 | Task 03, top clients | Figure 6 |
| SPL 7 | Profile of 10.10.117.210 | Figure 7 |
| SPL 8 | Domains, types and responses of 10.10.117.210 | Figure 8 |
| SPL 9 | Task 04, query types | Figure 9 |
| SPL 10 | Task 05, response codes | Figure 10 |
| SPL 11 | Task 06, activity over time | Figures 11-12 |
| SPL 12 | Task 07, client/domain pairs | Figure 13 |
| SPL 13 | Task 08, rare domains | Figure 14 |
| SPL 14 | Task 09, NXDOMAIN pairs | Figure 15 |
| SPL 15 | Task 10, keyword hunt | Figure 16 |
| SPL 16 | Phase 3A, volume vs unique domains | Figure 17 |
| SPL 17 | Phase 3B, domains by distinct clients | Figure 18 |
| SPL 18 | Phase 3C, NXDOMAIN by client | Figure 19 |
| SPL 19 | Phase 3D, query bursts | Figure 20 |
| SPL 20 | Phase 3E, domain pivot (rootshell-security.net) | Figure 21 |
| SPL 21 | AXFR requests by client | Figure 22 |
| SPL 22 | AXFR zones and servers | Figure 23 |
| SPL 23 | AXFR responses with answer data | Figure 24 |
| SPL 24 | Raw AXFR events | Figure 25 |
| SPL 25 | PTR sweep by subnet | Figure 26 |
| SPL 26 | Query types and responses for 192.168.202.79 | Figure 27 |
| SPL 27 | version.bind queries | Figure 28 |
| SPL 28 | DNSBL (blocklist) lookups | Figure 29 |

---

## SPL 1: Validation

*Evidence: Figure 1*

```spl
index=dns_lab sourcetype=dns_sample | head 20
```

## SPL 2: Field summary

*Evidence: Figure 2*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | fieldsummary | sort - count
```

## SPL 3: DNS fields only

*Evidence: Figure 3*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | fieldsummary | search field IN ("src_ip","dest_ip","fqdn","query_type","response_code","src_port","dest_port","proto","uid","ts") | table field count distinct_count | sort - count
```

## SPL 4: Task 01, event count and time range

*Evidence: Figure 4*

```spl
index=dns_lab sourcetype=dns_sample | stats count min(_time) as first max(_time) as last | convert ctime(first) ctime(last)
```

## SPL 5: Task 02, top domains

*Evidence: Figure 5*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | top limit=20 fqdn
```

## SPL 6: Task 03, top clients

*Evidence: Figure 6*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | top limit=20 src_ip
```

## SPL 7: Profile of 10.10.117.210

*Evidence: Figure 7*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=10.10.117.210 | stats count dc(fqdn) as unique_domains min(_time) as first max(_time) as last | convert ctime(first) ctime(last)
```

## SPL 8: Domains, types and responses of 10.10.117.210

*Evidence: Figure 8*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=10.10.117.210 | stats count by fqdn query_type response_code | sort - count | head 15
```

## SPL 9: Task 04, query types

*Evidence: Figure 9*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | stats count by query_type | sort - count
```

## SPL 10: Task 05, response codes

*Evidence: Figure 10*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | stats count by response_code | sort - count
```

## SPL 11: Task 06, activity over time

*Evidence: Figures 11-12*

```spl
index=dns_lab sourcetype=dns_sample | timechart span=1h count
```

## SPL 12: Task 07, client/domain pairs

*Evidence: Figure 13*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | stats count by src_ip fqdn | sort - count | head 30
```

## SPL 13: Task 08, rare domains

*Evidence: Figure 14*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | rare limit=30 fqdn
```

## SPL 14: Task 09, NXDOMAIN pairs

*Evidence: Figure 15*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search response_code=NXDOMAIN | stats count by src_ip fqdn | sort - count | head 30
```

## SPL 15: Task 10, keyword hunt

*Evidence: Figure 16*

```spl
index=dns_lab sourcetype=dns_sample | regex _raw="(?i)(login|verify|secure|update|account|password|wallet)" | `dnsf` | stats count by fqdn src_ip | sort - count
```

## SPL 16: Phase 3A, volume vs unique domains

*Evidence: Figure 17*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | stats count as dns_queries dc(fqdn) as unique_domains by src_ip | sort - dns_queries | head 20
```

## SPL 17: Phase 3B, domains by distinct clients

*Evidence: Figure 18*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | stats dc(src_ip) as unique_clients count as queries by fqdn | sort - unique_clients | head 30
```

## SPL 18: Phase 3C, NXDOMAIN by client

*Evidence: Figure 19*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search response_code=NXDOMAIN | stats count as nxdomain_count dc(fqdn) as unique_failed_domains by src_ip | sort - nxdomain_count
```

## SPL 19: Phase 3D, query bursts

*Evidence: Figure 20*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | bin _time span=5m | stats count as queries by _time src_ip | sort - queries | head 50
```

## SPL 20: Phase 3E, domain pivot (rootshell-security.net)

*Evidence: Figure 21*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search fqdn="rootshell-security.net" | stats count values(src_ip) as clients values(query_type) as query_types by fqdn
```

## SPL 21: AXFR requests by client

*Evidence: Figure 22*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search query_type=AXFR | stats count dc(fqdn) as zones dc(dest_ip) as dns_servers min(_time) as first max(_time) as last by src_ip | convert ctime(first) ctime(last)
```

## SPL 22: AXFR zones and servers

*Evidence: Figure 23*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=192.168.202.79 query_type=AXFR | stats count by fqdn dest_ip | sort - count
```

## SPL 23: AXFR responses with answer data

*Evidence: Figure 24*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=192.168.202.79 query_type=AXFR response_code=NOERROR | rex field=_raw "\t(?<answers>[^\t]+)\t(?<TTLs>[^\t]+)\t(?<rejected>[^\t]+)$" | table _time fqdn dest_ip answers | head 10
```

## SPL 24: Raw AXFR events

*Evidence: Figure 25*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=192.168.202.79 query_type=AXFR | head 3 | table _time _raw
```

## SPL 25: PTR sweep by subnet

*Evidence: Figure 26*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=192.168.202.79 query_type=PTR | rex field=fqdn "^\d+\.(?<third_octet>\d+)\.168\.192\.in-addr\.arpa" | stats count by third_octet | sort - count
```

## SPL 26: Query types and responses for 192.168.202.79

*Evidence: Figure 27*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=192.168.202.79 | stats count by query_type response_code | sort - count
```

## SPL 27: version.bind queries

*Evidence: Figure 28*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search fqdn="version.bind" | stats count dc(dest_ip) as dns_servers by src_ip | sort - count
```

## SPL 28: DNSBL (blocklist) lookups

*Evidence: Figure 29*

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search fqdn IN ("*sorbs.net","*spamhaus.org","*spamcop.net","*spamrats.com","*quorum.to","*tornevall.org","*apews.org","*nszones.com") | stats count dc(fqdn) as lookups by src_ip | sort - count
```

