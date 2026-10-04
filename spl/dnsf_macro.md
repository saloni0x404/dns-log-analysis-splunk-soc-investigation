# Field-Extraction Macro `dnsf`

Create it in **Settings > Advanced search > Search macros**:

- **Name:** `dnsf`
- **Definition** (no leading pipe):

```spl
rex "^(?<ts>[^\t]+)\t(?<uid>[^\t]+)\t(?<src_ip>[^\t]+)\t(?<src_port>[^\t]+)\t(?<dest_ip>[^\t]+)\t(?<dest_port>[^\t]+)\t(?<proto>[^\t]+)\t(?<trans_id>[^\t]+)\t(?<fqdn>[^\t]+)\t(?<qclass>[^\t]+)\t(?<qclass_name>[^\t]+)\t(?<qtype>[^\t]+)\t(?<query_type>[^\t]+)\t(?<rcode>[^\t]+)\t(?<response_code>[^\t]+)"
```

Zeek `dns.log` has no header row, so fields are named following the Zeek DNS log layout.
Fields after `response_code` (AA, TC, RD, RA, Z, answers, TTLs, rejected) are not extracted
by the macro. `answers` is extracted inside SPL 23 where needed.

## Sourcetype settings (`dns_sample`)

| Setting | Value | Purpose |
|---|---|---|
| TIME_PREFIX | `^` | Timestamp is at the start of each line |
| TIME_FORMAT | `%s.%6N` | Epoch seconds with microseconds |
| MAX_TIMESTAMP_LOOKAHEAD | `17` | Length of the epoch timestamp |
| TZ | `UTC` | Zeek logs epoch time (UTC) |
| MAX_DAYS_AGO | `10951` | Accept 2012 timestamps |
| DATETIME_CONFIG | `/etc/datetime.xml` | Enable timestamp detection |
| SHOULD_LINEMERGE | `false` | One event per line |

## Time conversion (Splunk shows IST, UTC + 5:30)

| Splunk displays (IST) | UTC | Example |
|---|---|---|
| 03/16/2012 18:00:05 | 16 Mar 2012 12:30:05 | First event |
| 03/17/2012 21:20:28 | 17 Mar 2012 15:50:28 | Last AXFR request |
| 03/18/2012 02:29:51 | 17 Mar 2012 20:59:51 | Last event |
