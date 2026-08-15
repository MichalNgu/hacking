## 🎛️ Řízení výsledků dotazu (Query Results Control)

Při provádění dotazů v SQL máme možnost výstup přesně řadit, omezovat jeho rozsah nebo vyhledávat konkrétní řádky podle zadaných kritérií.

### 1. Řazení výsledků (`ORDER BY`)

Kauzule `ORDER BY` řadí výsledky podle jednoho nebo více sloupců.

- **Vzestupné řazení (`ASC`):** Výchozí chování (od A po Z, od nejmenšího po největší).
    
    SQL
    
    ```
    SELECT * FROM logins ORDER BY password ASC;
    ```
    
- **Sestupné řazení (`DESC`):**
    
    SQL
    
    ```
    SELECT * FROM logins ORDER BY password DESC;
    ```
    
- **Řazení dle více sloupců:** Sekundární řazení pro případ duplicitních hodnot v prvním sloupci.
    
    SQL
    
    ```
    SELECT * FROM logins ORDER BY password DESC, id ASC;
    ```
    

> 💡 **Využití při SQL injection:** Příkaz `ORDER BY N` se běžně používá k **určení počtu sloupců** v původním SQL dotazu (zkouší se např. `ORDER BY 1`, `ORDER BY 2`..., dokud dotaz neselže).

### 2. Omezení počtu výsledků (`LIMIT`)

Kauzule `LIMIT` omezuje počet navrácených záznamů. Je klíčová zejména u Blind SQL injection nebo u dotazů vracejících velké množství řádků.

- **Získání prvních N řádků:**
    
    SQL
    
    ```
    SELECT * FROM logins LIMIT 2;
    ```
    
- **Použití s posunem (`LIMIT offset, count`):**
    
    SQL
    
    ```
    SELECT * FROM logins LIMIT 1, 2;
    ```
    
    _(Posun `1` přeskočí 1. řádek [číslování od 0] a vrátí následující `2` záznamy, tedy 2. a 3. řádek)._
    

### 3. Filtrování dat (`WHERE`)

Kauzule `WHERE` aplikuje podmínky na řádky před jejich navrácením.

- **Porovnání číselných hodnot:**
    
    SQL
    
    ```
    SELECT * FROM logins WHERE id > 1;
    ```
    
- **Porovnání řetězců:**
    
    SQL
    
    ```
    SELECT * FROM logins WHERE username = 'admin';
    ```
    
    _(Řetězce a data se vždy uzavírají do uvozovek `'` nebo `"`, čísla se píší přímo)._
    

### 4. Vyhledávání podle vzoru (`LIKE`)

Kauzule `LIKE` umožňuje hledat řetězce odpovídající určitému vzoru s využitím žolíkových znaků:

|**Žolíkový znak**|**Význam**|**Příklad**|
|---|---|---|
|**`%`**|Odpovídá **libovolnému počtu znaků** (0 nebo více).|`username LIKE 'admin%'` _(najde `admin`, `administrator` atd.)_|
|**`_`**|Odpovídá **právě jednomu** libovolnému znaku.|`username LIKE '___'` _(najde pouze 3místná jména, např. `tom`)_|