
---

# 🛡️ Introduction to Penetration Testing

## 💡 Klíčové koncepty
- **Metodologie:** Opakovatelný proces (Recon -> Enum -> Exploit -> Post-Exploit -> Report).
- **Hands-on:** Vše se učí v simulovaných labech (HTB Boxes).
- **Ethics:** Rozdíl mezi pentesterem a kyber-zločincem je **písemný souhlas**.

## ⚖️ Právní základy (Compliance)
- **Scope of Work (SoW):** Dokument definující povolené cíle (IP, URL, CIDR).
- **Rules of Engagement (RoE):** Pravidla, jak se smí útočit (např. zákaz útoků mimo pracovní dobu).
- **Liability:** Bez smlouvy jsi právně odpovědný za jakoukoli škodu.

## 🛠️ Fáze útoku v kurzu
1. **Reconnaissance:** Průzkum veřejných dat (OSINT).
2. **Enumeration:** Skenování služeb a verzí (Nmap).
3. **Exploitation:** Získání přístupu (Payloads, Metasploit).
4. **Post-Exploitation:** - **Privilege Escalation:** Získání Root/SYSTEM práv.
    - **Lateral Movement:** Pohyb mezi stroji v síti.
    - **Pillaging:** Hledání citlivých dat na stroji.
5. **Reporting:** Finální výstup pro klienta.

> [!IMPORTANT] 
> **Vždy pracuj v rámci scope!** Pokud narazíš na chybu mimo rozsah, nahlas to, ale nepokračuj v útoku bez písemného schválení.


---

# 🗺️ Academy Modules Layout & Methodology

## 🛤️ Fáze Pentestu (Life Cycle)

| Fáze | Cíl | Klíčové moduly |
| :--- | :--- | :--- |
| **Základy** | Rozumět IT infrastruktuře | Linux/Windows Fund., Networking, Web Requests |
| **Recon** | Najít všechny "vchody" | Nmap, Footprinting, OSINT |
| **Analysis** | Vybrat nejslabší místo | Vulnerability Assessment, Shells & Payloads |
| **Exploit** | Projít dovnitř | Password Attacks, SQL Injection, Metasploit |
| **Post-Ex** | Ovládnout síť | PrivEsc, Pillaging, Pivoting |

## 💡 Důležité termíny k zapamatování
- **Enumeration:** Aktivní dotazování cíle (např. pomocí Nmap).
- **Footprinting:** Pasivní i aktivní určování verze a typu softwaru.
- **Payload:** Kód, který se spustí na cílovém stroji (např. reverse shell).
- **Pivoting:** Technika, kdy se z jednoho stroje stane "proxy" pro útok na další stroje uvnitř sítě.
- **Pillaging:** "Rabování" informací z napadeného systému (hesla, dokumenty).

> [!TIP]
> **Learning Philosophy:** Neuč se jen příkazy. Snaž se pochopit "proč" to funguje. Pokud víš, proč nástroj selhal, dokážeš si cestu najít ručně.


---

# 🛡️ Penetration Testing Overview

## 📂 Základní pojmy
- **Confidentiality (Důvěrnost):** Data vidí jen ten, kdo má.
- **Integrity (Integrita):** Data se cestou nezměnila.
- **Availability (Dostupnost):** Služba běží, když je potřeba.
- **Inherent Risk:** Riziko, které zůstává i po zavedení všech ochran.

## 🏁 Typy Pentestů (Information Levels)
| Typ | Co známe? | Výhody |
| :--- | :--- | :--- |
| **Blackbox** | Jen IP / Doména | Simuluje skutečného vnějšího útočníka. |
| **Greybox** | User přístup / URL | Zlatá střední cesta, šetří čas na Reconu. |
| **Whitebox** | Kód / Konfigurace | Maximální pokrytí, najde i skryté logické chyby. |

## 🏗️ Prostředí k testování
- **Network / Web App / Mobile / API**
- **Cloud** (AWS, Azure, GCP)
- **IoT / SCADA**
- **Social Engineering / Physical Security**

## ⚖️ Právní minimum
- Vždy ověřit **vlastnictví aktiv** (pokud klient používá hosting u 3. stran, může být potřeba jejich souhlas).
- **GDPR / Data Protection:** Pokud narazíš na citlivá data (kreditky, platy), okamžitě je reportuj a dál s nimi nemanipuluj.

