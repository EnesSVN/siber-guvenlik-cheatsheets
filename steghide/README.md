# Steganografi Cheatsheet

Dosyaların içine gizlenmiş veri bulma/çıkarma.

## Steghide (JPEG/BMP/WAV/AU)

```bash
# Gizli veri çıkar
steghide extract -sf resim.jpg

# Passphrase sorulursa → boş bırak veya brute force

# Bilgi göster (veri var mı?)
steghide info resim.jpg

# Veri gizle
steghide embed -cf resim.jpg -ef gizli.txt
```

## Diğer Araçlar

```bash
# Dosya türünü kontrol et (uzantı yanıltıcı olabilir)
file dosya.jpg

# String'leri ara (gizli metin, flag, url)
strings dosya.jpg | grep -i flag
strings dosya.jpg | grep -i pass

# Exif verisi (metadata — GPS, kamera, yazar)
exiftool dosya.jpg

# Binwalk — gömülü dosyaları bul ve çıkar
binwalk dosya.jpg
binwalk -e dosya.jpg   # çıkar

# Zsteg — PNG/BMP steganografi
zsteg resim.png
```

## CTF Refleksi

Bir resim/ses dosyası verilirse sırayla:
1. `file` → gerçek türü ne?
2. `strings` → düz metin gizlenmiş mi?
3. `exiftool` → metadata'da ipucu var mı?
4. `steghide extract` → gömülü dosya var mı?
5. `binwalk -e` → içine başka dosya gömülü mü?

## Gerçek Kullanımım

```
# Agent Sudo (TryHackMe):
# Resimdeki gizli veriyi steghide ile çıkardım
steghide extract -sf cute-alien.jpg
```
