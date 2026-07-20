# NETSH (Network Shell) – Windows Native Port Forwarding

## Co je Netsh

**Netsh (Network Shell)** je vestavěný Windows nástroj pro správu síťových funkcí.

V pentestu se používá hlavně jako **nativní port forwarder**, protože:

- není potřeba nahrávat externí nástroje (`socat`, `chisel`, `plink`),
- využívá legitimní Windows binárku (**Living off the Land**),
- může obejít omezení, kde EDR blokuje neznámé programy.

---

# 1. Kdy použít Netsh Portproxy

Použití:

✅ Máš administrátorský přístup na Windows stroji  
✅ Windows má přístup do interní sítě  
✅ Tvůj Kali stroj se do interní sítě nedostane přímo  
✅ Potřebuješ zpřístupnit jeden konkrétní port

Typický scénář:

```
Kali
 |
 |
Windows Pivot Host
10.129.15.150
 |
 |
Internal Network
172.16.5.0/23
 |
 |
Domain Controller
172.16.5.25
```

Windows pivot funguje jako **most mezi dvěma sítěmi**.

---

# 2. Vytvoření Port Forwardingu

Spouští se na Windows jako Administrator.

Syntax:

```
netsh interface portproxy add v4tov4 ^
listenaddress=<PIVOT_IP> ^
listenport=<LOCAL_PORT> ^
connectaddress=<TARGET_IP> ^
connectport=<TARGET_PORT>
```

---

## Příklad: Forward RDP na interní server

Cíl:

```
Internal Server:
172.16.5.25:3389
```

Pivot:

```
10.129.15.150
```

Vytvoření tunelu:

```
netsh.exe interface portproxy add v4tov4 ^
listenaddress=10.129.15.150 ^
listenport=8080 ^
connectaddress=172.16.5.25 ^
connectport=3389
```

Výsledek:

```
Kali
 |
 | 10.129.15.150:8080
 |
Windows Pivot
 |
 | 172.16.5.25:3389
 |
RDP Server
```

---

# 3. Význam parametrů

|Parametr|Význam|
|---|---|
|v4tov4|IPv4 → IPv4 forwarding|
|listenaddress|IP adresa pivotu, kam se připojíš|
|listenport|Lokální port otevřený na pivotu|
|connectaddress|Cílová interní IP|
|connectport|Cílový port služby|

---

# 4. Kontrola aktivních Forwardů

Zobrazení pravidel:

```
netsh interface portproxy show v4tov4
```

Příklad výstupu:

```
Listen on ipv4:

Address        Port
10.129.15.150  8080

Connect to ipv4:

Address        Port
172.16.5.25    3389
```

---

# 5. Připojení z Kali

Po vytvoření proxy se nepřipojuješ na interní IP.

Použiješ IP Windows pivotu:

```
xfreerdp /v:10.129.15.150:8080 /u:administrator /p:Password
```

Výsledek:

Připojíš se na:

```
10.129.15.150:8080
```

ale ve skutečnosti jsi na:

```
172.16.5.25:3389
```

---

# 6. Praktické využití v pentestu

## A) Přístup k Active Directory

Častý scénář:

```
Phishing
   |
Admin PC kompromitováno
   |
Admin PC vidí DC síť
   |
Netsh Pivot
   |
Kali → Domain Controller
```

Forward například:

### SMB

```
netsh interface portproxy add v4tov4 ^
listenport=445 ^
listenaddress=10.129.15.150 ^
connectport=445 ^
connectaddress=172.16.5.10
```

Potom:

```
smbclient -L \\10.129.15.150
```

---

### LDAP

```
netsh interface portproxy add v4tov4 ^
listenport=389 ^
listenaddress=10.129.15.150 ^
connectport=389 ^
connectaddress=172.16.5.10
```

Použití:

- BloodHound
- LDAP enumeration
- AD attack paths

---

# 7. Windows Firewall problém

Portproxy pouze vytvoří přesměrování.

Windows Firewall může stále blokovat příchozí spojení.

Kontrola:

```
netstat -ano | findstr 8080
```

Povolení portu:

```
netsh advfirewall firewall add rule ^
name="Pivot Port 8080" ^
protocol=TCP ^
dir=in ^
localport=8080 ^
action=allow
```

---

# 8. Smazání Port Forwardu

Po dokončení testu odstranit:

```
netsh interface portproxy delete v4tov4 ^
listenaddress=10.129.15.150 ^
listenport=8080
```

Kontrola:

```
netsh interface portproxy show v4tov4
```

---

# Netsh vs ostatní Pivot techniky

|Technologie|Výhoda|Nevýhoda|
|---|---|---|
|SSH -L|Jednoduché, stabilní|Potřebuje SSH|
|Chisel|Univerzální SOCKS|Externí binárka|
|Socat|Rychlé forwardy|Externí nástroj|
|Meterpreter portfwd|Bez SSH|Vyžaduje MSF session|
|Netsh|Windows native, nenápadné|Pouze port forwarding|

---

# Pentester workflow

```
Initial Access
      |
      ↓
Windows Host získán
      |
      ↓
Zjistit rozhraní

ipconfig

      |
      ↓
Najít interní síť

172.16.x.x

      |
      ↓
Admin práva?

Ano
 |
 ↓
netsh portproxy

      |
      ↓
RDP / SMB / LDAP / HTTP

      |
      ↓
Lateral Movement
```