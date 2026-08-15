### 1. Fuzzing přípon (Extension Fuzzing)

Před vyhledáváním konkrétních souborů je potřeba zjistit, jaké technologie a přípony aplikovaný web používá (např. `.php`, `.html`, `.aspx`).

- **Princip:** Jako název souboru se použije běžný základ (nejčastěji `index`) a zástupný symbol `FUZZ` se umístí na pozici přípony.
    
- **Slovník:** Používá se specializovaný slovník přípon (např. `web-extensions.txt`).
    

> **Upozornění:** Slovník `web-extensions.txt` v SecLists již tečku obsahuje (řádky začínají např. `.php`), proto se tečka mezi název a `FUZZ` nepíše.

#### Příkaz pro testování přípon:

Bash

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ
```

- **Vyhodnocení:** Přípona, která vrátí kód **`200 OK`** (např. `.php`), odhalí použitou technologii na serveru.
    

### 2. Fuzzing stránek (Page Fuzzing)

Jakmile znáte správnou příponu (např. `.php`), můžete vyhledávat skryté skripty a stránky v daném adresáři.

- **Princip:** Zástupný symbol `FUZZ` se umístí na pozici názvu souboru a na konec se natvrdo připojí zjištěná přípona `.php`.
    
- **Slovník:** Používá se standardní slovník pro discovery (např. `directory-list-2.3-small.txt`).
    

#### Příkaz pro vyhledávání PHP stránek:

Bash

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php
```

### Jak interpretovat výsledky:

- **Status `200 OK` + Size `0`:** Stránka existuje (např. `index.php`), ale nevrací žádný viditelný obsah.
    
- **Status `200 OK` + Size `> 0`:** Stránka existuje a obsahuje data/kód (např. vyhledaná skrytá stránka).
    
- **Status `403 Forbidden`:** Soubor na serveru existuje (např. `.phps`), ale přístup k němu je blokován konfigurací serveru.