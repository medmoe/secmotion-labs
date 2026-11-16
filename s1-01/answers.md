# Worked answers

## 1. Shared passwords

| Rows | Accounts | How you can tell |
|---|---|---|
| 1, 3 | nora, priya | identical SHA-256 hashes: no salt, so the same password always gives the same hash |
| 4, 6 | quinn, sam | identical scrypt hashes, because both rows use the **same salt** |
| 5 | rosa | also has quinn's password, and **nobody can see it**: her own salt makes her hash different. That is what a salt is for. (We know because we made the data.) |

## 2. Stored safely?

| Row | Verdict | Why |
|---|---|---|
| 1–3 | not safe | plain SHA-256 is fast (1,554,687 guesses a second on one core, 21.2 billion on one GPU) and unsalted, so equal passwords show up as equal hashes and one guess tests every row at once |
| 4, 6 | slow hash, broken salt | scrypt is right, but the salt is reused: the two rows leak that they match. Give every user a fresh random salt (`secrets.token_bytes(16)`) |
| 5 | safe | scrypt + its own random salt: the way the video recommends |

## 3. Five random words

| Step | Arithmetic | Result |
|---|---|---|
| all passphrases | 7,776⁵ | 2.84 × 10¹⁹ |
| on average, half | 2.84 × 10¹⁹ ÷ 2 | 1.42 × 10¹⁹ guesses |
| plain SHA-256, one core | 1.42 × 10¹⁹ ÷ 1,554,687 / s | 290 thousand years |
| scrypt, one core | 1.42 × 10¹⁹ ÷ 24.5 / s | 18 billion years: longer than the universe has existed (13.8 billion) |

Even with plain SHA-256 on one RTX 4090 it is 21 years (the video's number). The words do most of the work; scrypt multiplies it again, about 63,457 times.
