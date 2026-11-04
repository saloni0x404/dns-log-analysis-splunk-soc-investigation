# Splunk Ingestion Guide (Zeek dns.log)

The Zeek `dns.log` has **no header row** and uses **epoch timestamps** from 2012. Both facts need handling at ingestion.

## 1. Upload

1. **Settings → Add Data → Upload**, select `dns.log`, click **Next**.
2. On **Set Source Type**, keep the plain `default` type. Do not use a structured `tsv` type, which treats line 1 as a header.
3. Open **Advanced** and add these settings, then click **Apply settings**:

| Setting | Value | Why |
|---|---|---|
| `TIME_PREFIX` | `^` | Timestamp is at the start of the line |
| `TIME_FORMAT` | `%s.%6N` | Epoch seconds with microseconds |
| `MAX_TIMESTAMP_LOOKAHEAD` | `17` | Length of the epoch timestamp |
| `TZ` | `UTC` | Zeek logs epoch time |
| `MAX_DAYS_AGO` | `10951` | **Required.** 2012 events are older than Splunk's default limit; without this they are stamped with today's date |
| `DATETIME_CONFIG` | `/etc/datetime.xml` | Enable timestamp detection |
| `SHOULD_LINEMERGE` | `false` | One event per line |

4. The preview **Time** column must show **3/16/12**, not today's date. Do not continue until it does.
5. **Save As** `dns_sample`, choose index `dns_lab`, then review and submit.

## 2. Validate

Set the time range to **All time**:

```spl
index=dns_lab sourcetype=dns_sample | stats count min(_time) as first max(_time) as last | convert ctime(first) ctime(last)
```

Expected: **427,935** events, from 2012-03-16 12:30:05 to 2012-03-17 20:59:51 UTC (shown 5 h 30 min later if your Splunk user is in IST).

## 3. Create the field-extraction macro

**Settings → Advanced search → Search macros → New**: name `dnsf`, app `search`, definition = the contents of [`../spl/dnsf_macro.txt`](../spl/dnsf_macro.txt). The definition starts with `rex` and has **no leading pipe**.

Use it like this:

```spl
index=dns_lab sourcetype=dns_sample | `dnsf` | top limit=20 src_ip
```

The top client should be `10.10.117.210` with 75,943 events.

## Field layout (Zeek DNS log)

`ts, uid, src_ip, src_port, dest_ip, dest_port, proto, trans_id, fqdn, qclass, qclass_name, qtype, query_type, rcode, response_code, AA, TC, RD, RA, Z, answers, TTLs, rejected`

A `-` in `response_code` means Zeek logged the query but no matching reply.
