## Obcházení ochranných prvků webových aplikací v SQLMapu

|**Mechanismus / Problém**|**Přepínač / Parametr**|**Popis a použití**|
|---|---|---|
|**Anti-CSRF tokeny**|`--csrf-token="nazev"`|Automaticky vyhledává a načítá čerstvé tokeny z odpovídajících stránek.|
|**Unikátní parametry**|`--randomize=param`|Pro každý požadavek vygeneruje náhodnou hodnotu vybraného parametru.|
|**Vypočítané parametry**|`--eval="python_kod"`|Spustí Python kód před odesláním (např. `--eval="import hashlib; h=hashlib.md5(id).hexdigest()"`).|
|**Anonymizace / Proxy**|`--proxy`, `--proxy-file`|Směruje provoz přes proxy server nebo seznam proxy adres.|
|**Síť Tor**|`--tor`, `--check-tor`|Automaticky připojí SQLMap k síti Tor a ověří funkční anonymní spojení.|
|**Blokování User-Agenta**|`--random-agent`|Nahradí výchozí SQLMap hlavičku náhodným prohlížečem (obchází základní pravidla WAF).|
|**Přeskočení detekce WAF**|`--skip-waf`|Přeskočí úvodní heuristické testy na přítomnost WAF pro snížení šumu v logách.|

**Klíčové Tamper skripty (`--tamper`)**

Tamper skripty upravují strukturu SQL payloadu těsně před odesláním. Lze je řetězit (např. `--tamper=between,randomcase`). Výpis všech dostupných skriptů zobrazíte pomocí `--list-tampers`.

| **Skript**                      | **Princip úpravy payloadu**                                                |
| ------------------------------- | -------------------------------------------------------------------------- |
| **`between`**                   | Nahradí `>` výrazem `NOT BETWEEN 0 AND #` a `=` výrazem `BETWEEN # AND #`. |
| **`randomcase`**                | Náhodně mění velikost písmen v SQL klíčových slovech (např. `SEleCt`).     |
| **`space2comment`**             | Nahradí mezery inline komentáři (`/**/`).                                  |
| **`space2plus` / `space2hash`** | Nahradí mezery znakem `+` nebo `#` s novým řádkem (`\n`).                  |
| **`equaltolike`**               | Nahradí všechny operátory `=` operátorem `LIKE`.                           |
| **`base64encode`**              | Zakóduje kompletní payload do formátu Base64.                              |
| **`versionedkeywords`**         | Obalí klíčová slova verzovanými komentáři MySQL (`/*!50000SELECT*/`).      |
|                                 |                                                                            |

**Pokročilé techniky obcházení WAF**

- **`--chunked` (Chunked Transfer Encoding):** Rozdělí tělo POST požadavku na malé části (_chunks_). Blokovaná SQL klíčová slova jsou rozsekána mezi jednotlivé bloky, takže projdou skrze WAF.
    
- **`--hpp` (HTTP Parameter Pollution):** Rozdělí payload mezi více parametrů se stejným názvem (např. `?id=1&id=UNION&id=SELECT...`), které kompatibilní backendy (např. ASP.NET) po doručení automaticky spojí.