# Hash Kırma Cheatsheet

John the Ripper + hash tanıma.

## Hash Türünü Tanı

| Başlangıç | Uzunluk | Muhtemel tür |
|---|---|---|
| `$1$` | — | MD5 (Linux) |
| `$2a$` / `$2b$` | — | bcrypt |
| `$5$` | — | SHA-256 (Linux) |
| `$6$` | — | SHA-512 (Linux) |
| Düz hex | 32 char | MD5 |
| Düz hex | 40 char | SHA-1 |
| Düz hex | 64 char | SHA-256 |

```bash
# Otomatik tanıma
hash-identifier
# veya
hashid <hash>
```

## John the Ripper

```bash
# Basit — wordlist ile kır
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Format belirt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Kırılmış sonuçları göster
john --show hash.txt

# Shadow dosyasını kır
unshadow /etc/passwd /etc/shadow > unshadowed.txt
john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt
```

## Yaygın John Formatları

| Format | Kullanım |
|---|---|
| `raw-md5` | Düz MD5 hash |
| `raw-sha1` | Düz SHA-1 |
| `raw-sha256` | Düz SHA-256 |
| `bcrypt` | bcrypt ($2a$) |
| `sha512crypt` | Linux shadow ($6$) |

## Hashcat (alternatif, GPU ile hızlı)

```bash
# MD5
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt

# SHA-1
hashcat -m 100 hash.txt /usr/share/wordlists/rockyou.txt

# SHA-256
hashcat -m 1400 hash.txt /usr/share/wordlists/rockyou.txt
```

## Gerçek Kullanımım

```
# Simple CTF'te:
# SQLi ile sızan hash → john ile kırmayı denedim
# VPN paket kaybı yüzünden hash bozuk geldi
# Ders: Kötü ağda blind SQLi ile çekilen veriye güvenme
```
