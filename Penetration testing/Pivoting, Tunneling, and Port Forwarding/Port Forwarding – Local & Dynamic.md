# Port Forwarding – Local & Dynamic

Port forwarding umožňuje zpřístupnit služby z interní sítě přes kompromitovaný **pivot host**. Používá se hlavně při **pivotingu a lateral movementu**, když útočný stroj nemá přímý přístup k cílovému segmentu.

---

# 1. Local Port Forwarding (-L)

## Účel

Používám, když znám konkrétní:

- cílovou IP
- cílový port
- službu, ke které se potřebuji dostat

Typické použití:

- RDP
- SSH
- MySQL/MSSQL
- interní webové aplikace

## Princip

Vytvoří lokální port na mém stroji → provoz jde přes pivot → na interní cíl.

```
Attacker
   |
localhost:PORT
   |
 SSH Tunnel
   |
Pivot Host
   |
Internal Host:PORT
```

---

## Syntaxe

```
ssh -L [LOCAL_PORT]:[TARGET_IP]:[TARGET_PORT] user@pivot-host
```

---

## Příklad

Pivot:

```
10.129.202.64
```

Interní databáze:

```
172.16.5.19:3306
```

Vytvoření tunelu:

```
ssh -L 1234:172.16.5.19:3306 ubuntu@10.129.202.64
```

Výsledek:

```
127.0.0.1:1234
        |
        |
172.16.5.19:3306
```

Připojení:

```
mysql -h 127.0.0.1 -P 1234
```

---

# 2. Dynamic Port Forwarding (-D)

## Účel

Používám, když:

- neznám všechny cíle v síti
- potřebuji enumerovat interní síť
- chci používat více služeb přes jeden tunel

Vytváří **SOCKS proxy**.

---

## Princip

```
Attacker
    |
 SOCKS Proxy
    |
 Pivot Host
    |
 Internal Network
```

Veškerý provoz poslaný do proxy jde přes pivot.

---

## Vytvoření SOCKS proxy

```
ssh -D 9050 user@pivot-host
```

Výsledek:

```
SOCKS5 Proxy

127.0.0.1:9050
```

---

# Proxychains konfigurace

Soubor:

```
sudo nano /etc/proxychains4.conf
```

Přidat:

```
socks5 127.0.0.1 9050
```

---

# Použití přes proxychains

## Nmap

⚠️ SOCKS neumí SYN scan.

Špatně:

```
proxychains nmap -sS 172.16.5.19
```

Správně:

```
proxychains nmap -sT -Pn -n 172.16.5.19
```

Parametry:

|Parametr|Význam|
|---|---|
|-sT|TCP Connect scan|
|-Pn|ignoruje ping|
|-n|vypne DNS resolving|

---

## Kontrola portu

Pokud Nmap nefunguje:

```
proxychains nc -zv 172.16.5.19 445
```

---

## SMB přes pivot

```
proxychains smbclient -L //172.16.5.19/ -N
```

---

## Metasploit

```
proxychains msfconsole
```

---

# Local vs Dynamic Forwarding

|Vlastnost|Local (-L)|Dynamic (-D)|
|---|---|---|
|Cíl|Jeden port|Celá síť|
|Typ|Přímý tunel|SOCKS proxy|
|Rychlost|Vyšší|Nižší|
|Stabilita|Lepší|Horší|
|Použití|RDP, DB, web|Enumerace, více služeb|
|Nástroje|SSH klient|Proxychains|

---

# SSH Tunnel Background Mode

Pokud nechci shell:

```
ssh -D 9050 -N -f user@pivot-host
```

Parametry:

|Parametr|Funkce|
|---|---|
|-D|SOCKS proxy|
|-N|neotevírá shell|
|-f|běží na pozadí|

---

# Troubleshooting

## Nmap ukazuje stále filtered

Zkus:

```
proxychains nc -zv IP PORT
```

---

## Ping nefunguje

Normální chování.

Použij:

```
-Pn
```

---

## DNS problémy

Používej:

```
-n
```

Neboť DNS dotazy nejdou přes SOCKS proxy.

---

## Kontrola aktivních tunelů

```
ps aux | grep ssh
```

Ukončení:

```
kill PID
```

nebo:

```
killall ssh
```

---

# RDP přes Pivot (doporučený postup)

Pro RDP nepoužívat Proxychains → bývá pomalé.

Použít Local Forwarding:

```
Attacker
 |
127.0.0.1:44444
 |
Pivot
 |
Internal-RDP:3389
```

Tunel:

```
ssh -L 44444:172.16.5.19:3389 ubuntu@10.129.202.64
```

Připojení:

```
xfreerdp /v:127.0.0.1:44444 /u:user /p:password /cert:ignore
```

---

# Pentester Workflow

```
1. Získám shell na pivot hostu

2. Enumeruji síť:
   ipconfig / ifconfig
   route

3. Najdu interní segment:
   172.16.x.x

4. Vytvořím tunel:
   -L pro konkrétní službu
   -D pro průzkum celé sítě

5. Enumeruji:
   nmap
   smbclient
   ldapsearch
   crackmapexec/netexec

6. Provedu lateral movement
```