> [!WARNING]
> Pentest je **momentální snapshot**. To, že je systém čistý dnes, neznamená, že bude čistý i po zítřejší aktualizaci.


---

# ⚖️ Právní rámec pro Pentesting (EU & UK)

## 🇪🇺 Evropská unie & ČR
| Zákon / Nařízení | Co řeší? | Dopad na pentestera |
| :--- | :--- | :--- |
| **GDPR** | Ochrana osobních údajů (PII). | Zákaz neoprávněné manipulace s daty uživatelů. Pokuty až 20 mil. EUR. |
| **NISD 2** | Bezpečnost kritické infrastruktury. | Povinnost hlásit incidenty. Týká se státní správy, energetiky atd. |
| **e-Privacy Directive** | Soukromí v elektronické komunikaci. | Reguluje odposlechy a sledování (cookies, metadata). |
| **Cybercrime Convention** | Mezinárodní spolupráce. | Rámec pro stíhání hackerů napříč hranicemi v EU. |

## 🇬🇧 Velká Británie (UK)
| Zákon | Co řeší? |
| :--- | :--- |
| **Computer Misuse Act 1990** | Hlavní zákon proti hackingu (neautorizovaný přístup). |
| **Data Protection Act 2018** | Britská verze GDPR. |
| **Investigatory Powers Act** | Reguluje vládní hacking a sledování. |

## 🛡️ Best Practices (Jak nebýt jednou nohou v kriminále)
1. **Písemný souhlas:** Vždy podepsaný kontrakt před prvním pingem.
2. **Data Minimization:** Pokud k důkazu o zranitelnosti stačí prvních 5 znaků z databáze, netahej jich 5000.
3. **Report:** Finální zprávu posílej klientovi vždy šifrovanou (např. 7z s heslem nebo přes zabezpečené úložiště).

> [!CAUTION] 
> V EU je trestným činem i pouhé **vytvoření nebo držení** "nástrojů pro hacking", pokud je úmyslem páchat trestnou činnost. Proto mějte nástroje vždy jen na pracovních strojích a používejte je výhradně k autorizovaným testům.


---

# ⚙️ Penetration Testing Process (Fáze)

## 🔄 Životní cyklus testu
Proces není lineární, ale **iterativní**. Např. po Lateral Movement se vracíš k Information Gathering na novém stroji.

| Pořadí | Fáze | Hlavní úkol |
| :--- | :--- | :--- |
| 1 | **Pre-Engagement** | Smlouvy, Scope, Rules of Engagement (RoE). |
| 2 | **Info Gathering** | Sběr dat o technologiích a infrastruktuře. |
| 3 | **Vuln Assessment** | Identifikace slabin (manuálně i skenery). |
| 4 | **Exploitation** | Průnik do systému (získání "Foothold"). |
| 5 | **Post-Exploitation** | **PrivEsc** (práva) a **Pillaging** (sběr dat uvnitř). |
| 6 | **Lateral Movement** | Přesun na další stroje v síti (Pivoting). |
| 7 | **Proof-of-Concept** | Dokumentace řetězce útoků (Attack Chain). |
| 8 | **Post-Engagement** | **Reporting** a **Cleanup** (úklid po útocích). |

## 💡 Klíčové termíny
- **Iterative Process:** Opakující se kolečko činností.
- **Foothold:** První stabilní přístup k cílovému systému.
- **Cleanup:** Kritický krok – odstranění všech útočných nástrojů po testu.

> [!IMPORTANT]
> **Reporting** je nejdůležitější část. Bez dobré zprávy je celý tvůj technický výkon pro klienta bezcenný.

---

# 📝 Pre-Engagement & Dokumentace

## 🤝 Typy NDAs
- **Unilateral:** Chrání jen jednu stranu.
- **Bilateral:** Obě strany drží jazyk za zuby (Standard).
- **Multilateral:** Zapojení více firem (např. subdodavatelé).

## 📄 Nezbytná dokumentace
| Dokument | Kdy se podepisuje? | Účel |
| :--- | :--- | :--- |
| **NDA** | Hned na začátku | Ochrana citlivých dat. |
| **Scoping Doc** | Před smlouvou | Definice, co se vlastně bude testovat. |
| **SoW / Smlouva** | Před zahájením | Právní a finanční rámec. |
| **RoE** | Před Kick-offem | Technické detaily: IP adresy, časy, omezení. |

