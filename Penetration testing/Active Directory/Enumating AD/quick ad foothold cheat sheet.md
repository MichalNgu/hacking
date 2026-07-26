# 📖 AD Foothold Cheat Sheet (User Credentials → BloodHound → SMB → ACLs)

Když dostaneš doménového uživatele:

```
Username: john
Password: Password123!
Domain: corp.local
```

Postup je většinou:

```
1. Ověření přístupu
2. BloodHound
3. SMB Enumeration
4. ACL Enumeration
5. Kerberoast / AS-REP
6. LAPS / GPP / Shares
7. PrivEsc
```

---

# 🩸 BloodHound

## Windows (SharpHound)

Spustí default collection (ACL, Groups, Sessions, Trusts, Local Admins atd.):

```
SharpHound.exe
```

Nebo explicitně:

```
SharpHound.exe -c All
```

Výsledkem je ZIP pro BloodHound. SharpHound sbírá mimo jiné:

- ACL práva
- Group Memberships
- Trusts
- Sessions
- Local Admins
- GPO vztahy

---

## Linux

Místo starého bloodhound-python dnes většina lidí používá:

```
netexec ldap dc.corp.local -u john -p 'Password123!' --bloodhound
```

nebo

```
bloodhound-python \
-u john \
-p 'Password123!' \
-d corp.local \
-c All \
-ns 10.10.10.10
```

---

# 🔍 První BloodHound Queries

Po importu:

```
Find Shortest Paths to Domain Admins

Find Principals with DCSync Rights

Find Computers where User is Local Admin

Find Kerberoastable Users

Find AS-REP Roastable Users

Find Foreign Group Memberships

Find High Value Targets
```

---

# 📂 SMB Enumeration

## NetExec (nejlepší volba)

Dříve CrackMapExec.

### Vypsání share:

```
netexec smb 10.10.10.0/24 \
-u john \
-p 'Password123!' \
--shares
```

---

### Vypsání práv na share

```
netexec smb dc01 \
-u john \
-p 'Password123!' \
--shares
```

Ukáže:

```
READ
WRITE
READ/WRITE
```

---

### Spidering share

```
netexec smb dc01 \
-u john \
-p 'Password123!' \
-M spider_plus
```

---

### Hledání hesel

```
netexec smb dc01 \
-u john \
-p 'Password123!' \
-M spider_plus \
--pattern password
```

---

# 🔥 SMBMap

Jedna z nejlepších věcí na SMB práva.

```
smbmap -H dc01 \
-u john \
-p 'Password123!'
```

Výstup:

```
Share        Permissions

ADMIN$       NO ACCESS
HR           READ ONLY
IT           READ,WRITE
BACKUPS      FULL CONTROL
```

Perfektní pro CPTS.

---

# 🔥 PowerView ACL Enumeration

Hledání zajímavých ACL.

## Všechna ACL

```
Find-InterestingDomainAcl
```

---

## Práva na konkrétní objekt

```
Get-DomainObjectAcl -Identity "Domain Admins"
```

---

## Hledání GenericAll

```
Find-InterestingDomainAcl |
?{$_.ActiveDirectoryRights -match "GenericAll"}
```

---

# 🎯 Práva co hledáš

## GenericAll

```
Full Control
```

Můžeš skoro cokoliv.

---

## GenericWrite

```
Write atributů objektu
```

Často Shadow Credentials.

---

## WriteOwner

```
Převzetí vlastnictví
```

---

## WriteDACL

```
Úprava ACL
```

Přidání GenericAll sobě.

---

## AddMember

```
Přidání do skupiny
```

Například:

```
Helpdesk → Server Admins
```

---

## ForceChangePassword

```
Reset hesla bez znalosti původního
```

---

## DCSync

BloodHound hrana:

```
GetChanges

GetChangesAll

GetChangesInFilteredSet
```

=

```
DCSync
```

---

# ⚡ Rychlá LDAP Enumerace

## NetExec

```
netexec ldap dc01 \
-u john \
-p 'Password123!'
```

---

## Uživatelé

```
netexec ldap dc01 \
-u john \
-p 'Password123!' \
--users
```

---

## Skupiny

```
netexec ldap dc01 \
-u john \
-p 'Password123!' \
--groups
```

---

## LAPS

```
netexec ldap dc01 \
-u john \
-p 'Password123!' \
--laps
```

---

# 🏆 CPTS Quick Workflow

```
Valid user
   ↓
NetExec SMB --shares
   ↓
SMBMap
   ↓
BloodHound
   ↓
Find Local Admin
   ↓
Find ACL Abuse
   ↓
Kerberoast
   ↓
LAPS
   ↓
DCSync
   ↓
DA
```