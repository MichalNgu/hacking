# SSH Pivoting: Sshuttle & Plink

---

# SSHUTTLE (Linux Pivoting)

## Účel

**Sshuttle = VPN přes SSH**

Používá SSH spojení k pivot hostu a vytvoří transparentní routování do interní sítě.

Výhoda oproti Proxychains:

- není potřeba SOCKS proxy
- aplikace fungují normálně
- Nmap funguje bez omezení
- nemusíš upravovat konfiguraci každého nástroje

Schéma:

```
Kali
10.10.14.18

   |
   | SSH Tunnel
   |

Pivot Host
10.129.202.64

   |
   |

Internal Network
172.16.5.0/23
```

---

# 1. Instalace

Na Kali:

```
sudo apt install sshuttle
```

---

# 2. Vytvoření Pivot tunelu

Syntax:

```
sudo sshuttle -r user@pivot_ip target_network
```

Příklad:

```
sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23 -v
```

Parametry:

|Parametr|Význam|
|---|---|
|-r|SSH připojení na pivot|
|172.16.5.0/23|síť přes kterou pivotujeme|
|-v|verbose mód|

---

# 3. Kontrola spojení

Po spuštění:

```
route -n
```

nebo:

```
ip route
```

Měl by existovat route přes sshuttle.

---

# 4. Použití bez Proxychains

Po vytvoření tunelu:

## Nmap

Normálně:

```
nmap -sT -Pn -A 172.16.5.19
```

---

## RDP

Přímo:

```
xfreerdp /v:172.16.5.19 /u:user /p:password
```

---

## SMB

```
smbclient -L //172.16.5.19/
```

---

## Web

Prohlížeč:

```
http://172.16.5.19
```

---

# SSHuttle vs Proxychains

|Vlastnost|SSHuttle|Proxychains|
|---|---|---|
|Typ|VPN-like routing|SOCKS proxy|
|Konfigurace|jednoduchá|ruční|
|Nmap|ano|pouze -sT|
|ICMP|omezeně|ne|
|Rychlost|vyšší|nižší|
|Použití|celé sítě|jednotlivé aplikace|

---

# PLINK (Windows Pivoting)

## Účel

**Plink.exe = SSH klient z PuTTY**

Použití:

- pivot z Windows
- vytvoření SOCKS proxy
- přesměrování provozu přes SSH

Typický scénář:

```
Windows Pivot
        |
        |
      SSH
        |
        |
Linux Pivot
        |
        |
Internal Network
```

---

# 1. Dynamic SSH Forwarding přes Plink

Na Windows:

```
plink.exe -ssh -D 9050 ubuntu@10.129.15.50
```

Výsledek:

```
Windows localhost:9050

        |
        |
      SOCKS

        |
        |

SSH Pivot
```

---

# 2. Nastavení Proxifier

Windows nemá Proxychains.

Používá se:

**Proxifier**

Nastavení:

### Proxy Server:

```
127.0.0.1:9050
```

Typ:

```
SOCKS5
```

---

### Routing Rule:

Například:

```
172.16.5.0/23
```

posílat přes:

```
127.0.0.1:9050
```

---

# 3. Použití

Po nastavení:

RDP:

```
mstsc.exe
```

Cíl:

```
172.16.5.19
```

Proxifier automaticky:

```
mstsc.exe

↓

SOCKS Proxy

↓

Plink

↓

SSH Pivot

↓

Internal Host
```

---

# SSH Pivoting Přehled

|Situace|Nástroj|Důvod|
|---|---|---|
|SSH přístup na Linux pivot|sshuttle|nejpohodlnější řešení|
|Meterpreter session|autoroute + socks_proxy|bez SSH|
|Potřebuji SOCKS proxy|ssh -D / plink|univerzální|
|Reverse shell přes pivot|socat reverse|stabilní redirect|
|Blokovaný outbound provoz|socat bind|využití interní komunikace|
|Windows prostředí|plink + Proxifier|SSH pivot bez Linuxu|

---

# Pentester Workflow

```
1.
Získám přístup na pivot host

↓

2.
Zjistím síť

ip route
ifconfig

↓

3.
Vyberu techniku:

SSH účet?
    |
    +-- sshuttle

Pouze shell?
    |
    +-- Meterpreter routing

Windows pivot?
    |
    +-- Plink

Potřebuji reverse?
    |
    +-- Socat

↓

4.
Enumeruji interní síť

↓

5.
Lateral Movement
```

---

# Zapamatovat

**sshuttle**

> "Udělá z mého Kali člena interní sítě."

**ssh -D / Plink**

> "Vytvoří SOCKS proxy."

**Proxychains**

> "Pošle konkrétní program přes proxy."

**Socat**

> "Přesměruje konkrétní TCP spojení."

**Meterpreter autoroute**

> "Metasploit vlastní routing přes session."