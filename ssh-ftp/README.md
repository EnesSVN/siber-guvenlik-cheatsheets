# SSH & FTP Cheatsheet

Uzak bağlantı protokolleri.

## SSH

```bash
# Temel bağlantı
ssh kullanıcı@<ip>

# Farklı port
ssh kullanıcı@<ip> -p 2222

# Anahtar (key) ile bağlantı
ssh -i id_rsa kullanıcı@<ip>

# Key dosyası izinleri (şart, yoksa reddeder)
chmod 600 id_rsa
```

### SSH Key Bulursan

```bash
# 1. İzinleri ayarla
chmod 600 id_rsa

# 2. Bağlan
ssh -i id_rsa kullanıcı@<ip>

# 3. Passphrase varsa → john ile kır
ssh2john id_rsa > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

## FTP

```bash
# Bağlan
ftp <ip>

# Anonim giriş (nmap anonymous login gösterirse)
# kullanıcı: anonymous
# şifre: (boş bırak veya herhangi email)
```

### FTP Komutları (bağlandıktan sonra)

| Komut | Ne yapar? |
|---|---|
| `ls` | Dosyaları listele |
| `cd <dizin>` | Dizin değiştir |
| `get <dosya>` | Dosya indir |
| `put <dosya>` | Dosya yükle |
| `binary` | Binary mod (resim/program için) |
| `passive` | Pasif moda geç (bağlantı sorunu varsa) |
| `bye` / `exit` | Çık |

### FTP'de Dikkat

- Anonim giriş varsa → dosyaları incele (credential, config olabilir)
- Yazma izni varsa → reverse shell yükleyebilirsin
- FTP şifresiz iletir → ağda sniff edilebilir
