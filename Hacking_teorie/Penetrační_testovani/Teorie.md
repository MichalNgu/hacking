
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


