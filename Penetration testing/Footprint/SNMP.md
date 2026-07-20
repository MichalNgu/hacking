# SNMP (Simple Network Management Protocol) – Port 161 UDP

SNMP je **síťový protokol pro monitoring a správu zařízení**.  
Používá model **dotaz → odpověď**.

Komunikace:

```
SNMP Manager
      |
      | Query (OID)
      ↓
SNMP Agent
      |
      | Response
      ↓
Manager
```

Data jsou organizována pomocí **OID (Object Identifier)**.

---

# SNMP Verze

|Verze|Bezpečnost|
|---|---|
|SNMPv1|Community string plaintext|
|SNMPv2c|Community string plaintext|
|SNMPv3|Autentizace + šifrování|

Nejčastější pentest cíl:

- **SNMPv1/v2c**
- slabý community string

---

# Enumerace

Cíl:

- získat informace o systému
- zjistit uživatele
- najít procesy
- objevit síťové rozhraní

---

# 1. Community String Brute Force

Community string funguje jako heslo.

Časté hodnoty:

```
public
private
manager
admin
```

---

## onesixtyone

```
onesixtyone \
-c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt \
TARGET
```

---

## Nmap

```
nmap -sU -p161 \
--script snmp-brute \
TARGET
```

---

## Metasploit

```
use auxiliary/scanner/snmp/snmp_login
```

---

# 2. SNMP Enumerace

Po získání community stringu:

Příklad:

```
public
```

---

## snmp-check

Nejpřehlednější výpis:

```
snmp-check \
-t TARGET \
-c public
```

Najde:

- uživatele
- procesy
- síťová rozhraní
- software

---

## snmpwalk

Kompletní výpis:

```
snmpwalk \
-v2c \
-c public \
TARGET .
```

---

# Zajímavé OID

## Informace o systému

```
1.3.6.1.2.1.1
```

Obsah:

- hostname
- OS
- uptime

---

## Spuštěné procesy

```
1.3.6.1.2.1.25.4.2.1.5
```

Hledat:

- hesla v parametrech
- skripty
- konfigurace

Příklad:

```
backup.sh --password Secret123
```

---

## Routing tabulka

```
1.3.6.1.2.1.4.21
```

Ukáže:

- další sítě
- další rozhraní
- možnosti pivotingu

---

# Útoky

---

# 1. SNMP Extend RCE

Pokud máš **write access**:

```
community = private
```

Můžeš zneužít SNMP Extend.

Princip:

```
SNMP Write
     |
     |
Přidání příkazu
     |
     |
Server spustí kód
```

Možný dopad:

- spuštění příkazů
- reverse shell

Nástroje:

- `snmpset`
- NET-SNMP

---

# 2. Information Disclosure

SNMP často odhalí:

- uživatele
- procesy
- software
- konfigurace
- síťové informace

Použití:

- další útoky
- exploitování verzí
- pivoting

---

# 3. Sniffing Community Stringu

SNMPv1/v2c neposílá community string šifrovaně.

Wireshark filtr:

```
snmp
```

Výsledek:

```
community: public
```

---

# 4. SNMPv3 Brute Force

SNMPv3 používá:

- username
- password
- hash
- šifrování

Nmap:

```
nmap \
--script snmp-brute \
--script-args snmp-brute.version=v3 \
TARGET
```

---

# SNMP Pentest Checklist

☐ Port 161 UDP otevřený  
☐ Verze SNMP  
☐ Community string brute force  
☐ `public/private` test  
☐ Výpis procesů  
☐ Výpis uživatelů  
☐ Software a verze  
☐ Síťová rozhraní  
☐ Routing tabulka  
☐ Write permissions

---

# HTB / CPTS SNMP Workflow

```
Nmap UDP Scan
      |
      ↓
Port 161
      |
      ↓
Find Community String
      |
      ↓
snmpwalk/snmp-check
      |
      ↓
Find Users / Processes / Passwords
      |
      ↓
Credentials Reuse
      |
      ↓
SSH / SMB / Web Access
```

---

# Nejčastější SNMP nálezy

|Finding|Dopad|
|---|---|
|Default community string|Únik informací|
|SNMP write access|Možné RCE|
|Plaintext community|Únik hesla|
|Citlivé procesy|Credentials|
|Odhalená síť|Pivoting|