## 🛡️ Checklist pro Kick-off Meeting
- [ ] **Emergency Contact:** Koho volat ve 2 ráno, když spadne server?
- [ ] **Critical Findings:** Pokud najdu kritickou chybu (RCE), hlásím ji okamžitě, nečekám na report.
- [ ] **Exclusions:** Jsou nějaké servery, na které nesmím za žádnou cenu sáhnout?
- [ ] **Log Monitoring:** Bude klient sledovat naše logy pro trénink svých lidí?

> [!WARNING]
> **Oprávněná osoba:** Smlouvu musí podepsat někdo, kdo k tomu má právo (CEO, CISO, CTO). Zaměstnanec bez pověření tě nemůže legálně najmout na testování firemní sítě.

## 🏢 Fyzický Pentest (Physical Assessment)
- Vyžaduje **Contractors Agreement**.
- Slouží jako potvrzení pro policii/ochranku při dopadení.

---

# 🔍 Information Gathering & Pillaging

## 📂 Kategorie sběru informací
| Kategorie | Co hledáme? | Nástroje / Zdroje |
| :--- | :--- | :--- |
| **OSINT** | Veřejná data, uniklá hesla, klíče. | GitHub, Google Dorks, LinkedIn, Shodan. |
| **Infrastructure** | IP rozsahy, subdomény, DNS, firewally. | Nmap, Dig, Whois, Sublist3r. |
| **Service Enum** | Verze běžících služeb (HTTP, SSH, SMB). | Nmap (-sV), Banner Grabbing. |
| **Host Enum** | Typ OS, role stroje, vnitřní nastavení. | Nmap (-O), enum4linux, SMBMap. |

## 🏴‍☠️ Pillaging (Post-Exploitation)
Proces sběru dat na kompromitovaném hostiteli za účelem:
1. **Důkazu dopadu:** Ukázka, co vše mohl útočník ukrást.
2. **Dalšího postupu:** Získání hesel pro **PrivEsc** nebo **Lateral Movement**.

### Co hledat při pillagingu:
- [ ] SSH klíče (`~/.ssh/id_rsa`)
- [ ] Historie příkazů (`.bash_history`, PowerShell history)
- [ ] Konfigurační soubory (web.config, wp-config.php, .env)
- [ ] Zálohy databází (.sql, .bak)
- [ ] Lokální uživatelské profily a dokumenty.

> [!TIP]
> **Internal vs External:** Služby, které vypadají z internetu zabezpečeně, jsou uvnitř sítě často nastaveny laxně. Admini spoléhají na to, že "uvnitř jsou jen přátelé".


---

# 🧠 Vulnerability Assessment (Analýza)

## 🔍 Proces analýzy dat
1. **Identifikace:** Mám seznam IP a portů.
2. **Výzkum (CVE):** Porovnání verzí služeb s databázemi zranitelností.
3. **Validace:** Ověření, zda je zranitelnost skutečně přítomná (vyloučení False Positives).
4. **Prioritizace:** Která zranitelnost představuje největší riziko pro klienta?

