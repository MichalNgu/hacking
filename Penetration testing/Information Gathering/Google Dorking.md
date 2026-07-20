# Google Dorking (Search Engine Discovery)

Cíl:

- najít veřejně dostupné citlivé informace
- objevit zapomenuté soubory
- najít admin panely a subdomény

Princip:

```
Google index
      ↓
Pokročilé operátory
      ↓
Citlivé informace
```

---

# 🔎 Základní operátory

|Operátor|Účel|Příklad|
|---|---|---|
|site:|Omezení na doménu|`site:target.com`|
|inurl:|Hledání v URL|`inurl:admin`|
|filetype: / ext:|Typ souboru|`filetype:pdf`|
|intitle:|Titulek stránky|`intitle:"index of"`|
|intext:|Text na stránce|`intext:"password"`|
|cache:|Cache verze stránky|`cache:target.com`|
|link:|Odkazy na web|`link:target.com`|

---

# Logické operátory

|Symbol|Význam|
|---|---|
|OR|jedna z možností|
|-|vyloučení|
|" "|přesná fráze|
|*|wildcard|
|..|rozsah|

Příklad:

```
site:target.com -www
```

Najde subdomény mimo [www](http://www).

---

# Praktické pentest dotazy

## 🔑 Login / Admin panely

```
site:target.com inurl:login
```

```
site:target.com (inurl:admin OR inurl:setup)
```

```
site:target.com intitle:"login"
```

---

## 📂 Citlivé soubory

Zálohy:

```
site:target.com ext:sql
```

```
site:target.com ext:bak
```

Konfigurace:

```
site:target.com (ext:conf OR ext:ini OR ext:config)
```

Logy:

```
site:target.com filetype:log
```

Dokumenty:

```
site:target.com filetype:pdf "confidential"
```

---

# 📁 Directory Listing

Hledání otevřených adresářů:

```
site:target.com intitle:"index of"
```

Příklady:

```
site:target.com intitle:"index of /backup"
```

```
site:target.com intitle:"index of /admin"
```

---

# 🌐 Subdomain Discovery

```
site:*.target.com -www
```

Hledá například:

```
dev.target.com
test.target.com
staging.target.com
```

---

# Zajímavé nálezy

|Nález|Význam|
|---|---|
|.sql|Databázová záloha|
|.bak|Backup soubor|
|.env|Environment konfigurace|
|config.php|Přihlašovací údaje|
|web.config|Windows konfigurace|
|PDF/DOCX|Interní dokumenty|

---

# Zdroje

Google Hacking Database:

- Exploit-DB GHDB

Použití:

- hledání specifických systémů
- známé dorky pro CMS a zařízení

---

# CPTS Footprinting Checklist

☐ `site:target.com`  
☐ Subdomény (`site:*.target.com -www`)  
☐ Login stránky  
☐ Admin panely  
☐ PDF/DOC dokumenty  
☐ Backup soubory (`sql`, `bak`)  
☐ Konfigurační soubory (`env`, `ini`)  
☐ Directory listing