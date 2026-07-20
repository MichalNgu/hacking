# 🌐 14. PRINCIPY EXTERNÍHO PRŮZKUMU A ENUMERACE

## 🔍 1. Sběr IP prostoru a ASN (Network Mapping)

Při mapování rozsahu se zjišťuje, jaké síťové bloky (Netblocks) a autonomní systémy (ASN) klient vlastní. Využívají se nástroje jako **BGP Toolkit (Hurricane Electric)**.

- **Příklad v praxi:** Zjistíš, že doména `inlanefreight.com` míří na IP `134.209.24.248`, a odhalíš přidružené jmenné servery (`NS1` a `NS2`).
    
    - _Proč:_ V této fázi se ještě nedotýkáš samotného Active Directory. Zjišťuješ, kde infrastruktura fyzicky leží (zda je self-hosted, nebo v cloudu jako AWS/Azure), abys nezaútočil na cizí cíle mimo rozsah (Out of Scope).
        

## 📇 2. Enumerace DNS a subdomén

Pomocí webů jako `viewdns.info` nebo příkazů `nslookup`/`dig` ověřuješ IP adresy nalezených serverů a hledáš skryté subdomény (např. `vpn.inlanefreight.com`, `owa.inlanefreight.com`).

Bash

```
# Ruční ověření jmenných serverů nalezených z BGP
nslookup ns1.inlanefreight.com
```
    
- _Proč:_ Vyhledáváš vstupní body do organizace. Pokud DNS odhalí subdoménu typu `dev-stage.inlanefreight.com`, víš, že jde o testovací prostředí, které bude pravděpodobně slaběji zabezpečené a propojené do vnitřní sítě.
        

## 📄 3. Analýza veřejných dat a úniků (OSINT)

Hledání citlivých dokumentů pomocí vyhledávacích operátorů (Google Dorks), analýza pracovních pozic (např. hledání SharePoint administrátora prozradí verzi používaného softwaru) a prohledávání GitHub repozitářů.

Plaintext

```
# Google Dork pro vyhledávání PDF dokumentů na webu cíle
filetype:pdf inurl:inlanefreight.com
```

    
- _Proč:_ Stažené PDF soubory často v metadatech obsahují jméno autora ve formátu interního AD (např. `INLANEFREIGHT\rgrimes`), což ti dává přesný vzorec pro tvorbu uživatelských jmen. Nástroje jako `Trufflehog` zase mohou v kódu vývojářů najít zapomenuté API klíče.
        

## 👥 4. Sběr uživatelských jmen (Username Harvesting)

Cílem je vytvořit seznam validních zaměstnanců firmy. Používají se Google Dorky na kontaktní stránky nebo specializované nástroje jako **`linkedin2username`**, které automaticky seškrabou zaměstnance firmy z LinkedInu a převedou je do formátů jako `jyu`, `jane.yu` nebo `j.yu`.

Plaintext

```
# Google Dork pro sběr e-mailových adres
intext:"@inlanefreight.com" inurl:inlanefreight.com
```
    
    - _Proč:_ Active Directory vyžaduje pro jakoukoliv autentizaci platné uživatelské jméno. Tento vygenerovaný seznam je naprosto klíčovým palivem pro následující útoky na vnější přihlašovací portály.
        

## 🔑 5. Lov přihlašovacích údajů (Credential Hunting & Breach Data)

Zneužití historických úniků dat (např. přes službu **DeHashed** nebo API skripty). Hledají se záznamy, kde figurují e-maily zaměstnanců s jejich starými hesly v čistém textu nebo v hashi.

Bash

```
# Vyhledávání uniklých dat v DeHashed pro doménu inlanefreight.local
sudo python3 dehashed.py -q inlanefreight.local
```

_Vrátí výsledky typu: `roger.grimes@inlanefreight.local : Ilovefishing!`_

- _Proč:_ Lidé recyklují hesla. Pokud Roger Grimes použil heslo `Ilovefishing!` v minulosti na nějakém fitness portálu, který unikl, existuje obrovská šance, že ho má (nebo jeho mírnou variaci jako `Ilovefishing2026!`) nastavené i do firemního VPN, Office 365 nebo Citrix brány.