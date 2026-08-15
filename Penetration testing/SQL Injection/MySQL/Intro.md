## 💻 Příkazová řádka (Command Line Interface - CLI)

Pro připojení k databázi MySQL/MariaDB z terminálu se používá příkaz `mysql`.

### Zabezpečení a možnosti připojení:

- **Lokální přihlášení:**
    
    Bash
    
    ```
    mysql -u root -p
    ```
    
    _(Z bezpečnostních důvodů zadávejte heslo až po výzvě `Enter password:`. Pokud zadáte heslo přímo za přepínač `-p` (bez mezery), uloží se v čitelné podobě do historie bash shellu `~/.bash_history`)._
    
- **Vzdálené přihlášení:**
    
    Bash
    
    ```
    mysql -u root -h docker.hackthebox.eu -P 3306 -p
    ```
    
    - `-h` specifikuje vzdáleného hostitele/IP adresu.
        
    - `-P` (velké P) specifikuje port (výchozí port MySQL je **3306**).
        

## 🗄️ Správa databází a tabulek

SQL příkazy nejsou citlivé na velikost písem (jsou _case-insensitive_), ale názvy databází a tabulek ano. Konvencí je psát SQL klíčová slova **VELKÝMI PÍSMENY**. Všechny dotazy v rozhraní CLI musí být zakončeny středníkem `;`.

### Základní příkazy pro práci s databází:

SQL

```
CREATE DATABASE users;   -- Vytvoří novou databázi s názvem "users"
SHOW DATABASES;          -- Zobrazí seznam všech dostupných databází
USE users;               -- Přepne se do vybrané databáze
SHOW TABLES;             -- Zobrazí seznam tabulek v aktuální databázi
DESCRIBE logins;         -- Zobrazí strukturu tabulky (sloupce, datové typy, klíče)
```

## 🛠️ Vytvoření tabulky a vlastnosti sloupců (CREATE TABLE)

Při vytváření tabulky definujeme název sloupců, jejich datové typy (`INT`, `VARCHAR`, `DATETIME`) a doplňující vlastnosti (omezení/constraints):

|**Vlastnost / Omezení**|**Popis a význam**|
|---|---|
|**`AUTO_INCREMENT`**|Hodnota se automaticky zvýší o 1 při vložení každého nového řádku (vhodné pro ID).|
|**`NOT NULL`**|Sloupec nesmí zůstat prázdný.|
|**`UNIQUE`**|Hodnota ve sloupci musí být v rámci celé tabulky unikátní (např. uživatelské jméno).|
|**`DEFAULT`**|Nastaví výchozí hodnotu, pokud není zadána (např. `DEFAULT NOW()` vloží aktuální datum a čas).|
|**`PRIMARY KEY`**|Určuje unikátní identifikátor (primární klíč) pro každý záznam v tabulce.|

### Kompletní SQL dotaz pro vytvoření tabulky:

SQL

```
CREATE TABLE logins (
    id INT NOT NULL AUTO_INCREMENT,
    username VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(100) NOT NULL,
    date_of_joining DATETIME DEFAULT NOW(),
    PRIMARY KEY (id)
);
```