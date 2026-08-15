## 🧩 Rozšíření a doplňky (Extensions & Add-ons)

> **K čemu to je:** Obě aplikace umožňují komunitě vyvíjet vlastní doplňky, které rozšiřují funkčnost – od pokročilého dekódování, přes zjišťování specifických zranitelností (v technologiích jako .NET, Java, PHP, AWS), až po import nových databází slovníků.

### 1. Burp Suite (BApp Store)

- **Kde to najdeš:** Záložka `Extensions` ➔ podzáložka `BApp Store` _(u starších verzí v záložce `Extender`)_.
    
- **Instalace:** Vyber doplňku ze seznamu a klikni na **Install**.
    
- **Požadavky na prostředí:** Některé rozšíření pro svou funkci vyžadují dodatečné prostředí (např. **Jython** pro Python rozšíření nebo **JRuby**).
    
- **Příklad z modulu:** `Decoder Improved` – přidává do Burp Suite novou záložku s možností využívat desítky dalších kódovacích a hašovacích algoritmů (např. přímo generovat MD5, SHA256 atd.).
    

#### 💡 Užitečná rozšíření v Burpu pro pentesting:

- **Autorize:** Automatizované testování zranitelností v řízení přístupu (Broken Access Control / Authorization Bypass).
    
- **JS Link Finder / Retire.js:** Vyhledávání endpointů v JavaScriptu a detekce zastaralých/zranitelných JS knihoven.
    
- **Active Scan++ / Additional Scanner Checks:** Rozšíření možností automatického skeneru o pokročilé detekční testy.
    
- **Java Deserialization Scanner / PHP Object Injection Check:** Skener specifických zranitelností v objektové deserializaci.
    

### 2. OWASP ZAP (ZAP Marketplace)

- **Kde to najdeš:** Tlačítko `Manage Add-ons` v horní liště ➔ záložka `Marketplace`.
    
- **Stav doplňků:**
    
    - **Release:** Stabilní doplňky připravené pro běžné použití.
        
    - **Beta / Alpha:** Nové nebo testovací doplňky, které mohou občas vykazovat chyby.
        
- **Příklad z modulu:** `FuzzDB Files` a `FuzzDB Offensive` – do ZAP Fuzzeru nainstalují rozsáhlé komunitní databáze slovníků přímo připravené pro útoky.
    
    - _Příklad použití:_ Po instalaci najdeš v Fuzzeru pod `File Fuzzers` zavedené slovníky jako `fuzzdb > attack > os-cmd-execution`, které se hodí např. pro obcházení WAF při testování Command Injection.
        

### 💡 Závěrečné doporučení k celému modulu

Webové proxy jako Burp Suite a OWASP ZAP jsou základním stavebním kamenem offensive i defensive bezpečnosti. Doporučujeme kombinovat jejich používání s dalšími klíčovými nástroji (jako `nmap`, `ffuf`, `gobuster`, `sqlmap`, `Wireshark` nebo `Metasploit`) při praktickém řešení zranitelných strojů na platformě Hack The Box!