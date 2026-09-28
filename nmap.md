# Nmap Cheatsheet

## Temel Tarama

```bash
nmap <hedef>                        # Varsayılan tarama (en yaygın 1000 port)
nmap -v <hedef>                     # Ayrıntılı çıktı
nmap <hedef1> <hedef2>              # Birden fazla hedef
nmap 192.168.1.0/24                 # Alt ağ taraması
nmap 192.168.1.1-50                 # IP aralığı taraması
nmap -iL hedefler.txt               # Dosyadan hedef listesi
```

## Port Seçenekleri

```bash
nmap -p 80 <hedef>                  # Belirli port
nmap -p 80,443,8080 <hedef>         # Birden fazla port
nmap -p 1-1024 <hedef>              # Port aralığı
nmap -p- <hedef>                    # Tüm 65535 port
nmap --top-ports 100 <hedef>        # En yaygın 100 port
```

## Tarama Türleri

```bash
nmap -sS <hedef>                    # SYN (stealth) tarama (varsayılan, root gerekir)
nmap -sT <hedef>                    # TCP connect tarama
nmap -sU <hedef>                    # UDP tarama
nmap -sN <hedef>                    # TCP Null tarama
nmap -sF <hedef>                    # TCP FIN tarama
nmap -sX <hedef>                    # Xmas tarama
nmap -sA <hedef>                    # ACK tarama (güvenlik duvarı tespiti)
nmap -sn <hedef>                    # Ping tarama (port tarama yok)
nmap -Pn <hedef>                    # Host keşfini atla (ping yok)
```

## Servis ve Versiyon Tespiti

```bash
nmap -sV <hedef>                    # Servis/versiyon tespiti
nmap -sV --version-intensity 9 <hedef>  # Maksimum yoğunlukta versiyon tespiti
nmap -O <hedef>                     # İşletim sistemi tespiti
nmap -A <hedef>                     # Agresif: OS, versiyon, script, traceroute
```

## Zamanlama ve Performans

```bash
nmap -T0 <hedef>                    # Paranoid (çok yavaş, IDS'den kaçınmak için)
nmap -T1 <hedef>                    # Sneaky
nmap -T2 <hedef>                    # Polite
nmap -T3 <hedef>                    # Normal (varsayılan)
nmap -T4 <hedef>                    # Aggressive (hızlı ağlar için)
nmap -T5 <hedef>                    # Insane (çok hızlı, hatalara yol açabilir)
nmap --min-rate 1000 <hedef>        # Saniye başına minimum paket sayısı
```

## NSE (Nmap Scripting Engine)

```bash
nmap -sC <hedef>                    # Varsayılan scriptleri çalıştır
nmap --script=<script> <hedef>      # Belirli script
nmap --script=vuln <hedef>          # Zafiyet taraması
nmap --script=http-enum <hedef>     # HTTP dizin keşfi
nmap --script=smb-vuln-* <hedef>    # SMB zafiyet scriptleri
nmap --script=banner <hedef>        # Banner grabbing
nmap --script-updatedb              # Script veritabanını güncelle
```

## Çıktı Seçenekleri

```bash
nmap -oN sonuc.txt <hedef>          # Normal metin çıktısı
nmap -oX sonuc.xml <hedef>          # XML çıktısı
nmap -oG sonuc.gnmap <hedef>        # Grepable çıktı
nmap -oA sonuc <hedef>              # Tüm formatlarda çıktı
nmap -v -oN sonuc.txt <hedef>       # Ayrıntılı + dosyaya kaydet
```

## Güvenlik Duvarı Atlatma

```bash
nmap -f <hedef>                     # Paket parçalama
nmap --mtu 16 <hedef>               # Özel MTU boyutu
nmap -D RND:10 <hedef>              # Rastgele decoy IP'leri
nmap -D <sahte_ip>,ME <hedef>       # Belirli decoy IP ile
nmap -S <sahte_ip> <hedef>          # Kaynak IP sahteciliği
nmap --source-port 53 <hedef>       # Kaynak port belirleme (53=DNS)
nmap --scan-delay 1s <hedef>        # Paketler arası gecikme
nmap --data-length 25 <hedef>       # Paketlere rastgele veri ekle
```

## Pratik Örnekler

```bash
# Hızlı ağ keşfi
nmap -sn 192.168.1.0/24

# Kapsamlı tek host taraması
nmap -A -T4 -p- <hedef>

# Web uygulaması taraması
nmap -sV -p 80,443,8080,8443 --script=http-* <hedef>

# SMB zafiyet taraması
nmap -p 445 --script=smb-vuln-* <hedef>

# Sessiz keşif
nmap -sS -T2 -Pn -f --data-length 25 <hedef>
```
