### Základní koncept a klíčové slovo `FUZZ`

`ffuf` funguje na základě propojování slovníku (wordlistu) a cílové URL. Místo v adrese, které se má dynamicky nahrazovat slovy ze slovníku, označíme klíčovým slovem (zástupným symbolem) – nejčastěji se používá **`FUZZ`**.

### Hlavní přepínače příkazu

- **`-w <cesta_ke_slovniku>:KEYWORD`**: Určuje cestu k souboru se slovníkem a volitelné klíčové slovo.
    
    - _Příklad:_ `-w /path/to/wordlist.txt:FUZZ`
        
- **`-u <URL>`**: Cílová URL adresa s vloženým klíčovým slovem `FUZZ`.
    
    - _Příklad:_ `-u http://SERVER_IP:PORT/FUZZ`
        
- **`-t <počet>`**: Počet současně běžících vláken (threads). Výchozí hodnota je `40`.
    
    - _Upozornění:_ Zvýšení vláken (např. `-t 200`) scan sice zrychlí, ale u vzdálených serverů může způsobit přetížení (DoS) nebo zahltit vaše síťové připojení.
        

### Praktický příkaz pro fuzzing adresářů

Příklad spuštění enumerace adresářů pomocí slovníku ze `SecLists`:

Bash

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ
```

### Jak číst výstup `ffuf`

Výstup obsahuje důležité informace o nalezených cestách:

- **Status (HTTP kód):**
    
    - `200 OK`: Adresář existuje a je přístupný.
        
    - `301 / 302 Redirect`: Přesměrování (často značí existující složku, např. `/blog` ➔ `/blog/`).
        
    - `403 Forbidden`: Adresář existuje, ale nemáte k němu přístup.
        
    - `404 Not Found`: Neexistující složka (standardně se skrývá).
        
- **Size / Words / Lines:** Velikost odpovědi serveru (v bajtech, slovech a řádcích), což pomáhá identifikovat odchylky.