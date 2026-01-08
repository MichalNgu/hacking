
## 🛡️ . Firewally, IDS/IPS a obcházení (Evasion)

Firewall je tvůj "vrátný". Jako pentester musíš vědět, na jaké vrstvě tě blokuje a jak ho zmást.

### Typy Firewallů dle hloubky inspekce
| Typ | Vrstva OSI | Popis | Hacking perspektiva |
| :--- | :--- | :--- | :--- |
| **Packet Filtering** | 3. a 4. | Filtruje jen podle IP a portů. | Snadno se obchází fragmentací paketů. |
| **Stateful Inspection** | 3. až 5. | Pamatuje si stav spojení (ví, co je odpověď). | Těžší na oklamání, vyžaduje legitimní handshake. |
| **Application (WAF)** | 7. | Rozumí obsahu (např. HTTP požadavkům). | Blokuje SQLi, XSS. Nutná obfuskace payloadu. |
| **Next-Gen (NGFW)** | 3. až 7. | Obsahuje IDS/IPS, kontroluje i šifrovaný provoz. | Nejtěžší soupeř. Vyžaduje pokročilé techniky tunelování. |

### Stavy portů při skenování (Nmap)
* **Open:** Cesta je volná, služba naslouchá.
* **Closed:** Stroj odpověděl (RST), ale port je zavřený. Žádný firewall v cestě.
* **Filtered:** **Zde je firewall.** Paket byl zahozen (Drop) a nepřišla žádná odpověď, nebo přišlo ICMP "Prohibited".

### Techniky obcházení (Firewall Evasion)
* **Fragmentace:** Rozdělení útoku na miniaturní pakety (`nmap -f`), které firewall nesloží dohromady, ale cílový OS ano.
* **Decoy Scanning:** Skenování s falešnými IP adresami (`nmap -D RND:10`), které zamaskují tvou skutečnou adresu v logách.
* **Source Port Spoofing:** Nastavení zdrojového portu na **53 (DNS)** nebo **80 (HTTP)**, protože mnoho špatně nastavených firewallů těmto portům důvěřuje.
* **Tunelování:** Schování zakázaného protokolu (např. SSH) do povoleného (např. HTTP/HTTPS nebo DNS).

### IDS vs. IPS
* **IDS (Detection):** Detekuje útok a loguje ho (pasivní). *Tvůj cíl: Být potichu.*
* **IPS (Prevention):** Detekuje útok a okamžitě tě odpojí (aktivní). *Tvůj cíl: Nebýt odhalen po prvním paketu.*

---

## 🛠️ 7. Strategie Enumerace (Postup útoku)

Při příchodu k neznámé síti postupuj systematicky podle vrstev:

1.  **L2 (Linková):** Odposlech (Sniffing), hledání CDP paketů (Cisco info), ARP skenování.
2.  **L3 (Síťová):** Ping (pokud není blokován), zjištění TTL (identifikace OS), Traceroute (mapování cesty).
3.  **L4 (Transportní):** Port scan (TCP SYN scan), UDP scan služeb (DNS, SNMP).
4.  **L7 (Aplikační):** Banner Grabbing (zjištění verze SSH/HTTP), prohledávání webu, SMB enumerace.