## 📚 Zdroje pro výzkum (Kde hledat exploity)
- [Exploit Database](https://www.exploit-db.com/)
- [NVD (NIST)](https://nvd.nist.gov/)
- [Vulners](https://vulners.com/)
- [Packet Storm](https://packetstormsecurity.com/)

## 🔄 Interaktivní kolečko
Pokud analýza nic neodhalí:
`Vulnerability Assessment` ➔ `Nedostatek dat` ➔ **Návrat k Information Gathering** ➔ `Nová data` ➔ `Zpět k analýze`.

> [!TIP]
> **Mirroring:** Pokud musíš být extrémně tichý (**Evasive**), postav si kopii cílového systému u sebe lokálně (virtuální stroj) a testuj exploity nejdřív tam, abys nevyvolal poplach u klienta.


---

# 🚀 Exploitation (Průnik)

## 🎯 Cíle fáze
- Získat **Foothold** (první přístup).
- Minimalizovat riziko pádu systému.
- Získat podklady pro report (screenshoty, logy).

## 📊 Rozhodovací proces (Prioritizace)
Při výběru exploitu sleduj:
- **Success Rate:** Jak spolehlivý je kód?
- **Stability:** Je exploit "bezpečný" pro běh systému?
- **Effort:** Jak dlouho trvá konfigurace?

## 🛠️ Příprava Reverse Shellu
Pokud upravuješ PoC pro získání přístupu:
1. **Payload:** Vyber správný typ (např. `python`, `php`, `powershell`).
2. **LHOST/LPORT:** Nastav svou IP a port, na kterém posloucháš (netcat/msfconsole).
3. **Encryption:** Pokud je to možné, použij šifrované spojení, abys nebyl odhalen (Evasion).

> [!CAUTION]
> **Exploit != Magic.** Vždy musíš rozumět tomu, co daný kód dělá, než ho spustíš. Spouštění neznámých skriptů z internetu pod právy roota je nejrychlejší cesta k průšvihu.

## ✅ Po úspěšném průniku
- Zapiš si přesný čas a použitý exploit.
- Udělej screenshot (např. příkaz `whoami` a `ifconfig`).
- Pokračuj na fází **Post-Exploitation**.

---

# 🚩 Post-Exploitation (Uvnitř systému)

## 🛠️ Checklist po získání přístupu
1. **Stabilizace Shellu:** Ujistit se, že spojení nespadne při první chybě.
2. **Persistence:** Nastavit si cestu zpět (zadní vrátka).
3. **Pillaging:** - [ ] `whoami`, `id`, `hostname` (Kdo jsem a kde jsem?)
    - [ ] `ip a`, `route` (Kam odtud vidím?)
    - [ ] Prohledat `/home` nebo `C:\Users` pro citlivé soubory.
4. **PrivEsc:** Najít cestu k Root/SYSTEM (Sudo práva, špatné verze jádra, Misconfigurations).

## 🛡️ Evasive Testing (Úrovně)
- **Evasive:** Cílem je nebýt odhalen (Blind spots).
- **Hybrid:** Testujeme, při jaké intenzitě nás SIEM/EDR zachytí.
- **Non-Evasive:** Maximizace pokrytí testu bez ohledu na hluk.

## ⚖️ Regulace pro data (Exfiltration)
Při testování vynášení dat pamatuj na:
- **PCI-DSS:** Kreditní karty.
- **HIPAA:** Zdravotní údaje.
- **GDPR:** Osobní údaje v EU.

> [!CAUTION]
> **Cleanup:** Vše, co v této fázi vytvoříte (uživatele, skripty, persistence), musíte po testu smazat! Nechte systém ve stejném stavu, v jakém byl.


---

# 🏹 Lateral Movement (Pohyb v síti)

## 🏗️ Pivoting & Tunneling
Technika, jak se dostat do "neviditelných" částí sítě.
- **Proxychains:** Nástroj pro směrování provozu skrze kompromitovaný hostitel.
- **SSH Tunneling:** Vytvoření šifrovaného tunelu pro skenování vnitřní sítě.

## 🔑 Techniky útoků na identitu
| Technika | Popis |
| :--- | :--- |
| **Pass-the-Hash** | Přihlášení pomocí NT hashe bez znalosti čistého hesla. |
| **Pass-the-Ticket** | Zneužití Kerberos lístků ve Windows doméně. |
| **Responder** | LLMNR/NBT-NS poisoning pro zisk hashů z lokální sítě. |

## 🛡️ Ochrana (Co doporučit klientovi)
- **Network Segmentation:** Rozdělení sítě na malé izolované kusy (VLAN).
- **Least Privilege:** Uživatelé mají přístup jen k tomu, co nezbytně potřebují.
- **EDR/IDS:** Monitoring podezřelých pohybů (např. když se účetní snaží připojit k SQL serveru).

> [!IMPORTANT]
> **Lateral Movement** končí v momentě, kdy dosáhnete cíle (např. ovládnutí Domain Controllera) nebo vyčerpáte všechny možnosti v rámci daného rozsahu (**Scope**).


---

# 🧪 Proof-of-Concept (PoC)

## 🎯 Účel PoC
- **Validace:** Potvrzení, že zranitelnost existuje.
- **Reprodukce:** Umožnit vývojářům chybu nasimulovat a opravit.
- **Impact:** Ukázat reálný dopad na byznys (např. přístup k databázi klientů).

## 🔗 Attack Chain (Řetězec útoku)
Nestačí ukázat jednu chybu. Nejlepší PoC ukazuje, jak se chyby řetězí:
1. **Vstup:** SQL Injection na webu.
2. **Přístup:** Zisk shellu jako `www-data`.
3. **Escalation:** Zneužití špatných práv k souboru `/etc/shadow`.
4. **Cíl:** Úplné ovládnutí serveru (Root).

## 🛠️ Remediation (Náprava)
- Nezaměřovat se jen na "rozbití" PoC skriptu.
- Řešit **kořenovou příčinu** (Root Cause).
- *Příklad:* Místo resetování hesla jednoho admina raději zavést **MFA** (vícefaktorové ověřování) a přísnější politiku hesel.

> [!TIP]
> **Show, don't just tell.** Video nahrávka nebo série screenshotů s časovým razítkem a IP adresou je pro klienta mnohem přesvědčivější než suchý text.


---

# 🏁 Post-Engagement & Reporting

## 🧹 Cleanup Checklist
- [ ] Smazány všechny nahrané nástroje (`/tmp`, `C:\Temp`).
- [ ] Smazáni vytvoření uživatelé a změněná hesla.
- [ ] Ukončeny všechny běžící reverzní shelly a procesy.
- [ ] Odstraněny záznamy v `authorized_keys` nebo `cron`.

## 📄 Struktura Reportu
1. **Executive Summary:** Pro vedení (byznys dopad).
2. **Technical Summary:** Pro IT správce (přehled zranitelností).
3. **Attack Chain:** Jak se chyby spojily v jeden velký průnik.
4. **Detailed Findings:** Screenshoty + kroky k reprodukci.
5. **Appendices:** Seznam všech skenovaných IP, otevřených portů atd.

## ⚖️ Etika a integrita
- **Nestrannost:** Pentester chyby hledá, ale neopravuje.
- **DRAFT vs FINAL:** Klient nejdříve dostane pracovní verzi (Draft), po zapracování jeho připomínek a retestu dostane finální verzi.

> [!IMPORTANT]
> **Soft Skills:** Klient si nebude pamatovat váš geniální exploit, ale to, jak jste s ním komunikovali, zda jste byli profesionální a zda mu vaše zpráva skutečně pomohla zlepšit bezpečnost.


---

# 🛡️ Master Methodology: Penetration Testing

## 1. Pre-Engagement (Příprava)
- **Dokumenty:** NDA, SoW, RoE.
- **Schůzky:** Scoping, Kick-off.
- **Cíl:** Definovat hranice (Scope) a pravidla.

## 2. Information Gathering (Průzkum)
- **OSINT:** Veřejné zdroje (GitHub, LinkedIn).
- **Enumerace:** Mapování sítě, služeb a verzí (Nmap).

## 3. Vulnerability Assessment (Analýza)
- **Výzkum:** Hledání CVE a exploitů.
- **Prioritizace:** Hodnocení pravděpodobnosti úspěchu vs. rizika poškození.

## 4. Exploitation (Průnik)
- **Foothold:** Získání prvního přístupu do systému.
- **Customization:** Úprava PoC kódů pro konkrétní prostředí.

## 5. Post-Exploitation (Uvnitř)
- **Persistence:** Udržení přístupu.
- **PrivEsc:** Zvýšení práv na Root/SYSTEM.
- **Pillaging:** Sběr citlivých dat (hesla, konfigurace).

## 6. Lateral Movement (Pohyb v síti)
- **Pivoting:** Použití stroje jako tunelu do vnitřní sítě.
- **AD Attacks:** Pass-the-Hash, Responder, Kerberoasting.

## 7. Reporting & Post-Engagement (Ukončení)
- **PoC:** Důkaz zranitelnosti (screenshoty/video).
- **Cleanup:** Smazání všech stop a nástrojů.
- **Final Report:** Předání zprávy a retest po opravách.

> [!QUOTE]
> **Think Outside The Box!** Technologie se mění, ale metodologie a kritické myšlení zůstávají základem úspěchu.

