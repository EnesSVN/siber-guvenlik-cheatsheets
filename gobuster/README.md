# Gobuster Cheatsheet

Gizli dizin ve dosya keşfi. Web taramasında nmap'ten sonraki adım.

## Temel Kullanım

```bash
# Dizin keşfi (en sık kullanım)
gobuster dir -u http://<ip> -w /usr/share/wordlists/dirb/common.txt

# Daha büyük wordlist
gobuster dir -u http://<ip> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

## Önemli Flagler

| Flag | Ne yapar? |
|---|---|
| `dir` | Dizin/dosya brute force modu |
| `-u` | Hedef URL |
| `-w` | Wordlist dosyası |
| `-x` | Uzantı ara (php,txt,html,bak) |
| `-t` | Thread sayısı (varsayılan 10) |
| `-o` | Çıktıyı dosyaya kaydet |
| `-s` | Sadece belirli status code'ları göster |
| `-b` | Belirli status code'ları gizle |
| `-r` | Redirect'leri takip et |

## Sık Kullanılan Komutlar

```bash
# PHP dosyaları ara
gobuster dir -u http://<ip> -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak

# Hızlı tara (50 thread)
gobuster dir -u http://<ip> -w /usr/share/wordlists/dirb/common.txt -t 50

# Alt dizinde ara
gobuster dir -u http://<ip>/simple -w /usr/share/wordlists/dirb/common.txt
```

## Wordlist'ler (Kali'de hazır)

```
/usr/share/wordlists/dirb/common.txt          → küçük, hızlı
/usr/share/wordlists/dirb/big.txt              → orta
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt → büyük, kapsamlı
/usr/share/wordlists/rockyou.txt               → şifre kırma (gobuster için değil)
```

## Gerçek Kullanımım

```bash
# Simple CTF'te:
gobuster dir -u http://10.10.x.x -w /usr/share/wordlists/dirb/common.txt
# /simple dizini bulundu → CMS Made Simple → SQLi zinciri başladı
```

## Alternatifler

| Araç | Fark |
|---|---|
| `dirb` | Daha basit, recursive varsayılan |
| `feroxbuster` | Rust, çok hızlı, recursive |
| `ffuf` | Fuzzing + dizin keşfi, en esnek |
