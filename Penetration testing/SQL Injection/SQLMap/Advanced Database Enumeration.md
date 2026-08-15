## Přehled pokročilé enumerace v SQLMapu

| **Funkce / Cíl**         | **Příkaz / Přepínač**         | **Popis a využití**                                                               |
| ------------------------ | ----------------------------- | --------------------------------------------------------------------------------- |
| **Struktura DB**         | `--schema`                    | Získá kompletní schématickou strukturu (databáze, tabulky, sloupce, datové typy). |
| **Hledání tabulek**      | `--search -T <slovo>`         | Vyhledá tabulky podle klíčového slova (používá operátor `LIKE`).                  |
| **Hledání sloupců**      | `--search -C <slovo>`         | Vyhledá sloupce podle klíčového slova napříč všemi databázemi.                    |
| **Extrakce hesel**       | `--dump -D <db> -T <table>`   | Stáhne tabulku a při detekci hashů automaticky nabídne jejich prolomení.          |
| **Systémová hesla**      | `--passwords`                 | Získá a proláme hesla přímo z administrátorských tabulek DBMS (např. `root`).     |
| **Kompletní exfiltrace** | `--dump-all --exclude-sysdbs` | Stáhne všechna data ze všech uživatelských databází mimo systémové.               |

**Detailní breakdown klíčových postupů**

- **Analýza architektury databáze (`--schema`)**
    
    Generuje ucelený přehled o všech databázích, tabulkách a datových typech (`int`, `varchar`, `text`, `blob`). Šetří čas při orientaci v rozsáhlých systémech.
    
- **Cílené vyhledávání podle vzorů (`--search`)**
    
    - **Vyhledání uživatelských tabulek:** `sqlmap -u "URL" --search -T user`
        
    - **Vyhledání citlivých sloupců (hesla, tokeny, emaily):** `sqlmap -u "URL" --search -C pass`
        
- **Extrakce a prolamování hesel**
    
    - **Slovníkový útok:** Pokud SQLMap v databázi detekuje řetězce odpovídající hashům (MD5, SHA-1 atd.), spustí vícevláknový slovníkový útok.
        
    - **Vestavěné zdroje:** Obsahuje slovník s **1,4 milionu záznamů** a podporuje přes **31 hashovacích algoritmů**.
        
- **Systémové účty databáze (`--passwords`)**
    
    Místo dat z aplikačních tabulek cílí přímo na systémové tabulky DBMS obsahující přístupové údaje systémových uživatelů (např. `root@localhost` nebo `debian-sys-maint`).
    
- **Bezobslužný režim (`--all --batch`)**
    
    Spustí automatickou kompletní enumeraci celého systému bez nutnosti potvrzovat interaktivní dotazy. Výstupy se průběžně ukládají do CSV/HTML souborů na lokálním disku.