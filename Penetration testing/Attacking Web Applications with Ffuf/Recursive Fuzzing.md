### Proč používat rekurzivní fuzzing?

Místo ručního spouštění příkazů pro každý nově nalezený adresář (a následně pro jeho podadresáře a soubory) dokáže `ffuf` celý proces **automatizovat**. Jakmile objeví novou složku, automaticky do ní zařadí další skenovací úlohu.

### Klíčové přepínače pro rekurzivní skenování

- **`-recursion`**: Zapne rekurzivní režim.
    
    > ⚠️ **Upozornění:** Příkaz pro `-u` musí v tomto režimu končit klíčovým slovem `FUZZ` (např. `http://SERVER_IP:PORT/FUZZ`).
    
- **`-recursion-depth <číslo>`**: Definuje maximální hloubku zanoření.
    
    - _Příklad (`-recursion-depth 1`):_ Prohledá hlavní adresáře a jejich přímé podadresáře (např. `/blog/admin`), ale nepůjde hlouběji (např. do `/blog/admin/users/`).
        
- **`-e <přípona>`**: Definuje příponu souborů (např. `-e .php`), která se automaticky zkouší u všech prohledávaných cílů.
    
- **`-v` (Verbose):** Vypisuje **plné URL adresy**. Bez tohoto přepínače je v rekurzivním výstupu těžké rozpoznat, pod který adresář konkrétní nalezený soubor patří.
    

### Praktický příkaz pro rekurzivní fuzzing

Bash

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```

### Jak číst rekurzivní výstup

1. **Přidávání úloh (Queueing):** Při nalezení adresáře uvidíte zprávu: `[INFO] Adding a new job to the queue: http://SERVER_IP:PORT/forum/FUZZ`
    
2. **Výpis celých URL díky `-v`:** `[Status: 200, Size: 0] | URL | http://SERVER_IP:PORT/blog/index.php`
    

> **Shrnutí:** Rekurzivní skenování pošle výrazně více požadavků a trvá déle (slovník se efektivně zdvojnásobí zkoušením variant s `.php` i bez), ale kompletně zmapuje celou strukturu aplikace v rámci jednoho jediného příkazu.