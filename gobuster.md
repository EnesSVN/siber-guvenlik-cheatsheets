# Gobuster / ffuf Cheatsheet

## Gobuster

### Dizin/Dosya Keşfi (dir modu)

```bash
gobuster dir -u http://hedef.com -w /usr/share/wordlists/dirb/common.txt

# Uzantı filtresi
gobuster dir -u http://hedef.com -w wordlist.txt -x php,html,txt,bak

# Cookie ile kimlik doğrulama
gobuster dir -u http://hedef.com -w wordlist.txt -c "session=abc123"

# Özel header
gobuster dir -u http://hedef.com -w wordlist.txt -H "Authorization: ******"

# HTTPS (SSL hatalarını yoksay)
gobuster dir -u https://hedef.com -w wordlist.txt -k

# Durum kodu filtresi
gobuster dir -u http://hedef.com -w wordlist.txt -s "200,204,301,302"
gobuster dir -u http://hedef.com -w wordlist.txt -b "404,403"   # 404 ve 403'ü hariç tut

# Thread sayısı ve gecikme
gobuster dir -u http://hedef.com -w wordlist.txt -t 50 --delay 100ms

# Sonuçları kaydet
gobuster dir -u http://hedef.com -w wordlist.txt -o sonuc.txt

# Proxy üzerinden
gobuster dir -u http://hedef.com -w wordlist.txt --proxy http://127.0.0.1:8080
```

### DNS Alt Alan Adı Keşfi (dns modu)

```bash
gobuster dns -d hedef.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# IP adresini de göster
gobuster dns -d hedef.com -w wordlist.txt -i

# Wildcard'ı zorla
gobuster dns -d hedef.com -w wordlist.txt --wildcard
```

### Virtual Host Keşfi (vhost modu)

```bash
gobuster vhost -u http://hedef.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
gobuster vhost -u http://hedef.com -w wordlist.txt --append-domain
```

---

## ffuf (Fuzz Faster U Fool)

### Temel Söz Dizimi

```bash
ffuf -u http://hedef.com/FUZZ -w wordlist.txt
```

### Dizin/Dosya Keşfi

```bash
# Temel
ffuf -u http://hedef.com/FUZZ -w /usr/share/wordlists/dirb/common.txt

# Uzantıyla
ffuf -u http://hedef.com/FUZZ -w wordlist.txt -e .php,.html,.txt,.bak

# Belirli durum kodlarını filtrele
ffuf -u http://hedef.com/FUZZ -w wordlist.txt -mc 200,301,302
ffuf -u http://hedef.com/FUZZ -w wordlist.txt -fc 404,403

# Belirli yanıt boyutunu filtrele
ffuf -u http://hedef.com/FUZZ -w wordlist.txt -fs 1234

# Sonuçları kaydet
ffuf -u http://hedef.com/FUZZ -w wordlist.txt -o sonuc.json -of json
```

### Parametre Fuzzing

```bash
# GET parametresi
ffuf -u "http://hedef.com/page?FUZZ=deger" -w params.txt

# GET değeri
ffuf -u "http://hedef.com/page?id=FUZZ" -w /usr/share/seclists/Fuzzing/numbers.txt

# POST verisi
ffuf -u http://hedef.com/login -w wordlist.txt -d "username=admin&pass=FUZZ" -X POST
```

### Virtual Host / Alt Alan Adı Keşfi

```bash
ffuf -u http://hedef.com -H "Host: FUZZ.hedef.com" -w subdomains.txt -fs <varsayilan_boyut>
```

### Kimlik Doğrulama ve Header

```bash
# Cookie
ffuf -u http://hedef.com/FUZZ -w wordlist.txt -H "Cookie: session=abc123"

# Authorization header
ffuf -u http://hedef.com/FUZZ -w wordlist.txt -H "Authorization: ******"

# HTTPS (SSL hatalarını yoksay)
ffuf -u https://hedef.com/FUZZ -w wordlist.txt -k
```

### Çoklu Wordlist (Cluster Bomb)

```bash
ffuf -u http://hedef.com/login -w kullanicilar.txt:USER -w parolalar.txt:PASS \
  -d "username=USER&pass=PASS" -X POST -fc 401
```

### Seçenekler Özeti

```
-u          Hedef URL (FUZZ yerine kelime eklenir)
-w          Wordlist (yol:ETIKET söz dizimi çoklu wordlist için)
-H          HTTP header
-b          Cookie
-d          POST verisi
-X          HTTP metodu
-t          Thread sayısı (varsayılan: 40)
-p          İstekler arası gecikme
-mc         Durum kodu filtresi (virgülle ayrılmış)
-fc         Durum kodu hariç tut
-ms         Yanıt boyutu filtresi
-fs         Yanıt boyutu hariç tut
-ml         Satır sayısı filtresi
-mr         Regex eşleşmesi
-e          Uzantılar
-o          Çıktı dosyası
-of         Çıktı formatı (json, ejson, html, md, csv, ecsv)
-k          SSL doğrulamasını atla
-r          Yönlendirmeleri takip et
-v          Ayrıntılı mod
```

## Yaygın Wordlist'ler

```
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirb/big.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/common.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```
