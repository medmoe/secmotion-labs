# Worked answers

## 1. Symmetric, asymmetric or both

| Row | System | Kind | The clue |
|---|---|---|---|
| 1 | online shop (shop.bytebakery.example) | both (hybrid) | “public-key handshake … then AES” |
| 2 | nightly backup of the orders database | asymmetric | “RSA-2048-OAEP … public key” |
| 3 | customers' cake-design photos, stored | symmetric | “AES-128”, one key |
| 4 | staff chat (40 people) | symmetric | “one AES-128 key shared” |
| 5 | recipe updates sent to franchise ovens | asymmetric | “private key … public key” (a signature) |
| 6 | supplier portal (supplier.bytebakery.example) | both (hybrid) | “session key … wrapped with … RSA key, then AES” |
| 7 | the manager's laptop | symmetric | “AES-128”, one person |

## 2. The four mistakes

| Row | What goes wrong, and the fix |
|---|---|
| 2 | RSA-OAEP takes at most 190 bytes per operation and is slow (question 3). Use hybrid: AES-128-GCM for the file, the owner's RSA key wraps only the 16-byte AES key |
| 3 | ECB encrypts equal blocks to equal blocks, so a picture's outline shows through (the cupcake logo). Same key, but GCM mode: noise, and tampering is detected |
| 4 | everyone can read every chat, and anyone who leaves keeps the key for months. Give each person a key pair; public keys set up a fresh AES key per conversation (both) |
| 6 | no forward secrecy: a recording of today's traffic plus a key that leaks next year opens every old session. Move to TLS 1.3, which removed RSA key exchange |

Rows 1, 5 and 7 are fine: 1: the hybrid the video ends on, with a fresh key every session; 5: a signature: it hides nothing, it proves the bakery sent it. Only the private key can make one, any oven can check it; 7: one person, so no key to share: symmetric is the right tool.

## 3. The arithmetic

| Question | Arithmetic | Result |
|---|---|---|
| (a) a shared key per pair | 40 × 39 ÷ 2 | 780 keys |
| (a) a key pair each | 2 × 40 | 80 keys (9.8× fewer) |
| (b) RSA operations | 50,000,000 ÷ 190, rounded up | 263,158 |
| (b) RSA restore time | 263,158 × 991 µs | ≈ 261 s, about 4 min 21 s |
| (b) encrypted size | 263,158 × 256 bytes | 67.4 MB: 35% bigger |
| (b) AES instead | 3,125,000 blocks × 8 ns | ≈ 25 ms, same size as the file |

Hybrid gets the best of both: one RSA operation (≈ 1 ms) to unwrap the AES key, then AES for the 50 MB. That is how HTTPS works, and how an encrypted backup should.
