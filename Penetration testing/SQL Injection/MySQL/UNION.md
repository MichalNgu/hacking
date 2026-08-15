## 🔗 Spojování dotazů pomocí klauzule `UNION`

Operátor `UNION` v SQL slouží k **kombinování výsledků ze dvou nebo více samostatných příkazů `SELECT`** do jednoho společného výstupu. Při SQL Injection umožňuje útočníkovi k původnímu dotazu připojit vlastní `SELECT` a načíst libovolná data z databáze (např. z jiných tabulek či databází).

SQL

```
-- Příklad spojení dvou tabulek se stejnou strukturou:
SELECT * FROM ports UNION SELECT * FROM ships;
```

## ⚠️ Dvě základní pravidla pro fungování `UNION`

Při použití operátoru `UNION` musí injektovaný dotaz striktně dodržet dvě pravidla databázového stroje:

### 1. Stejný počet sloupců (Equal Number of Columns)

Oba spojované příkazy `SELECT` **musí vracet přesně stejný počet sloupců**.

Pokud se počet sloupců liší, databáze vrátí chybu:

`ERROR 1222 (21000): The used SELECT statements have a different number of columns`

### 2. Odpovídající datové typy (Data Types Matching)

Sloupce na odpovídajících pozicích (např. 1. sloupec prvního dotazu a 1. sloupec druhého dotazu) musejí mít kompatibilní datové typy (číslo, řetězec, datum...).

## 🧩 Řešení rozdílného počtu sloupců (Výplňová data / Junk Data)

Pokud má původní tabulka více sloupců než data, která chceme získat, zbývající pozice v injektovaném `SELECT` dotazu musíme doplnit tzv. **výplňovými daty (junk data)**.

### Možnosti doplnění sloupců:

- **Čísla (např. `1, 2, 3...`):** Vhodná pro sledování pozic sloupců ve výstupu na stránce.
    
- **Textové řetězce (např. `'junk'`):** Vloží konstantní text.
    
- **`NULL`:** Hodnota `NULL` je nejuniverzálnější, protože je kompatibilní se všemi datovými typy v SQL.
    

## 💡 Příklady injekce

### Příklad: Původní dotaz očekává 2 sloupce

Chceme získat pouze uživatelská jména (`username`) z tabulky `passwords`. Druhý sloupec doplníme číslem `2`:

SQL

```
SELECT * FROM products WHERE product_id = '1' UNION SELECT username, 2 FROM passwords-- -
```

### Příklad: Původní dotaz očekává 4 sloupce

Pokud původní tabulka obsahuje 4 sloupce, doplníme zbývající tři pozice čísly `2, 3, 4`:

SQL

```
SELECT * FROM products WHERE product_id = '1' UNION SELECT username, 2, 3, 4 FROM passwords-- -
```

#### Výsledek v databázi:

|**Sloupec 1**|**Sloupec 2**|**Sloupec 3**|**Sloupec 4**|
|---|---|---|---|
|_produkt_1_|_produkt_2_|_produkt_3_|_produkt_4_|
|**admin**|**2**|**3**|**4**|

Hodnota `username` se zobrazí v prvním sloupci a výplňová čísla obsadí zbývající pozice.