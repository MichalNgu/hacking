# Nmap Cheat Sheet

## Klíčové parametry pro Host Discovery

|Parametr|Popis|Kdy použít?|
|---|---|---|
|`-sn`|Zakáže skenování portů (Ping Scan).|Chceš jen rychle zjistit, které stroje jsou aktivní.|
|`-PE`|Vynutí ICMP Echo Request.|Testování, zda firewall propouští ping.|
|`-iL <soubor>`|Načte seznam IP adres ze souboru.|Máš připravený seznam cílů.|
|`-oA <název>`|Uloží výstup ve formátech Nmap, GNmap a XML.|Dokumentace a reportování.|
|`--packet-trace`|Zobrazí každý odeslaný a přijatý paket.|Diagnostika problémů se skenem.|
|`--reason`|Vypíše důvod, proč je host označen jako UP.|Analýza firewallů a filtrů.|

---

# Nmap Cheat Sheet: Od průzkumu po enumeraci

|Kategorie|Příkaz|Co dělá?|Kdy použít?|
|---|---|---|---|
|Host Discovery|`nmap -sn <IP/rozsah>`|Ping Scan bez skenování portů.|Zjištění aktivních hostů.|
|Host Discovery|`nmap -PE --disable-arp-ping <IP>`|Vynutí ICMP ping i v lokální síti.|Simulace vzdáleného útoku.|
|Skenování portů|`nmap -sS <IP>`|SYN Scan (Half-Open Scan).|Výchozí rychlý TCP scan.|
|Skenování portů|`nmap -sT <IP>`|TCP Connect Scan.|Pokud nemáš root oprávnění.|
|Skenování portů|`nmap -p- <IP>`|Skenuje všech 65 535 TCP portů.|Kompletní audit portů.|
|Skenování portů|`nmap -F <IP>`|Fast Scan (Top 100 portů).|Rychlý průzkum.|
|Enumerace|`nmap -sV <IP>`|Detekce verzí služeb.|Identifikace Apache, SSH, FTP apod.|
|Enumerace|`nmap -sC <IP>`|Spustí výchozí NSE skripty.|Základní automatizovaná enumerace.|
|Enumerace|`nmap -O <IP>`|Odhad operačního systému.|Rozlišení Windows/Linux.|
|UDP Scan|`nmap -sU <IP>`|Skenuje UDP porty.|DNS, DHCP, SNMP a další UDP služby.|
|Agresivní scan|`nmap -A <IP>`|Kombinace `-sV`, `-sC`, `-O` a traceroute.|Rychlá komplexní enumerace.|
|Diagnostika|`nmap --reason <IP>`|Zobrazí důvod stavu portu.|Analýza firewallů.|
|Ukládání|`nmap -oA <název> <IP>`|Uloží výstup do více formátů.|Pozdější analýza a reporting.|

---

# Nmap Evasion Cheat Sheet (Firewall / IDS / IPS)

## 1. Průzkum pravidel (Co firewall pustí?)

|Příkaz|Technika|Účel|
|---|---|---|
|`nmap -sA <IP>`|ACK Scan|Zjišťuje, zda je port filtered nebo unfiltered.|
|`nmap -sV --reason <IP>`|Reason Analysis|Zobrazí důvod odpovědi cíle.|
|`nmap -sU -p 53 <IP>`|UDP Scan|Testuje dostupnost UDP služeb.|

---

## 2. Maskování identity

|Příkaz|Technika|Účel|
|---|---|---|
|`-D RND:10`|Decoys|Přidá návnadové IP adresy.|
|`-S <IP_Address>`|IP Spoofing|Použije jinou zdrojovou IP adresu.|
|`-e <interface>`|Interface Selection|Určí síťové rozhraní.|
|`--spoof-mac 0`|MAC Spoofing|Vygeneruje náhodnou MAC adresu.|

---

## 3. Manipulace s pakety

|Příkaz|Technika|Účel|
|---|---|---|
|`-f`|Fragmentation|Rozdělí pakety na menší fragmenty.|
|`--mtu <číslo>`|Custom MTU|Nastaví velikost fragmentů.|
|`--data-length <n>`|Payload Append|Přidá náhodná data do paketu.|
|`--badsum`|Bad Checksum|Odesílá pakety s chybným checksumem.|

---

## 4. Zneužití důvěry

|Příkaz|Technika|Účel|
|---|---|---|
|`--source-port 53`|DNS Source|Zdrojový port 53 (DNS).|
|`--source-port 80`|HTTP Source|Zdrojový port 80 (HTTP).|
|`--source-port 443`|HTTPS Source|Zdrojový port 443 (HTTPS).|

---

## 5. Časování a ticho

|Příkaz|Technika|Účel|
|---|---|---|
|`-T0`|Paranoid|Extrémně pomalý scan.|
|`-T1`|Sneaky|Velmi pomalý scan.|
|`--scan-delay <čas>`|Custom Delay|Přidá pauzu mezi pakety.|

---

# Kombinace parametrů

|Parametr|Význam|
|---|---|
|`-sS`|SYN Scan|
|`-Pn`|Vypne Host Discovery|
|`-n`|Zakáže DNS překlady|
|`-T1`|Pomalé časování|
|`--source-port 53`|Zdrojový port 53|
|`-f`|Fragmentace paketů|
|`--data-length 24`|Přidání náhodných dat|

## Ukázkový příkaz

```bash
sudo nmap -sS -Pn -n -p 80,443 -T1 --source-port 53 -f --data-length 24 <IP>
```