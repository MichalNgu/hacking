Černé listiny (_Blacklists_) ověřují příponu nahrávaného souboru vůči seznamu zakázaných hodnot. Tento přístup je z bezpečnostního hlediska konceptuálně slabý, protože málokdy obsahuje všechny spustitelné alternativní přípony a často nepočítá s odlišnostmi v konfiguraci webového serveru.

**Slabiny černých listin a způsoby jejich obcházení**

|**Slabina / Chyba v logice**|**Princip obcházení**|**Příklad**|
|---|---|---|
|**Opomenuté přípony**|Použití alternativních přípon, které interpret PHP na serveru stále zpracuje jako spustitelný kód.|`.phtml`, `.php3`, `.php4`, `.php5`, `.pht`, `.phar`, `.pgif`|
|**Velikost písmen (Case Sensitivity)**|Na systémech s nerozlišováním velikosti písmen (Windows / necitlivé souborové systémy).|`.pHp`, `.PhP`, `.pHTML`|
|**Nerekurzivní odstraňování**|Pokud filtr pouze jednou odstraní zakázaný řetězec `php`, složený název se propojí do platné přípony.|`.p.phphp` $\rightarrow$ po vymazání `php` vznikne `.php`|

**Metodika automatizovaného testování přípon (Burp Intruder)**

1. **Zachycení požadavku:** Zachyťte HTTP požadavek na nahrání souboru (`/upload.php`) v Burp Suite Proxy a odešlete jej do záložky **Intruder**.
    
2. **Definice pozice:** Vyčistěte automatické pozice a označte pouze příponu v parametru `filename="shell.php"`.
    
3. **Nahrání wordlistu:** V záložce _Payloads_ načtěte seznam známých přípon pro daný framework (např. z repozitáře _SecLists_ nebo _PayloadsAllTheThings_).
    
4. **Vypnutí URL Encoding:** Vypněte možnost _URL Encode these characters_, aby tečka a speciální znaky v příponách zůstaly v původním tvaru.
    
5. **Analýza odpovědí:** Spusťte test a seřaďte výsledky podle délky odpovědi (`Content-Length`) nebo HTTP stavu. Přípony s odlišnou délkou odpovědi indikují, že prošly filtrem.
    

**Bezpečnostní doporučení**

Nahraďte černé listiny **striktním Whitelistem** na straně serveru, který povoluje výhradně očekávané formáty (např. pouze `['jpg', 'jpeg', 'png']`). Všechny ostatní přípony musí být automaticky zamítnuty.