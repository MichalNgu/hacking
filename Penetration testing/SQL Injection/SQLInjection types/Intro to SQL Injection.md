## 🌐 Použití SQL ve webových aplikacích

Webové aplikace (např. v PHP) běžně propojí databázi s logikou aplikace tak, že uživatelské vstupy vkládají přímo do SQL dotazů.

### Příklad zranitelného PHP kódu:

PHP

```
$searchInput = $_POST['findUser'];
// Vstup uživatele je přímo spojen s SQL dotazem bez ošetření:
$query = "select * from logins where username like '%$searchInput'";
$result = $conn->query($query);
```

Pokud aplikace uživatelský vstup neošetří, útočník ho může zneužít k vystoupení z určených mezí datového řetězce.

## 💉 Co je to Injection a jak vzniká SQLi?

- **Sanitizace (Sanitization):** Proces odstranění nebo zakódování nebezpečných řídicích znaků (jako `'`, `"`, `;`, `--`), čímž se zabrání změně struktury SQL dotazu.
    
- **SQL Injection (SQLi):** Nastává, když aplikace interpretuje vstup uživatele jako **spustitelný SQL kód** namísto pouhého textového řetězce.
    

### Mechanismus úniku z dotazu:

Vložením jednoduché uvozovky (`'`) útočník ukončí původní textový řetězec a zbytek vstupu je databázovým strojem zpracován jako kód:

SQL

```
-- Původní dotaz pro vstup "admin":
select * from logins where username like '%admin'

-- Injektovaný dotaz (vstup: 1'; DROP TABLE users;):
select * from logins where username like '%1'; DROP TABLE users;'
```

## ⚠️ Syntaktické chyby (Syntax Errors)

Při spuštění nahoře uvedeného příkladu databáze vrátí syntaktickou chybu:

`Error: near line 1: near "'": syntax error`

Chyba vzniká kvůli **neuzavřené koncové uvozovce (`'`)**, která zbyla z původního dotazu.

Aby byl útok úspěšný, musí být pozměněný SQL dotaz syntakticky platný. Toho se v praxi dosahuje nejčastěji:

1. **Databázovými komentáři** (odříznou zbytek původního dotazu).
    
2. **Doplněním a vyvážením uvozovek** a logických operátorů.
    

## 🧩 Typy SQL Injekcí

SQL injections se dělí podle způsobu a místa, kde útočník získává výstup z injektovaného dotazu:

```
                      ┌───────────────────────────────┐
                      │    Typy SQL Injekcí (SQLi)    │
                      └───────────────┬───────────────┘
                                      │
     ┌────────────────────────────────┼────────────────────────────────┐
     ▼                                ▼                                ▼
  In-band                         Blind                           Out-of-band
(Využívá stejný kanál)         (Nedirektivní / Slepá)         (Využívá externí kanál)
  ├─ UNION Based                 ├─ Boolean Based                └─ DNS / HTTP exfiltrace
  └─ Error Based                 └─ Time Based
```

|**Kategorie**|**Typ SQLi**|**Princip fungování**|
|---|---|---|
|**In-Band** _(přímý výstup)_|**UNION-Based**|Používá operátor `UNION` k připojení vlastního dotazu. Výsledek se zobrazí přímo na webové stránce.|
||**Error-Based**|Záměrně vyvolá databázovou chybu, která v chybové zprávě na front-endu vrátí požadovaná data.|
|**Blind** _(slepá SQLi)_|**Boolean-Based**|Vyhodnocuje logické podmínky (`TRUE`/`FALSE`), které mění vzhled nebo obsah načtené stránky.|
||**Time-Based**|Vyhodnocuje logické podmínky na základě časové prodlevy odpovědi serveru (funkce `SLEEP()`).|
|**Out-of-Band**|**OOB SQLi**|Používá se, pokud není přímý výstup. Data jsou odeslána na externí server útočníka (např. přes DNS dotaz).|