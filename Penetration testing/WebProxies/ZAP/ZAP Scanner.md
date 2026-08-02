# ZAP Scanner – Quick Notes

## Co je ZAP Scanner

- Open-source alternativa k Burp Scanneru.
- Obsahuje:
    - Spider
    - Ajax Spider
    - Passive Scanner
    - Active Scanner
    - Reporting

---

# Spider

## Co dělá

- Prochází odkazy
- Ověřuje URL
- Vytváří Site Tree

## Použití

Attack → Spider

## Výsledek

Najde:

- stránky
- adresáře
- soubory
- endpointy

### Poznámka

Podobné jako Burp Crawler.

---

# Ajax Spider

## Co dělá

- Hledá URL generované JavaScriptem
- Zachytává AJAX požadavky

## Kdy použít

Po dokončení normálního Spideru.

### Zapamatovat

Spider = HTML odkazy

Ajax Spider = JavaScript odkazy

---

# Scope

## Co je Scope

Seznam URL, které bude ZAP testovat.

## Výhody

- Omezení rozsahu testu
- Menší počet requestů
- Bezpečnější skenování

---

# Passive Scanner

## Co dělá

Analyzuje odpovědi bez posílání útoků.

## Hledá

- Chybějící security hlavičky
- X-Frame-Options
- DOM XSS indikace
- Konfigurační chyby

## Výhoda

Bezpečný provoz.

---

# Active Scanner

## Co dělá

Aktivně testuje aplikaci.

### Testuje

- Parametry
- Formuláře
- Endpointy

### Hledá

- XSS
- SQL Injection
- Command Injection
- LFI/RFI
- Directory Traversal
- Další webové zranitelnosti

## Nevýhoda

- Hodně requestů
- Může být hlučný
- Pouze s povolením

---

# Alerts

## Priorita

### High

Nejzávažnější

- často RCE
- SQLi
- Command Injection

### Medium

Střední riziko

### Low

Menší problémy

### Informational

Pouze informace

---

# Příklad nálezu

## Remote OS Command Injection

Payload:

```
127.0.0.1&cat /etc/passwd&
```

Evidence:

```
root:x:0:0:
```

Důkaz, že příkaz byl spuštěn na serveru.

---

# Reporting

## Export

Report → Generate HTML Report

### Formáty

- HTML
- XML
- Markdown

## Obsah

- Nálezy
- Severity
- Důkazy
- URL
- Doporučení

---
### Burp vs ZAP

|Funkce|Burp|ZAP|
|---|---|---|
|Crawler|Crawl|Spider|
|JS Crawling|Omezeně|Ajax Spider|
|Passive Scan|Ano|Ano|
|Active Scan|Ano|Ano|
|Cena|Pro placený|Zdarma|
|CPTS využití|Často|Často|
