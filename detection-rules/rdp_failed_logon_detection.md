Detecting Excessive Failed Logons (Possible Brute Force):

After generating the RDP brute-force activity, I wanted to build an actual Splunk detection rather than just eyeballing the raw events, since that's the difference between "I have logs" and "I have a SOC."

**Maps to:** MITRE ATT&CK T1110 (Brute Force)
**Data source:** Windows Security Event Log (Event ID 4625), forwarded via the Splunk Universal Forwarder

## The search

```spl
index=main sourcetype="WinEventLog:Security" EventCode=4625
| bin _time span=5m
| stats count by _time, host
| where count >= 5
```

Here's what each stage does:

1. `index=main sourcetype="WinEventLog:Security" EventCode=4625` — pulls every failed-logon event out of the Security log.
2. `bin _time span=5m` — groups events into 5-minute time buckets, so I can look at how many failures happened close together rather than treating every timestamp as unique.
3. `stats count by _time, host` — counts how many failed logons landed in each 5-minute bucket, per machine.
4. `where count >= 5` — filters down to only the buckets with 5 or more failures, which is a pattern much more consistent with password guessing than a real user mistyping their password once or twice.

## Validation against my own attack data

Running this against the traffic I generated in the RDP brute-force scenario, Splunk returned 16 total matching `EventCode=4625` events across a 60-minute window, which the query correctly grouped into two flagged buckets:

| Time | Host | Count |
|---|---|---|
| 2026-09-30 03:10:00 | DESKTOP-GTDV9R5 | 6 |
| 2026-09-30 03:35:00 | DESKTOP-GTDV9R5 | 10 |

Both of my test bursts (6 attempts and 10 attempts) were correctly flagged by the `>= 5` threshold, and no unrelated single-attempt logons showed up as false positives.

## Tuning notes

- The `>= 5` threshold was picked as a reasonable starting point for a lab, not tuned against real production traffic — in a real environment this would need adjusting based on legitimate user behavior (e.g. someone genuinely forgetting their password a couple of times).
- A stronger version of this detection would also look for a **successful** logon (`EventCode=4624`) shortly after a flagged burst of failures from the same account — that combination (many failures immediately followed by a success) is a much more confident "brute force succeeded" signal than failures alone. That's a natural v2 of this search.
- This only catches bursts within a single 5-minute window — a slower, more patient brute-force attempt (a few attempts every 10 minutes, say) would slip past this specific threshold and would need a longer bucket window or a different approach (e.g. tracking failures over a rolling 24-hour period per account).
