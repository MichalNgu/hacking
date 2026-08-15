## 🔍 Metody vyhledávání XSS

Detekce zranitelností XSS vyžaduje kombinaci automatických nástrojů a manuální analýzy. Rozlišujeme tři základní přístupy:

### 1. Automatické skenování (Automated Discovery)

Nástroje automaticky vyhledávají vstupní pole, odesílají připravené payloady a kontrolují, zda se kód spustil nebo objevil v rendered HTML.

- **Profesionální skenery (Active / Passive):**
    
    - **Burp Suite Pro / OWASP ZAP / Nessus:** Zvládají aktivní skenování (odesílání injection payloadů) i pasivní analýzu klientského JavaScriptu na přítomnost zranitelných _Sinks_.
        
- **Open-Source nástroje pro CLI:**
    
    - **XSStrike:** Pokročilý nástroj analyzující kontext vstupu a generující optimální payloady přímo na míru.
        
    - **XSSer / Brute XSS:** Nástroje pro masivní fúzování vstupů (fuzzing).
        

Bash

```
# Ukázka použití XSStrike pro testování parametru v URL
git clone https://github.com/s0md3v/XSStrike.git
cd XSStrike && pip install -r requirements.txt
python xsstrike.py -u "http://target.com/index.php?task=test"
```

> ⚠️ **Důležité:** Žádný automatizovaný skener není 100% přesný. Vždy je nutné nalezenou zranitelnost ověřit **manuálně**.

### 2. Manuální testování (Manual Discovery)

Spočívá v ručním zadávání testovacích řetězců do formulářů, URL parametrů nebo HTTP hlaviček (`Cookie`, `User-Agent`).

- **Použití databází payloadů:** Repositaráře jako _PayloadAllTheThings_ nebo _Payload-Box_ obsahují tisíce payloadů pro různé konteksty (uzavírání uvozovek, HTML atributy, obcházení WAF).
    
- **Vlastní skriptování:** Pokud veřejné seznamy nebo skenery selžou, nejlepším přístupem bývá napsat si vlastní Python skript pro úpravu a odesílání payloadů na míru dané aplikaci.
    

### 3. Revize zdrojového kódu (Code Review)

Nejspolehlivější technika pro odhalení zranitelností, které unikly automatickým nástrojům (včetně Zero-Day zranitelností).

- **Front-end review (DOM XSS):** Hledání nebezpečných spojení mezi vstupy uživatele (_Source_) a jejich nekritickým vykreslením do DOMu (_Sink_, např. `innerHTML`, `document.write`).
    
- **Back-end review (Stored / Reflected):** Kontrola, zda server provádí adekvátní sanitizaci a kontextové kódování (HTML Entity Encoding) před uploadem nebo vrácením dat.