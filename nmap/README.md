# nmap Cheatsheet

Ağ keşfi ve port tarama aracı. Her CTF'in ilk adımı.

## Temel Tarama

```bash
# Standart tarama (en çok kullandığım)
nmap -sC -sV <ip>
# -sC  = default script'leri çalıştır (versiyon, zafiyet ipucu)
# -sV  = servis versiyonlarını tespit et

# Tüm portları tara (yavaş ama kapsamlı)
nmap -p- <ip>

# Belirli port
nmap -p 80,443,8080 <ip>

# Port aralığı
nmap -p 1-1000 <ip>
```

## Tarama Türleri

```bash
# TCP SYN (varsayılan, hızlı, gizli)
nmap -sS <ip>

# TCP Connect (tam bağlantı)
nmap -sT <ip>

# UDP tarama (DNS, SNMP gibi servisleri bulur)
nmap -sU <ip>

# Ping tarama (host canlı mı?)
nmap -sn <subnet>
# Örnek: nmap -sn 192.168.1.0/24
```

## Faydalı Flagler

| Flag | Ne yapar? |
|---|---|
| `-sC` | Default NSE script'leri çalıştır |
| `-sV` | Servis versiyon tespiti |
| `-p-` | Tüm 65535 portu tara |
| `-O` | İşletim sistemi tespiti |
| `-A` | Agresif: OS + versiyon + script + traceroute |
| `-T4` | Hız seviyesi (0=paranoid, 5=insane) |
| `-oN dosya.txt` | Çıktıyı dosyaya kaydet |
| `-Pn` | Ping atma, direkt tara (host "kapalı" görünürse) |
| `--open` | Sadece açık portları göster |

## Gerçek Kullanım Örneklerim

```bash
# Simple CTF'te kullandığım
nmap -sC -sV 10.10.x.x
# Bulduğum: 21 (FTP), 80 (HTTP), 2222 (SSH)

# Hızlı + kapsamlı combo
nmap -sC -sV -T4 --open <ip>

# Önce hızlı tara, sonra detaylı
nmap --open <ip>              # hangi portlar açık?
nmap -sC -sV -p 21,80,2222 <ip>  # sadece açıklara odaklan
```

## Port → Refleks Tablosu

| Port | Servis | Aklına gelmesi gereken |
|---|---|---|
| 21 | FTP | Anonim giriş dene (`anonymous`) |
| 22 | SSH | Brute force (hydra) veya credential reuse |
| 80/443 | HTTP/S | Web → gobuster, nikto, Burp |
| 139/445 | SMB | EternalBlue? `smbclient` ile bak |
| 3306 | MySQL | Uzaktan bağlantı açık mı? |
| 8080 | HTTP alt | Proxy, Tomcat, alternatif web |
