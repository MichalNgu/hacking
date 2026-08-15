Při blokování kritických znaků jako lomítka (`/`, `\`) nebo středníku (`;`) je k dispozici technika jejich rekonstrukce **z existujících proměnných prostředí (Environment Variables)** nebo pomocí **dynamické ASCII manipulace**.

### 🐧 Linux: Substringing z proměnných prostředí

Syntaxe `${PROMENNA:START:DELKA}` umožňuje vyříznout konkrétní znak z jakékoliv systémové proměnné:

| **Znak** | **Zdrojová proměnná**    | **Linux Payload**   | **Princip / Popis**                                                   |
| -------- | ------------------------ | ------------------- | --------------------------------------------------------------------- |
| **`/`**  | `$PATH`, `$PWD`, `$HOME` | `${PATH:0:1}`       | První znak proměnné `$PATH` (typicky `/usr/bin...`) je lomítko `/`.   |
| **`;`**  | `$LS_COLORS`             | `${LS_COLORS:10:1}` | Proměnná `LS_COLORS` obsahuje formátovací řetězce oddělené středníky. |
|          | `$IFS`                   | `${IFS}`            | Výchozí oddělovač polí v Unixu (Internal Field Separator).            |

> 💡 **Tip:** Seznam všech proměnných a jejich hodnot získáte příkazem `printenv`.

### 🪟 Windows: Substringing v CMD a PowerShellu

V prostředí Windows se používá buď ořezávání řetězců v `cmd.exe`, nebo indexování polí v PowerShellu:

#### 1. Windows Command Line (`cmd.exe`)

- **Lomítko `\`:** `%HOMEPATH:~6,-11%`
    
    - _Vysvětlení:_ Proměnná `%HOMEPATH%` obsahuje např. `\Users\htb-student`. Syntaxe vytáhne podřetězec od 6. znaku s negativním koncem `-11`, čímž izoluje znak `\`.
        

#### 2. Windows PowerShell

V PowerShellu lze k řetězci přistupovat jako k poli znaků podle indexu:

- **Lomítko `\` z HOMEPATH:** `$env:HOMEPATH[0]`
    
- **Lomítko `\` z Program Files:** `$env:PROGRAMFILES[10]`
    

### 🔄 Technika ASCII Character Shifting (Posun znaků)

Pokud jsou blokovány i proměnné prostředí, lze chybějící znak vygenerovat posunem ASCII hodnoty předcházejícího znaku (pomocí příkazu `tr`):

Bash

```
# Vygenerování zpětného lomítka '\' (ASCII 92) z hranaté závorky '[' (ASCII 91):
echo $(tr '!-}' '"-~'<<<[)
```

#### 🧩 Řešení cvičení: Vygenerování středníku `;`

1. V ASCII tabulce má středník `;` hodnotu **59**.
    
2. Znak přímo před ním je dvojtečka `:` s hodnotou **58**.
    
3. **Payload pro vygenerování středníku `;`:**
    
    Bash
    
    ```
    echo $(tr '!-}' '"-~'<<<:)
    ```