
# 🛡️ Security Assessment Report: Machine "CAP" (HTB)

# 

**Datum:** 11. ledna 2026 | **Autor:** Michal Nguyen | **Status:** Final Report

* * *

## 1\. Executive Summary (Shrnutí)

# 

Během penetračního testu stroje „Cap“ byla identifikována kritická bezpečnostní rizika umožňující **úplnou kompromitaci systému**. Útok byl veden řetězením tří chyb: od neoprávněného přístupu k datům přes webové rozhraní (IDOR), přes odposlech citlivých údajů v nešifrované komunikaci (FTP), až po zneužití systémových privilegií (Capabilities).

**Hlavní dopad:** Útočník získal přístup k důvěrným souborům a následně plná administrátorská práva (root), což v reálném prostředí znamená riziko krádeže dat, instalace ransomware nebo trvalého poškození systému.

* * *

## 2\. Klasifikace rizik a závažnosti (Vulnerability Matrix)

# 

| **Zranitelnost** | **Klasifikace (CVSS)** | **Závažnost** | **Dopad** |
| --- | --- | --- | --- |
| **IDOR** (Zneužití URL parametrů) | 8.1 (High) | 🔴 **Kritická** | Neoprávněný přístup k souborům ostatních uživatelů. |
| **FTP Cleartext Communication** | 7.5 (High) | 🟠 **Vysoká** | Únik přihlašovacích údajů při odposlechu sítě. |
| **Misconfigured Capabilities** | 7.8 (High) | 🟠 **Vysoká** | Eskalace privilegií na úroveň Root. |

* * *

## 3\. Technická analýza útoku (Podrobný postup)

### Krok A: Externí průzkum a Enumerace (Nmap)

# 

Prvním krokem byl aktivní sken cílového systému pro identifikaci běžících služeb a jejich verzí.

-   **Příkaz:** `nmap -sV -sC -p- 10.10.10.245`
    
-   **Zjištění:** Sken odhalil tři klíčové vstupní body:
    
    1.  **Port 21 (FTP):** Služba `vsftpd 3.0.3` – potenciální cíl pro odposlech nešifrovaných dat.
        
    2.  **Port 22 (SSH):** Služba `OpenSSH 8.2p1` – standardní cesta pro vzdálenou správu.
        
    3.  **Port 80 (HTTP):** Webový server `gunicorn` (Python framework), sloužící jako dashboard pro síťovou analýzu.
        

_\- Podrobnosti viz screenshots/nmap\_scan.png_

* * *

### Krok B: Web Exploitation (IDOR)

# 

Analýza webového rozhraní ukázala, že aplikace generuje záznamy síťového provozu ve formátu PCAP.

-   **Zranitelnost:** Aplikace využívá nezabezpečené ID v URL pro přístup k souborům (Insecure Direct Object Reference).
    
-   **Vektor útoku:** Původní URL `/data/6` odkazovalo na prázdný záznam. Ruční manipulací s URL na `/data/0` jsem přistoupil k nultému záznamu systému, který obsahoval citlivý provoz uživatele `nathan`.
    

_\- Podrobnosti viz screenshots/page1.png a page2.png_

* * *

### Krok C: Exfiltrace údajů (Wireshark)

# 

Stažený soubor `0.pcap` byl podroben analýze. Protože protokol **FTP nešifruje data**, přihlašovací proces probíhá v prostém textu.

-   **Nález:** Pomocí filtru `ftp` a funkce "Follow TCP Stream" jsem extrahoval přihlašovací údaje.
    
-   **Identita:** Uživatel `nathan` použil heslo `Buck...`.
    

_\- Podrobnosti viz screenshots/wireshark.png_

* * *

### Krok D: Foothold (SSH Přístup)

# 

Získané údaje jsem použil k navázání interaktivního spojení se serverem skrze zabezpečený protokol SSH.

-   **Příkaz:** `ssh nathan@10.10.10.245`
    
-   **Výsledek:** Úspěšná autentizace a získání přístupu k souborovému systému pod identitou uživatele `nathan`.
    

_\- Podrobnosti viz screenshots/ssh\_connect.jpg_

* * *

### Krok E: Privilege Escalation (Linux Capabilities)

# 

Po získání přístupu byl proveden audit systému pro nalezení cesty k právům roota. Hledání se zaměřilo na soubory s rozšířenými atributy (Capabilities).

-   **Analýza:** Příkaz `getcap -r / 2>/dev/null` odhalil, že binárka `/usr/bin/python3.8` disponuje příznakem `cap_setuid+ep`.
    
-   **Exploitace:** Tato konfigurace umožňuje uživateli spustit Python a v rámci jeho procesu změnit své UID na 0 (root), čímž dojde k obejití standardních restrikcí systému.
    
-   **Příkaz:** `/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/sh")'`
    

_\- Podrobnosti viz screenshots/Privilege\_escalation.png_

* * *

## 4\. Získané důkazy (Flags)

# 

| **Typ Flag**  | **Hodnota (Hash)** | **Screenshot**  |
| ------------- | ------------------ | --------------- |
| **User Flag** | `4b3c.....`        | `flag_nat.png`  |
| **Root Flag** | `d569.....`        | `root_flag.png` |
* * *

## 5\. Doporučení pro nápravu (Remediation)

# 

1.  **Validace přístupu (IDOR):** Implementovat kontrolu na straně serveru, která ověří vlastnictví souboru před jeho stažením.
    
2.  **Šifrování přenosu:** Okamžitě zakázat protokol FTP a nahradit jej **SFTP**, který šifruje autentizaci i přenášená data.
    
3.  **Hloubková obrana (Capabilities):** Odstranit nepotřebná privilegia z interpretů (Python, PHP). Využívat zásadu minimálních privilegií (**Principle of Least Privilege**).

Do tohoto reportu byly HTB flagy, hesla a další citlivé údaje úmyslně anonymizovány, aby bylo dodrženo právní a etické rámce platformy Hack The Box.
