# ICMP Tunneling (Ping Tunneling) – ptunnel-ng

## Přehled

**ICMP tunneling** je technika pivotingu, která zapouzdří jiný síťový provoz do ICMP paketů (ping).

Použití:

- obcházení omezení firewallu
- komunikace ze sítě, kde je povolen pouze ICMP
- vytvoření skrytého tunelu mezi Kali a pivot hostem

Princip:

```
Kali
 |
 | ICMP Echo Request/Reply
 |
Pivot Host
 |
 |
Internal Network
```

Normálně:

```
TCP → Firewall blokuje
```

Po tunelu:

```
TCP
 ↓
ICMP Payload
 ↓
Firewall vidí pouze ping
```

---

# ptunnel-ng

## Použití

Použij pokud:

- SSH nefunguje
- HTTP/HTTPS je blokované
- DNS tunneling není možný
- jediná povolená komunikace je ICMP

Typický scénář:

```
Kali
 |
 ICMP Tunnel
 |
Ubuntu Pivot
 |
Internal Network
```

---

# 1. Kompilace ptunnel-ng

Na Kali vytvoříš binárku kompatibilní s cílovým systémem.

```
git clone https://github.com/utoni/ptunnel-ng.git

cd ptunnel-ng
```

Instalace závislostí:

```
sudo apt install automake autoconf -y
```

Build:

```
./autogen.sh
make
```

Výsledná binárka:

```
src/ptunnel-ng
```

Přenos na pivot:

```
scp ptunnel-ng user@pivot:/tmp/
```

---

# 2. Spuštění ptunnel-ng serveru

Na pivot hostu:

```
sudo ./ptunnel-ng
```

Server čeká na ICMP komunikaci.

Pro specifický forwarding:

```
sudo ./ptunnel-ng \
-r 10.129.202.64 \
-R 22
```

Parametry:

|Parametr|Význam|
|---|---|
|-r|IP pivot hostu|
|-R|cílový port po tunelu|

Příklad:

```
ICMP Tunnel
     |
     |
Pivot
     |
     |
SSH :22
```

---

# 3. Spuštění klienta na Kali

Vytvoření lokálního portu:

```
sudo ./ptunnel-ng \
-p 10.129.202.64 \
-l 2222 \
-r 10.129.202.64 \
-R 22
```

Výsledek:

```
localhost:2222

      |
      |
   ICMP Tunnel

      |
      |

Pivot SSH:22
```

---

# 4. SSH přes ICMP tunnel

Normálně:

```
ssh user@10.129.202.64
```

Přes tunnel:

```
ssh \
-p 2222 \
user@127.0.0.1
```

---

# 5. Vytvoření SOCKS proxy

SSH dynamic forwarding:

```
ssh \
-D 9050 \
-p 2222 \
user@127.0.0.1
```

Vznikne:

```
127.0.0.1:9050

       |
       |
   ICMP Tunnel

       |
       |
 Internal Network
```

---

# Proxychains konfigurace

Soubor:

```
sudo nano /etc/proxychains4.conf
```

Přidat:

```
[ProxyList]

socks5 127.0.0.1 9050
```

---

# Použití v interní síti

## Nmap

Používej:

```
proxychains nmap \
-sT \
-Pn \
-n \
172.16.5.19
```

Důvod:

|Parametr|Účel|
|---|---|
|-sT|TCP Connect scan|
|-Pn|bez ICMP discovery|
|-n|žádné DNS|

---

## RDP přes ICMP

Přímý port forward:

```
sudo ./ptunnel-ng \
-p 10.129.202.64 \
-l 8888 \
-r 172.16.5.19 \
-R 3389
```

Připojení:

```
xfreerdp \
/v:127.0.0.1:8888
```

Výsledek:

```
Kali
 |
 ICMP
 |
Ubuntu Pivot
 |
Windows RDP
```

---

# Praktické využití v pentestu

## 1. Obcházení izolovaných sítí

Scénář:

```
Internet

Kali

 |
 | pouze ICMP povoleno

Pivot Server

 |
 |
Internal VLAN
```

Řešení:

```
ptunnel-ng
```

---

# 2. Přístup do OT / SCADA sítí

Časté prostředí:

- průmyslové sítě
- výrobní systémy
- izolované VLAN

Situace:

- žádný internet
- žádný SSH
- pouze ping monitoring

ICMP tunneling může vytvořit komunikační kanál přes povolený protokol.

---

# 3. Přístup bez SSH služby

Výhoda:

Nepotřebuješ:

- SSH server
- uživatelský účet
- otevřené TCP porty

Stačí:

- možnost posílat ICMP
- práva pro spuštění ptunnel-ng

---

# Srovnání tunneling technik

|Technik|Kdy použít|
|---|---|
|SSH -D|Máš SSH účet|
|Chisel|Nemáš SSH, můžeš spustit binárku|
|Socat|Potřebuješ jednoduchý redirect|
|DNS tunneling|Povolen pouze DNS|
|ICMP tunneling|Povolen pouze ping|
|Netsh|Windows pivot bez uploadu nástrojů|

---

# Troubleshooting

## Tunnel nefunguje

Kontrola ICMP:

```
ping pivot-ip
```

Musí fungovat.

---

## Permission denied

ptunnel potřebuje raw socket:

```
sudo ./ptunnel-ng
```

---

## Pomalu přenáší data

Normální chování.

ICMP tunneling je:

- pomalejší
- náchylnější na ztrátu paketů
- vhodný spíše pro shell a menší provoz

---

# Detekce Blue Teamem

Moderní NDR systémy sledují:

## 1. Velikost ICMP paketů

Normální ping:

```
~64 bytes
```

Tunnel:

```
500-1500 bytes
```

---

## 2. Frekvence

Normální:

```
1 ping / sekundu
```

Tunnel:

```
stovky až tisíce paketů
```

---

## 3. Anomální komunikace

Například:

```
Server → Internet IP
ICMP nonstop několik hodin
```

= podezřelé.

---

# Pentester rozhodovací strom

```
Potřebuji pivot?

        |
        v

Mám SSH?
 |
 +-- Ano
 |      |
 |      sshuttle / SSH -D
 |
 +-- Ne
        |
        Můžu spustit binárku?
        |
        +-- Ano
        |       |
        |       Chisel
        |
        +-- Ne
                |
                Co je povolené?

                HTTP → rpivot
                DNS  → dnscat2
                ICMP → ptunnel-ng
                Windows native → netsh
```