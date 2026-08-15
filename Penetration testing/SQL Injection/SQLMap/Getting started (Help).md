## 📖 Nápověda a základní volby (Help Listing)

SQLMap nabízí dvě úrovně nápovědy podle požadované podrobnosti:

- **Základní nápověda (`sqlmap -h`):** Zobrazí nejběžnější volby pro zadání cíle (`-u`), nastavení HTTP požadavků nebo základní úroveň vypisování informací (`-v`).
    
- **Rozšířená nápověda (`sqlmap -hh`):** Zobrazí kompletní přehled všech pokročilých parametrů (přímé připojení k DB `-d`, načtení logu z proxy `-l`, POST data `--data`, HTTP hlavičky `-H`, cookies `--cookie` atd.).
    

## ⚡ Základní scénář testování (Basic Scenario)

Pokud testujeme webovou aplikaci se zranitelným GET parametrem (např. `[http://www.example.com/vuln.php?id=1](http://www.example.com/vuln.php?id=1)`), spustíme automatickou detekci následujícím příkazem:

Bash

```
sqlmap -u "http://www.example.com/vuln.php?id=1" --batch
```

### Význam použitých parametrů:

- **`-u URL` (`--url`):** Určuje cílovou adresu včetně testovaného parametru.
    
- **`--batch`:** Automaticky odpovídá výchozí možností (`default`) na všechny interaktivní dotazy nástroje (např. zda přeskočit testy pro jiné databáze, zda rozšířit rozsah testů atd.). Vhodné pro skriptování a zrychlení práce.
    

## 🔍 Výstup detekce SQLMapu

Při úspěšném odhalení zranitelnosti SQLMap vypíše souhrnnou zprávu:

1. **Typy nalezených injekcí:** Nástroj automaticky otestuje a vypíše funkční vektory (např. _Boolean-based blind_, _Error-based_, _Time-based blind_, _UNION query_).
    
2. **Přesné Payloady:** Zobrazí konkrétní SQL kód použitý pro ověření zranitelnosti.
    
3. **Fingerprinting prostředí:** Určí přesnou verzi databázového serveru (např. `MySQL >= 5.0`) a technologii webového serveru (např. `PHP 5.2.6`, `Apache 2.2.9`).
    
4. **Logování:** Všechny nalezené výsledky a stažená data automaticky ukládá do adresáře `~/.sqlmap/output/CÍLOVÁ_DOMÉNA/`.