## 📐 1. Prefiksy a Sufiksy (`--prefix` a `--suffix`)

Při testování atypických SQL dotazů nemusí výchozí detekční vzory SQLMapu správně uzavřít stávající SQL syntaxi. Pomocí parametrů `--prefix` a `--suffix` lze ručně definovat obalovací znaky pro injektovaný kód (tzv. _vector_).

- **Příklad zranitelného dotazu v aplikaci:**
    
    PHP
    
    ```
    $query = "SELECT id, name FROM users WHERE id LIKE (('" . $_GET["q"] . "')) LIMIT 0,1";
    ```
    
- **Spuštění s vlastním prefixem/suffixem:**
    
    Bash
    
    ```
    sqlmap -u "http://www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
    ```
    
- **Výsledný vykonaný SQL dotaz:**
    
    SQL
    
    ```
    SELECT id, name FROM users WHERE id LIKE (('test%')) UNION ALL SELECT 1, VERSION()-- -')) LIMIT 0,1
    ```
    

## ⚙️ 2. Nastavení úrovně a rizika (`--level` a `--risk`)

Tento pár parametrů určuje šíři testovacích vzorů (vektorů) a obalovacích podmínek (_boundaries_).

|**Parametr**|**Rozsah**|**Výchozí**|**Popis**|
|---|---|---|---|
|**`--level`**|`1` až `5`|`1`|Rozšiřuje počet testovaných pozic (např. HTTP hlavičky jako `Cookie`, `User-Agent`, `Referer`) a zkouší komplexnější syntaktické struktury.|
|**`--risk`**|`1` až `3`|`1`|Zvyšuje riziko poškození databáze. Úroveň `2` a `3` přidává náročné testy (např. heavy `OR` podmínky), které mohou modifikovat data nebo způsobit DoS.|

> ⚠️ **Poznámka:** Zvýšení parametrů z výchozího stavu (`--level=1 --risk=1`) na maximum (`--level=5 --risk=3`) zvýší počet odesílaných testovacích payloadů z cca **72 až na více než 7 800** na jeden parametr, což výrazně prodlouží dobu skenování.

## 🎯 3. Přesné vyhodnocování odezvy (Advanced Tuning)

Pokud aplikace vrací velké množství dynamického obsahu, SQLMap vyžaduje upřesnění pro spolehlivé odlišení stavu `TRUE` a `FALSE`:

- **`--code=200`:** Fixuje vyhodnocení stavu `TRUE` pouze na konkrétní HTTP stavový kód (např. 200 vs 500).
    
- **`--string="success"`:** Určuje textový řetězec, který se vyskytuje **pouze** při pravdivé odezvě (`TRUE`).
    
- **`--titles`:** Instruuje vyhodnocovací engine, aby srovnával pouze obsah HTML tagu `<title>`.
    
- **`--text-only`:** Odstraní z HTML odpovědi všechny struktury a tagy (`<script>`, `<style>`) a srovnává pouze viditelný text.
    

## 🛠️ 4. Omezení technik a úprava UNION dotazů

### Omezení technik (`--technique`)

Umožňuje vynutit nebo přeskočit konkrétní typy injekcí. Označení využívá zkratky **BEUSTQ**:

Bash

```
# Testovat pouze Boolean-based (B), Error-based (E) a UNION (U) injekce:
sqlmap -u "http://www.example.com/?id=1" --technique=BEU
```

### Manuální ladění UNION injekce

- **`--union-cols=N`:** Ručně definuje přesný počet sloupců (vynechá automatické zjišťování přes `ORDER BY`).
    
- **`--union-char='a'`:** Nahradí výchozí výplňové hodnoty (`NULL` nebo čísla) zadaným znakem.
    
- **`--union-from=TABLE`:** Přidá klauzuli `FROM <table>` na konec `UNION` dotazu (nutné např. u databází Oracle, kde se používá `FROM dual`).