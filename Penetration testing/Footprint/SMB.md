# SMB (Server Message Block)

## Základní informace

|Vlastnost|Popis|
|---|---|
|Protokol|SMB (Server Message Block)|
|Porty|139, 445|
|Účel|Sdílení souborů, tiskáren a dalších síťových prostředků|
|Architektura|Klient–Server|
|Autentizace|NTLM, Kerberos|

### Jak SMB funguje

|Krok|Popis|
|---|---|
|1|Klient naváže spojení|
|2|Proběhne vyjednání verze SMB (Dialect Negotiation)|
|3|Proběhne autentizace|
|4|Získá se přístup ke sdíleným složkám (Shares)|
|5|Probíhá práce se soubory a prostředky|

### Porty

|Port|Popis|
|---|---|
|139|Starší SMB přes NetBIOS|
|445|Moderní SMB přímo přes TCP/IP|

---

# Fáze 1 – Průzkum a Enumerace

**Cíl:** Zjistit, jaké SMB prostředky jsou dostupné a jaké informace lze získat.

|Úkol|Příkaz|
|---|---|
|Výpis sdílených složek anonymně|`smbclient -L //10.10.x.x/ -N`|
|Výpis sdílených složek jako uživatel|`smbclient -L //10.10.x.x/ -U uzivatel`|
|Kompletní enumerace SMB|`enum4linux -a 10.10.x.x`|
|RPC průzkum|`rpcclient -U "" -N 10.10.x.x`|

### Užitečné RPC příkazy

|Příkaz|Účel|
|---|---|
|`enumdomusers`|Výpis uživatelů domény|
|`querydominfo`|Informace o doméně|

---

# Fáze 2 – Přístup ke sdíleným prostředkům

## Připojení ke sdílené složce

|Úkol|Příkaz|
|---|---|
|Připojení ke sdílené složce|`smbclient //10.10.x.x/share -N`|

### Základní příkazy v SMB klientovi

|Příkaz|Funkce|
|---|---|
|`ls`|Výpis souborů|
|`cd`|Změna adresáře|
|`get`|Stažení souboru|
|`put`|Nahrání souboru|

---

# Kontrola zabezpečení SMB

|Co kontroluješ|Nástroj / Příkaz|
|---|---|
|Anonymní přístup|`smbclient -L //IP -N`|
|Verze SMB a OS|`nmap --script smb-os-discovery IP`|
|SMB Signing|`nmap --script smb-security-mode IP`|
|Známé SMB zranitelnosti|`nmap --script smb-vuln* IP`|

---

# Checklist SMB Enumerace

| Kontrola                     | Hotovo |
| ---------------------------- | ------ |
| Otevřený port 445            | ☐      |
| Otevřený port 139            | ☐      |
| Anonymní login               | ☐      |
| Výpis share složek           | ☐      |
| Enumerace uživatelů          | ☐      |
| Enumerace skupin             | ☐      |
| Heslová politika             | ☐      |
| SMB Signing                  | ☐      |
| Verze SMB                    | ☐      |
| Známé zranitelnosti          | ☐      |
| Přístup ke sdíleným souborům | ☐      |
| Citlivé soubory              | ☐      |
| Zápis do share               | ☐      |