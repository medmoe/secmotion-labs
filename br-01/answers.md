# Worked answers

## 1. Each server against the advisory

| Row | Server | OpenSSL | Status | Why |
|---|---|---|---|---|
| 1 | `shop.bytebakery.example` | 1.0.1f | vulnerable: patch now | 1.0.1 up to 1.0.1f is the affected range. |
| 2 | `mail.bytebakery.example` | 0.9.8y | never affected | the heartbeat code arrived in 1.0.1; this older line never had it. |
| 3 | `supplier.bytebakery.example` | 1.0.1g | fixed version | 1.0.1g is the fix. |
| 4 | `stock.bytebakery.example` | 1.0.1 | vulnerable: patch now | 1.0.1 up to 1.0.1f is the affected range. |
| 5 | `ovens.bytebakery.example` | 1.0.0l | never affected | the heartbeat code arrived in 1.0.1; this older line never had it. |
| 6 | `staging.bytebakery.example` | 1.0.1c | vulnerable: patch now | 1.0.1 up to 1.0.1f is the affected range. |
| 7 | `pay.bytebakery.example` | 1.0.1g | fixed version | 1.0.1g is the fix. |

## 2. The check, line by line

| Line | record_length | payload_length | 1.0.1g | The arithmetic |
|---|---|---|---|---|
| 1 | 24 | 5 | echo | 5 + 19 = 24 ≤ 24: it fits, so 1.0.1g echoes it. |
| 2 | 24 | 500 | discard | 500 + 19 = 519 > 24: discarded. The unpatched server sent back 500 bytes from a 24-byte record: 479 of them server memory past the record's end. The video's hook: the 495 bytes after “bread” were her own 16 bytes of padding, then 479 bytes of other people's data. |
| 3 | 83 | 64 | echo | 64 + 19 = 83 ≤ 83: it fits, so 1.0.1g echoes it. |
| 4 | 24 | 65,535 | discard | 65,535 + 19 = 65,554 > 24: discarded. The unpatched server sent back 65,535 bytes from a 24-byte record: 65,514 of them server memory past the record's end. |
| 5 | 51 | 16 | echo | 16 + 19 = 35 ≤ 51: it fits, so 1.0.1g echoes it. The record has more than 16 bytes of padding, which RFC 6520 allows: the check only asks that the claim fits. |
| 6 | 20 | 16,384 | discard | 16,384 + 19 = 16,403 > 20: discarded. The unpatched server sent back 16,384 bytes from a 20-byte record: 16,367 of them server memory past the record's end. |
| 7 | 37 | 18 | echo | 18 + 19 = 37 ≤ 37: it fits, so 1.0.1g echoes it. |
| 8 | 39 | 21 | discard | 21 + 19 = 40 > 39: discarded. Only one byte over, so the unpatched reply took a byte of the sender's own padding and nothing else; the check doesn't guess intent, any claim that doesn't fit is thrown away. |

Lines 2, 4, 6 and 8 are discarded. Together the lies took 82,360 bytes of staging's memory. Normal logs showed none of this; only a debug trace did.

## 3. New key and certificate

| Row | Server | New key? | Why |
|---|---|---|---|
| 1 | `shop.bytebakery.example` | yes | it ran 1.0.1f with a 2013 key: patch, then a new key and certificate, then reset customer sessions and ask for new passwords. |
| 2 | `mail.bytebakery.example` | no | 0.9.8 never had the bug: nothing leaked, nothing to replace. |
| 3 | `supplier.bytebakery.example` | yes | patched on 8 Apr, but its key sat for two years in the memory of a vulnerable 1.0.1e. Patching doesn't take back a key that may already have leaked. |
| 4 | `stock.bytebakery.example` | yes | office network only, but anyone on it (a guest's laptop, a compromised PC) could send a heartbeat. Vulnerable since 2012: new key and certificate too. |
| 5 | `ovens.bytebakery.example` | no | 1.0.0 never had the bug: nothing to replace. |
| 6 | `staging.bytebakery.example` | yes | the trace shows lying heartbeats reached it: assume its key and the test accounts leaked. New key and certificate, and never share keys between staging and the shop. |
| 7 | `pay.bytebakery.example` | no | its key was born after the fix, on a version that never leaked it. |

The order that matters: patch first, then the new key and certificate (a new key on an unpatched server can leak again), then revoke the old certificate and reset sessions and passwords.
