# Web Fingerprinting (Information Gathering)

Cíl:

- zjistit technologie webu
- najít verze služeb
- odhalit CMS, frameworky a ochrany
- připravit útok na konkrétní zranitelnosti

Pentester neútočí na "web", ale na konkrétní technologii:

```
Apache 2.4.41
PHP 7.4
WordPress 5.x
```

---

# 1. HTTP Headers & Banner Grabbing

Nejrychlejší kontrola:

```
curl -I http://domain.com
```

Přesměrování:

```
curl -I -L http://domain.com
```

---

## Hledat:

|Header|Informace|
|---|---|
|Server|Webserver + verze|
|X-Powered-By|Backend technologie|
|X-Redirect-By|CMS informace|
|Link|API/cesty|

Příklad:

```
Server: Apache/2.4.41
X-Powered-By: PHP/7.4
```

---

# 2. WAF Detection

Zjistit ochranu před skenováním:

```
wafw00f domain.com
```

Najde například:

- Cloudflare
- Wordfence
- Akamai

Pokud existuje WAF:

- pomalejší skenování
- méně požadavků
- vyhnout se detekci

---

# 3. Automatický Fingerprinting

## WhatWeb

Rychlá identifikace:

```
whatweb domain.com
```

Najde:

- CMS
- frameworky
- verze
- pluginy

---

## Nikto

Kontrola konfigurace:

```
nikto -h domain.com
```

Pouze základní fingerprint:

```
nikto -h domain.com -Tuning b
```

Hledá:

- zastaralý software
- nebezpečné soubory
- špatné hlavičky
- default stránky

---

# Typické nálezy Nikto

|Nález|Význam|
|---|---|
|outdated software|možné CVE|
|license.txt|únik informací|
|readme.html|informace o CMS|
|config backup|citlivá data|
|chybějící security headers|slabší ochrana|

---

# 4. Specializované nástroje

|Nástroj|Účel|
|---|---|
|Wappalyzer|Browser fingerprinting|
|BuiltWith|Technologie webu|
|Nmap HTTP scripts|Enumerace|

Nmap:

```
nmap -p80 \
--script http-enum,http-title \
TARGET
```

---

# CMS Fingerprinting

Hledat:

WordPress:

```
/wp-login.php
/wp-content/
```

Joomla:

```
/administrator/
```

Drupal:

```
/core/
```

---

# Web Fingerprinting Workflow

```
IP / Domain
      |
      ↓
HTTP Headers
      |
      ↓
WAF Detection
      |
      ↓
WhatWeb
      |
      ↓
Nikto
      |
      ↓
CMS Detection
      |
      ↓
Version Search
      |
      ↓
Vulnerability Enumeration
```

---

# CPTS Web Footprinting Checklist

☐ IPv4 / IPv6  
☐ Webserver verze  
☐ Backend technologie  
☐ CMS identifikace  
☐ WAF detekce  
☐ Security headers  
☐ Default soubory  
☐ Backup soubory  
☐ API endpointy  
☐ Verze komponent