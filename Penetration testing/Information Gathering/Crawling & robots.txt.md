# Crawling & robots.txt (Web Information Gathering)

Cíl:

- zmapovat strukturu webu
- najít skryté stránky a soubory
- odhalit zapomenuté informace

---

# 🕸️ Web Crawling (Spidering)

Crawler automaticky prochází web:

```
Seed URL
   ↓
Stažení stránky
   ↓
Extrahování odkazů
   ↓
Další stránky
```

---

# Strategie crawlu

|Typ|Popis|Použití|
|---|---|---|
|Breadth-First|Prochází nejdříve všechny odkazy na úrovni|Rychlé mapování webu|
|Depth-First|Jde hluboko do jedné větve|Hledání skrytých souborů|

---

# Co hledat při crawlu

### HTML komentáře

Vývojáři mohou nechat:

- poznámky
- testovací údaje
- interní informace

Příklad:

```
<!-- TODO: remove admin panel -->
```

---

### Metadata

Hledat:

- jména autorů
- verze CMS
- technologie

---

### Citlivé soubory

Typické nálezy:

```
.bak
.old
~
config.php
web.config
settings.php
error.log
```

---

### Skryté cesty

Například:

```
/dev
/test
/admin
/backup
```

---

# 🤖 robots.txt

Umístění:

```
/robots.txt
```

Používá se pro vyhledávače, ale pro pentestera může odhalit zajímavé cesty.

---

# Struktura robots.txt

|Položka|Význam|
|---|---|
|User-agent|Pro koho pravidlo platí|
|Disallow|Zakázané cesty|
|Allow|Povolené výjimky|
|Sitemap|Mapa webu|

Příklad:

```
User-agent: *
Disallow: /admin/
Disallow: /backup/
```

---

# Zajímavé cesty

|Cesta|Možný význam|
|---|---|
|/admin/|Admin panel|
|/backup/|Zálohy|
|/dev/|Vývojová verze|
|/config/|Konfigurace|

---

# Kontext je důležitý

Samotný robots.txt není zranitelnost.

Příklad:

```
robots.txt
      ↓
Disallow: /backup/
      ↓
/backup/
      ↓
db_backup.sql
      ↓
Únik databáze
```

---

# Nástroje

## Scrapy instalace

```
sudo apt install python3-venv

python3 -m venv venv

source venv/bin/activate

pip install scrapy
```

Ukončení:

```
deactivate
```

---

# CPTS Footprinting Checklist

☐ Prošel jsem `/robots.txt`  
☐ Zkontroloval jsem `sitemap.xml`  
☐ Prošel jsem HTML source  
☐ Hledal jsem komentáře  
☐ Hledal jsem backup soubory  
☐ Kontroloval jsem skryté adresáře  
☐ Ověřil jsem directory listing