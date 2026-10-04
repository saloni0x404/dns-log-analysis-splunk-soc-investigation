# SPL Searches Used

All searches use `index=dns_lab`, `sourcetype=dns_sample` and time range **All time**.
The macro `dnsf` extracts the DNS fields (see [dnsf_macro.md](dnsf_macro.md)).
In the Zeek log, `-` stands for "no value".

## SPL 1: Validation

```spl
index=dns_lab sourcetype=dns_sample | head 20
```

## SPL 2: Field summary

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | fieldsummary | sort - count
```

## SPL 3: DNS fields only

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | fieldsummary | search field IN ("src_ip","dest_ip","fqdn","query_type","response_code","src_port","dest_port","proto","uid","ts") | table field count distinct_count | sort - count
```

## SPL 4: Task 01, event count and time range

```spl
index=dns_lab sourcetype=dns_sample | stats count min(_time) as first max(_time) as last | convert ctime(first) ctime(last)
```

## SPL 5: Task 02, top domains

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | top limit=20 fqdn
```

## SPL 6: Task 03, top clients

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | top limit=20 src_ip
```

## SPL 7: Profile of 10.10.117.210

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=10.10.117.210 | stats count dc(fqdn) as unique_domains min(_time) as first max(_time) as last | convert ctime(first) ctime(last)
```

## SPL 8: Domains/types/responses of 10.10.117.210

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=10.10.117.210 | stats count by fqdn query_type response_code | sort - count | head 15
```

## SPL 9: Task 04, query types

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | stats count by query_type | sort - count
```

## SPL 10: Task 05, response codes

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | stats count by response_code | sort - count
```

## SPL 11: Task 06, activity over time

```spl
index=dns_lab sourcetype=dns_sample | timechart span=1h count
```

## SPL 12: Task 07, client/domain pairs

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | stats count by src_ip fqdn | sort - count | head 30
```

## SPL 13: Task 08, rare domains

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | rare limit=30 fqdn
```

## SPL 14: Task 09, NXDOMAIN pairs

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search response_code=NXDOMAIN | stats count by src_ip fqdn | sort - count | head 30
```

## SPL 15: Task 10, keyword hunt

```spl
index=dns_lab sourcetype=dns_sample | regex _raw="(?i)(login|verify|secure|update|account|password|wallet)" | `dnsf` | stats count by fqdn src_ip | sort - count
```

## SPL 16: Phase 3A, volume vs unique domains

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | stats count as dns_queries dc(fqdn) as unique_domains by src_ip | sort - dns_queries | head 20
```

## SPL 17: Phase 3B, domains by distinct clients

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | stats dc(src_ip) as unique_clients count as queries by fqdn | sort - unique_clients | head 30
```

## SPL 18: Phase 3C, NXDOMAIN by client

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search response_code=NXDOMAIN | stats count as nxdomain_count dc(fqdn) as unique_failed_domains by src_ip | sort - nxdomain_count
```

## SPL 19: Phase 3D, query bursts

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | bin _time span=5m | stats count as queries by _time src_ip | sort - queries | head 50
```

## SPL 20: Phase 3E, domain pivot

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search fqdn="rootshell-security.net" | stats count values(src_ip) as clients values(query_type) as query_types by fqdn
```

## SPL 21: AXFR requests by client

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search query_type=AXFR | stats count dc(fqdn) as zones dc(dest_ip) as dns_servers min(_time) as first max(_time) as last by src_ip | convert ctime(first) ctime(last)
```

## SPL 22: AXFR zones and servers

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=192.168.202.79 query_type=AXFR | stats count by fqdn dest_ip | sort - count
```

## SPL 23: AXFR responses with answers

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=192.168.202.79 query_type=AXFR response_code=NOERROR | rex field=_raw "\t(?<answers>[^\t]+)\t(?<TTLs>[^\t]+)\t(?<rejected>[^\t]+)$" | table _time fqdn dest_ip answers | head 10
```

## SPL 24: Raw AXFR events

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=192.168.202.79 query_type=AXFR | head 3 | table _time _raw
```

## SPL 25: PTR sweep by subnet

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=192.168.202.79 query_type=PTR | rex field=fqdn "^\d+\.(?<third_octet>\d+)\.168\.192\.in-addr\.arpa" | stats count by third_octet | sort - count
```

## SPL 26: Query types/responses for 192.168.202.79

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search src_ip=192.168.202.79 | stats count by query_type response_code | sort - count
```

## SPL 27: version.bind queries

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search fqdn="version.bind" | stats count dc(dest_ip) as dns_servers by src_ip | sort - count
```

## SPL 28: DNSBL lookups

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | search fqdn IN ("*sorbs.net","*spamhaus.org","*spamcop.net","*spamrats.com","*quorum.to","*tornevall.org","*apews.org","*nszones.com") | stats count dc(fqdn) as lookups by src_ip | sort - count
```

