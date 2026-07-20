# DNS (Domain Name System) – Port 53

DNS převádí **doménová jména na IP adresy**. Funguje jako distribuovaná databáze s hierarchií:

```
Root
 ↓
TLD (.com, .cz)
 ↓
Domain
 ↓
Subdomain
```

Používá dva protokoly:

- **UDP 53**
    - běžné DNS dotazy
    - rychlá komunikace
- **TCP 53**
    - Zone Transfer (AXFR)
    - velké odpovědi

---

# DNS Enumerace

Cíl:

- zjistit IP adresy
- najít subdomény
- objevit interní systémy
- najít citlivé informace

---

## DNS Records

|Příkaz|Účel|
|---|---|
|`dig domain.htb A`|IPv4 adresa|
|`dig domain.htb MX`|Mail servery|
|`dig domain.htb NS`|Nameservery|
|`dig domain.htb TXT`|SPF, klíče, informace|
|`dig ANY domain.htb`|Všechny záznamy|

---

# Zone Transfer (AXFR)

Nejzajímavější DNS chyba.

Umožňuje stáhnout celou DNS zónu:

```
dig axfr domain.htb @DNS_SERVER
```

Pokud je špatná konfigurace:

Získáš:

- subdomény
- IP adresy
- interní názvy serverů

Příklad:

```
dev.domain.htb
vpn.domain.htb
dc.domain.htb
```

---

# Reverse DNS Lookup

Zjištění domény z IP:

```
dig -x 10.10.10.x
```

Použití:

- hledání názvů serverů
- mapování sítě

---

# Subdomain Enumeration

Pokud AXFR nefunguje:

## Gobuster DNS

```
gobuster dns \
-d domain.htb \
-w wordlist.txt \
-r DNS_SERVER
```

Hledá:

```
dev.domain.htb
test.domain.htb
admin.domain.htb
```

---

# Virtual Host Fuzzing

DNS záznam nemusí existovat, ale web může reagovat na Host header.

Nástroj:

```
ffuf -u http://IP \
-H "Host: FUZZ.domain.htb" \
-w wordlist.txt
```

Hledá skryté weby:

```
admin.domain.htb
staging.domain.htb
```

---

# DNS Spoofing / Poisoning

Síťový útok typu MitM.

Princip:

```
Oběť
 |
 | "Kde je banka.cz?"
 ↓
Útočník
 |
 | "Banka.cz = moje IP"
```

Nástroje:

- Bettercap
- Ettercap

Cíl:

- přesměrování uživatele
- phishing
- krádež přihlašovacích údajů

---
# DNS Footprinting Checklist

Hledej zajímavé názvy:

|Název|Význam|
|---|---|
|`dev`|Vývojové prostředí|
|`test`|Testovací systém|
|`staging`|Předprodukční server|
|`vpn`|VPN vstup|
|`gw`|Gateway|
|`firewall`|Síťová infrastruktura|
|`dc`|Domain Controller|
|`ad`|Active Directory|
|`kerberos`|AD autentizace|
|`old`|Starý systém|
|`temp`|Zapomenutý systém|

---
# HTB / CPTS DNS Workflow

```
Nmap
 ↓
Port 53
 ↓
DNS Enumeration
 ↓
dig records
 ↓
AXFR test
 ↓
Subdomain brute force
 ↓
VHost fuzzing
 ↓
Web/AD Enumeration
```

---
# Nejčastější DNS nálezy

|Finding|Dopad|
|---|---|
|Open Zone Transfer|Únik celé DNS databáze|
|Citlivé TXT záznamy|Únik klíčů/informací|
|Odhalené subdomény|Nové attack surface|
|Staré subdomény|Zapomenuté služby|
|Interní názvy serverů|Informace o infrastruktuře|

---
## CPTS poznámka
U DNS vždy kontroluj:

1. **AXFR**
2. **TXT záznamy**
3. **Subdomény**
4. **VHosty**
5. **Názvy indikující AD (`dc`, `kerberos`, `ldap`)**
