# Burp Scanner – Quick Reference

## Co je Burp Scanner

- Automatický skener zranitelností v Burp Suite Pro.
- Kombinuje:
    - Crawler (mapování webu)
    - Passive Scan (pasivní analýza)
    - Active Scan (aktivní testování)

---

## Scope

### K čemu slouží

Definuje, co Burp bude testovat.

### Postup

1. Target → Site Map
2. Pravé tlačítko → Add to Scope
3. Volitelně Remove from Scope
4. Přidat regex include/exclude pravidla

### V praxi

- Vyřadit logout endpointy
- Vyřadit destruktivní funkce
- Omezit scan pouze na cílovou aplikaci

---

## Crawl

### Co dělá

- Prochází odkazy
- Vyplňuje formuláře
- Mapuje aplikaci
- Vytváří Site Map

### Co nedělá

- Nehledá skryté adresáře
- Nenahrazuje ffuf/gobuster

### V praxi

Používám před každým testem pro získání přehledu o aplikaci.

---

## Passive Scan

### Co dělá

Pouze analyzuje existující odpovědi.

### Hledá

- Chybějící security hlavičky
- Cookie problémy
- Clickjacking
- DOM XSS indikace
- Konfigurační chyby

### Výhoda

- Bezpečný
- Neposílá nové payloady

---

## Active Scan

### Co dělá

1. Crawl
2. Discovery nových cest
3. Passive Scan
4. Ověření nálezů
5. Fuzzing parametrů
6. Testování běžných zranitelností

### Hledá

- XSS
- SQLi
- Command Injection
- SSTI
- LFI/RFI
- Další OWASP zranitelnosti

### V praxi

Používat pouze s povolením zákazníka.

---

## Severity & Confidence

### Severity

- High
- Medium
- Low
- Info

### Confidence

- Certain
- Firm
- Tentative

### Priorita

1. High + Certain
2. High + Firm
3. Medium + Certain

---

## Reporting

### Export

Target → Site Map → Issue → Report Issues

### Obsah reportu

- Popis zranitelnosti
- Důkaz (PoC)
- Dopad
- Doporučení k opravě

### Poznámka

Nikdy neposílat klientovi pouze automatický Burp report jako finální pentest report.