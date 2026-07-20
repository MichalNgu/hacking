# .well-known URI (Web Recon)

Cíl:

- najít skryté konfigurační endpointy
- zjistit používané autentizační systémy
- odhalit API a infrastrukturu

Adresář:

```
https://target.com/.well-known/
```

---

# 📂 Důležité soubory

|Soubor|Co odhaluje|
|---|---|
|security.txt|Kontakt na security tým, bug bounty|
|openid-configuration|OAuth/OpenID endpointy|
|assetlinks.json|Vazba web ↔ Android aplikace|
|mta-sts.txt|Nastavení bezpečnosti e-mailů|
|change-password|Endpoint pro změnu hesla|

---

# 🔐 OpenID Connect Discovery

Pokud existuje:

```
/.well-known/openid-configuration
```

Dostaneš JSON s informacemi o autentizaci.

Hledat:

|Položka|Význam|
|---|---|
|authorization_endpoint|Login endpoint|
|token_endpoint|Získání tokenů|
|jwks_uri|Veřejné JWT klíče|
|scopes_supported|Dostupné informace o uživateli|

---

# Proč je to důležité

Může odhalit:

- skryté API
- OAuth konfiguraci
- JWT infrastrukturu
- externí Identity Provider:

```
Okta
Azure AD
Google Identity
```

---

# Praktické příkazy

Kontrola security.txt:

```
curl -i https://target.com/.well-known/security.txt
```

OpenID konfigurace:

```
curl -s https://target.com/.well-known/openid-configuration
```

Fuzzing:

```
gobuster dir \
-u https://target.com/.well-known/ \
-w wordlist.txt
```

---

# CPTS Footprinting Checklist

☐ `/ .well-known/security.txt`  
☐ `/ .well-known/openid-configuration`  
☐ OAuth endpointy  
☐ JWT/JWKS klíče  
☐ API endpointy  
☐ Identity provider  
☐ Další neznámé soubory