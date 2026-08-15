## 🛠️ Způsoby konfigurace HTTP požadavků v SQLMapu

Chybně nastavený požadavek (např. chybějící relační cookie, špatně formátovaná POST data nebo odhalitelný User-Agent) je nejčastější příčinou neúspěchu při detekci zranitelnosti. SQLMap nabízí několik flexibilních možností pro přesnou replikaci HTTP provozu:

### 1. Převod cURL příkazu z prohlížeče

Nejsnazší způsob, jak přenést kompletní požadavek přímo z vývojářských nástrojů (Developer Tools):

1. Otevřete záložku _Network_ v prohlížeči.
    
2. Klikněte pravým tlačítkem na požadavek $\rightarrow$ **Copy as cURL**.
    
3. V terminálu nahraďte slovo `curl` za `sqlmap`.
    

Bash

```
sqlmap 'http://www.example.com/?id=1' -H 'User-Agent: Mozilla/5.0...' -H 'Cookie: PHPSESSID=...'
```

### 2. Testování GET a POST dat

- **GET parametry (`-u`):** Předávají se přímo v URL.
    
    Bash
    
    ```
    sqlmap -u "http://www.example.com/page.php?id=1"
    ```
    
- **POST data (`--data`):** Používá se pro formulářová data.
    
    Bash
    
    ```
    sqlmap -u "http://www.example.com/" --data "uid=1&name=test"
    ```
    
- **Cílení na konkrétní parametr (`-p` nebo `*`):**
    
    - Pomocí parametru `-p`: `sqlmap -u "..." --data "uid=1&name=test" -p uid`
        
    - Pomocí hvězdičky `*` (custom injection marker): `sqlmap -u "..." --data "uid=1*&name=test"`
        

### 3. Načtení kompletního HTTP požadavku ze souboru (`-r`) – **Doporučeno**

Pro složité požadavky s mnoha hlavičkami nebo pro rozsáhlá POST data (včetně JSON/XML) je nejlepší uložit celý požadavek z **Burp Suite** do textového souboru (např. `req.txt`).

Bash

```
sqlmap -r req.txt
```

> 💡 **Tip pro hvězdičku (`*`):** Hvězdičku lze vložit na libovolné místo v souboru `req.txt` (např. do HTTP hlavičky, cesty URL nebo hodnoty JSON vlastnosti `{"id": "1*"}`), čímž SQLMapu přesně určíte místo pro injekci.

## ⚙️ Úprava hlaviček a obcházení detekce

|**Parametr**|**Popis**|**Příklad použití**|
|---|---|---|
|**`--cookie`**|Ruční nastavení HTTP Cookie|`sqlmap -u "..." --cookie="PHPSESSID=ab4530..."`|
|**`-H` / `--header`**|Přidání libovolné HTTP hlavičky|`sqlmap -u "..." -H="X-Forwarded-For: 127.0.0.1"`|
|**`--random-agent`**|Použije náhodný User-Agent (obchází základní WAF blokující řetězec `sqlmap`)|`sqlmap -u "..." --random-agent`|
|**`--mobile`**|Simuluje požadavek z mobilního zařízení|`sqlmap -u "..." --mobile`|
|**`--method`**|Vynutí jinou HTTP metodu (např. PUT, DELETE)|`sqlmap -u "..." --data="id=1" --method PUT`|

### 📦 Podpora JSON a XML dat

SQLMap automaticky rozpozná struktury **JSON** (např. `{"id": 1}`) i **XML** (např. `<id>1</id>`) zadané v poli `--data` nebo v souboru `-r`. Při spuštění pouze potvrdíte zpracování strukturovaného těla ([Y/n]).