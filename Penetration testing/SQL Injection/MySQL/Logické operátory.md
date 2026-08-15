## ⚖️ Logické operátory v SQL

Logické operátory slouží k vyhodnocování složitějších podmínek v klauzuli `WHERE`. V prostředí MySQL/MariaDB vrací hodnota **`1`** stav _Pravda (TRUE)_ a hodnota **`0`** stav _Nepravda (FALSE)_.

### Přehled základních logických operátorů:

|**Operatórové slovo**|**Znakový ekvivalent**|**Význam / Funkce**|**Příklad v MySQL**|**Navrácená hodnota**|
|---|---|---|---|---|
|**`AND`**|**`&&`**|Pravda, pokud **obě** podmínky platí současně.|`SELECT 1 = 1 AND 'a' = 'a';`|`1` _(True)_|
|**`OR`**|**`||`**|Pravda, pokud platí **alespoň jedna** z podmínek.|
|**`NOT`**|**`!`** nebo **`!=`**|Invertuje hodnotu (_NOT TRUE_ $\rightarrow$ _FALSE_).|`SELECT NOT 1 = 1;`|`0` _(False)_|

## 🔍 Použití operátorů v dotazech

Logické operátory se nejčastěji kombinují v příkazu `SELECT` s klauzulí `WHERE` k přesnému vyfiltrování dat:

SQL

```
-- Vybere všechny uživatele, jejichž jméno NENÍ 'john'
SELECT * FROM logins WHERE username != 'john';

-- Vybere uživatele s ID větším než 1 A ZÁROVEŇ se jménem odlišným od 'john'
SELECT * FROM logins WHERE username != 'john' AND id > 1;
```

## 🔝 Priorita operátorů (Operator Precedence)

Pokud dotaz obsahuje více operací současně, vyhodnocují se v přesně stanoveném pořadí (od nejvyšší priority po nejnižší):

1. **Aritmetika (Násobení / Dělení):** `/`, `*`, `%`
    
2. **Aritmetika (Sčítání / Odčítání):** `+`, `-`
    
3. **Porovnání:** `=`, `>`, `<`, `<=`, `>=`, `!=`, `LIKE`
    
4. **Logické NE:** `NOT` (`!`)
    
5. **Logické A:** `AND` (`&&`)
    
6. **Logické NEBO:** `OR` (`||`)
    

### 💡 Příklad vyhodnocení priority:

SQL

```
SELECT * FROM logins WHERE username != 'tom' AND id > 3 - 2;
```

1. **První krok (Odčítání `-`):** Spočítá `3 - 2`, dotaz se zjednoduší na `id > 1`.
    
2. **Druhý krok (Porovnání `!=` a `>`):** Zkontroluje podmínky `username != 'tom'` a `id > 1`.
    
3. **Třetí krok (Spojení `AND`):** Navrátí pouze záznamy, které vyhovují oběma podmínkám současně.
    

> ℹ️ **Význam pro SQL Injection:** Pochopení priority operátorů (zejména `AND` vs. `OR`) je klíčové při sestavování zranitelných vstupů v přihlašovacích formulářích (např. `' OR 1=1 --`).