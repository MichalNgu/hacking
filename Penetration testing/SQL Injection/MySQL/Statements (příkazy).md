## 📝 SQL Příkazy (SQL Statements)

Pro práci s daty v databázi se používají standardní příkazy jazyka SQL. Ty se dělí na DDL (_Data Definition Language_ – struktura) a DML (_Data Manipulation Language_ – data).

### 1. `INSERT INTO` (Vkládání dat)

Příkaz `INSERT` slouží k přidávání nových řádků/záznamů do tabulky.

- **Vložení hodnot do všech sloupců:**
    
    SQL
    
    ```
    INSERT INTO logins VALUES (1, 'admin', 'p@ssw0rd', '2020-07-02');
    ```
    
- **Vložení hodnot do vybraných sloupců** (vynechá sloupce s `AUTO_INCREMENT` nebo `DEFAULT` hodnotou):
    
    SQL
    
    ```
    INSERT INTO logins (username, password) VALUES ('administrator', 'adm1n_p@ss');
    ```
    
- **Vložení více záznamů najednou:**
    
    SQL
    
    ```
    INSERT INTO logins (username, password) VALUES ('john', 'john123!'), ('tom', 'tom123!');
    ```
    

### 2. `SELECT` (Čtení a výběr dat)

Příkaz `SELECT` načítá data z databáze a je klíčovým stavebním kamenem pro útoky SQL Injection.

- **Výběr všech sloupců (`*` funguje jako žolík):**
    
    SQL
    
    ```
    SELECT * FROM logins;
    ```
    
- **Výběr konkrétních sloupců:**
    
    SQL
    
    ```
    SELECT username, password FROM logins;
    ```
    

### 3. `ALTER TABLE` (Úprava struktury tabulky)

Příkaz `ALTER` mění vlastnosti a sloupce existující tabulky.

|**Operace**|**SQL Syntaxe**|
|---|---|
|**Přidání sloupce**|`ALTER TABLE logins ADD newColumn INT;`|
|**Přejmenování sloupce**|`ALTER TABLE logins RENAME COLUMN newColumn TO newerColumn;`|
|**Změna datového typu**|`ALTER TABLE logins MODIFY newerColumn DATE;`|
|**Smazání sloupce**|`ALTER TABLE logins DROP newerColumn;`|

### 4. `UPDATE` (Úprava existujících záznamů)

Příkaz `UPDATE` mění hodnoty v již upložených řádcích. Bez klauzule `WHERE` by upravil **všechny** řádky v tabulce.

SQL

```
UPDATE logins SET password = 'change_password' WHERE id > 1;
```

_(Tento dotaz změní heslo na `'change_password'` u všech záznamů, jejichž `id` je větší než 1)._

### 5. `DROP` (Trvalé smazání)

Příkaz `DROP` nenávratně odstraní celou tabulku nebo databázi bez potvrzovacího dotazu.

SQL

```
DROP TABLE logins;     -- Smaže tabulku "logins"
DROP DATABASE users;   -- Smaže celou databázi "users"
```