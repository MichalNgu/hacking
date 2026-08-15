## 🛠️ Diagnostika a ladění chyb v SQLMapu

Při testování nebo konfiguraci složitějších HTTP požadavků může docházet k chybám. SQLMap nabízí několik přepínačů pro analýzu komunikace a nalezení příčiny problémů:

### 1. Zobrazení chyb databáze (`--parse-errors`)

Přepínač **`--parse-errors`** automaticky parsuje a vypisuje chybové hlášky databázového serveru (DBMS) přímo do konzole.

Bash

```
sqlmap -u "http://www.target.com/vuln.php?id=1" --parse-errors
```

- **Význam:** Umožňuje okamžitě rozpoznat syntaktické chyby v injektovaném dotazu (např. neuzavřené uvozovky nebo závorky) a přizpůsobit payload.
    

### 2. Uložení veškerého provozu do souboru (`-t`)

Parametr **`-t FILE`** uloží kompletní příchozí i odchozí HTTP provoz (požadavky i odpovědi) do textového souboru pro zpětnou kontrolu.

Bash

```
sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt
```

- **Význam:** Vhodné pro detailní manuální analýzu odeslaných hlaviček a odpovědí serveru.
    

### 3. Zvýšení podrobnosti výstupu (`-v`)

Přepínač **`-v VERBOSE`** nastavuje úroveň detailu vypisovaných informací (hodnoty **`0` až `6`**, výchozí je `1`).

- **`-v 3`:** Zobrazí odeslané HTTP požadavky.
    
- **`-v 4`:** Zobrazí i HTTP hlavičky.
    
- **`-v 6`:** Zobrazí kompletní odchozí i příchozí HTTP provoz v reálném čase přímo v terminálu.
    

Bash

```
sqlmap -u "http://www.target.com/vuln.php?id=1" -v 6 --batch
```

### 4. Směrování provozu přes Proxy (`--proxy`)

Pomocí parametru **`--proxy`** lze veškerou komunikaci SQLMapu přesměrovat přes lokalní proxy (např. Burp Suite nebo OWASP ZAP na `[http://127.0.0.1:8080](http://127.0.0.1:8080)`).

Bash

```
sqlmap -u "http://www.target.com/vuln.php?id=1" --proxy="http://127.0.0.1:8080"
```

- **Význam:** Umožňuje v Burp Suite sledovat všechny požadavky v záložce _HTTP history_, opakovat je v nástroji _Repeater_ nebo detailně zkoumat reakce serveru.