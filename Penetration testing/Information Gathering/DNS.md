# DNS Enumeration (Information Gathering)

Cíl:

- zjistit infrastrukturu cíle
- najít subdomény
- objevit skryté služby
- získat informace pro další útoky

---

# 1. DNS Records Enumeration (dig)

Nástroj:

```
dig
```

|Příkaz|Účel|
|---|---|
|`dig domain.com A`|IPv4 adresa|
|`dig domain.com AAAA`|IPv6 adresa|
|`dig domain.com MX`|Mail servery|
|`dig domain.com NS`|DNS servery|
|`dig domain.com TXT`|TXT informace|
|`dig -x IP`|Reverse lookup|
|`dig +short domain.com`|Pouze výsledek|
|`dig +trace domain.com`|Celá DNS cesta|

---

# 2. DNS Zone Transfer (AXFR)

Cíl:

Získat kompletní DNS záznamy domény.

```
dig axfr @nameserver domain.com
```

Může odhalit:

- subdomény
- interní servery
- IP adresy
- názvy služeb

Automatizace:

```
dnsenum --enum domain.com
```

---

# 3. Subdomain Enumeration

Hledání:

```
dev.domain.com
test.domain.com
staging.domain.com
vpn.domain.com
```

---

## DNS Brute Force

### Gobuster

```
gobuster dns \
-d domain.com \
-w wordlist.txt
```

---

## DNSenum

```
dnsenum \
--enum domain.com \
-f wordlist.txt \
-r
```

Parametry:

|Parametr|Význam|
|---|---|
|`-f`|wordlist|
|`-r`|rekurzivní hledání|

---

# 4. Virtual Host Enumeration (VHost)

Hledání webů na stejné IP:

```
10.10.10.10

dev.site.com
admin.site.com
test.site.com
```

---

## Gobuster VHost

```
gobuster vhost \
-u http://IP \
-w wordlist.txt \
--append-domain
```

Používá HTTP:

```
Host: dev.site.com
```

---

# 5. Certificate Transparency Logs

Pasivní hledání subdomén.

Výhoda:

- žádný přímý kontakt se serverem
- často najde zapomenuté subdomény

Příklad:

```
curl -s \
"https://crt.sh/?q=domain.com&output=json" |
jq -r '.[].name_value' |
sort -u
```

Najde například:

```
dev.domain.com
vpn.domain.com
api.domain.com
```

---

# DNS Information Gathering Workflow

```
Domain
  |
  ↓
dig NS
  |
  ↓
Find Nameservers
  |
  ↓
Zone Transfer (AXFR)
  |
  ↓
Subdomain Enumeration
  |
  ├── dnsenum
  ├── gobuster dns
  └── crt.sh
  |
  ↓
VHost Enumeration
  |
  ↓
Web Enumeration
```

---

# CPTS Checklist – DNS Recon

☐ A/AAAA záznam  
☐ MX záznam  
☐ NS servery  
☐ TXT záznamy  
☐ AXFR test  
☐ Subdomain brute force  
☐ Certificate Transparency  
☐ VHost fuzzing  
☐ Interní názvy serverů