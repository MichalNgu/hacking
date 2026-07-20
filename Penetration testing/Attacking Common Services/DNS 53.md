# 🌐 DNS (53) – Domain Name System

DNS převádí doménová jména na IP adresy a obsahuje mnoho informací o infrastruktuře. Špatná konfigurace může odhalit servery, interní názvy, subdomény nebo umožnit manipulaci s překladem adres.

---

# 🔍 1. DNS Enumerace

První krok:

```
nmap -sC -sV -p 53 <IP>
```

Kontroluji:

- DNS software
- verzi
- dostupné funkce
- možnost zone transferu

---

# 📋 2. DNS Zone Transfer (AXFR)

Zone Transfer slouží k přenosu DNS záznamů mezi DNS servery.

Špatná konfigurace → útočník může získat kompletní DNS zónu.

Může odhalit:

- subdomény
- servery
- IP adresy
- MX záznamy
- interní názvy

---

## Test pomocí dig

```
dig AXFR @<DNS_SERVER> <DOMAIN>
```

Příklad:

```
dig AXFR @ns1.example.com example.com
```

Úspěch:

```
example.com.    A      10.10.10.10
dev.example.com A      10.10.10.20
mail.example.com MX    10.10.10.30
```

---

## Nmap AXFR kontrola

```
nmap --script dns-zone-transfer \
--script-args dns-zone-transfer.domain=<DOMAIN> \
-p 53 <DNS_SERVER>
```

---

# 🕵️ 3. DNS Enumeration (Subdomains)

Cíl:

- najít skryté služby
- objevit zapomenuté servery

Příklady:

```
dev.example.com
test.example.com
vpn.example.com
admin.example.com
```

---

## Subfinder

Pasivní enumerace:

```
subfinder -d example.com
```

Používá:

- veřejné databáze
- certifikáty
- API zdroje

---

## DNS brute-force

Pomocí wordlistu:

```
dnsenum example.com
```

nebo:

```
gobuster dns \
-d example.com \
-w subdomains.txt
```

---

# ☁️ 4. Subdomain Takeover

Vzniká při:

```
subdomain.example.com
        |
        |
        CNAME
        ↓
old-service.provider.com
```

Pokud služba již neexistuje:

```
CNAME → mrtvá služba
```

Útočník může:

1. zaregistrovat stejnou službu
2. převzít subdoménu
3. zobrazovat vlastní obsah

---

Typické cíle:

- AWS S3
- GitHub Pages
- Azure
- Heroku
- CloudFront

---

# 💉 5. DNS Spoofing / Cache Poisoning

Cíl:

- změnit DNS odpověď
- přesměrovat uživatele na falešný server

Normální:

```
user
 |
DNS query
 |
DNS server
 |
google.com → 142.x.x.x
```

Po útoku:

```
user
 |
DNS query
 |
útočník odpoví dříve
 |
google.com → IP útočníka
```

---

# 🕸️ 6. Lokální DNS Spoofing (MITM)

Používá se v lokální síti.

Nástroje:

- Bettercap
- Ettercap

---

## Ettercap workflow

### Úprava DNS pravidel

Soubor:

```
/etc/ettercap/etter.dns
```

Příklad:

```
target.com A 192.168.1.50
```

---

### Spuštění spoofingu

```
ettercap -T -q -i eth0
```

Aktivace pluginu:

```
dns_spoof
```

Výsledek:

Oběť zadá:

```
target.com
```

Dostane:

```
192.168.1.50
```

---

# 📧 7. DNS Records – důležité záznamy

|Záznam|Význam|Útočné využití|
|---|---|---|
|A|IPv4 adresa|nalezení serveru|
|AAAA|IPv6 adresa|alternativní útok|
|MX|Mail server|identifikace email infrastruktury|
|CNAME|Alias|Subdomain takeover|
|TXT|Textová data|SPF, DKIM, API klíče|
|NS|Name server|DNS infrastruktura|
|PTR|Reverse DNS|zjištění názvů zařízení|

---

# 🔎 8. Reverse DNS Lookup

Z IP zjistí název:

```
dig -x <IP>
```

Příklad:

```
dig -x 10.10.10.5
```

Výsledek:

```
dc01.domain.local
```

---

# 🛠️ Nástroje

|Nástroj|Účel|
|---|---|
|dig|DNS dotazy|
|nslookup|základní DNS test|
|dnsenum|DNS enumerace|
|fierce|automatická DNS enumerace|
|subfinder|hledání subdomén|
|gobuster dns|brute-force subdomén|
|bettercap|MITM DNS spoofing|
|ettercap|DNS spoofing|

---

# 🚩 Typický DNS Attack Flow

```
1. Nmap port 53
        ↓
2. Identifikace DNS serveru
        ↓
3. AXFR test
        ↓
4. Subdomain enumeration
        ↓
5. Kontrola CNAME záznamů
        ↓
6. Hledání takeover
        ↓
7. DNS spoofing / MITM (lokální síť)
        ↓
8. Další útok na objevené služby
```

---

# 📌 Shrnutí technik

|Technika|Potřeby|Výsledek|
|---|---|---|
|Zone Transfer|špatná konfigurace DNS|kompletní mapa infrastruktury|
|Subdomain Enumeration|DNS informace|nalezení skrytých služeb|
|Subdomain Takeover|mrtvý CNAME|převzetí subdomény|
|DNS Spoofing|MITM pozice|přesměrování provozu|