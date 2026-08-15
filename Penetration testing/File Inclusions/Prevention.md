Ochrana před zranitelnostmi File Inclusion (LFI/RFI) spočívá v zamezení přímého předávání uživatelského vstupu do souborových funkcí, striktním omezování povolených cest a správné konfiguraci prostředí.

| **Oblast zabezpečení**        | **Klíčové opatření / Nastavení**                                   | **Účel a mechanizmus**                                                                                                          |
| ----------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| **Whitelisting**              | Mapování klíčů (např. `switch/case` nebo JSON mapy)                | Uživatel volí pouze identifikátor (např. `page=1`), backend načte odpovídající soubor bez použití vstupního řetězce jako cesty. |
| **Ochrana proti Traversal**   | `basename($input)` v PHP                                           | Oznese ze vstupu veškeré cesty a vrátí pouze samotný název souboru. Zamezí posunům o úroveň výše (`../`).                       |
| **Rekurzivní čistění**        | Cyklus s `str_replace('../', '', $input)`                          | Odstraňuje sekvence `../` rekurzivně, čímž znemožní obcházení typu `....//`.                                                    |
| **Zakázání RFI**              | `allow_url_include = Off`<br><br>  <br><br>`allow_url_fopen = Off` | Vypne možnost načítat a spouštět vzdálené soubory přes HTTP/FTP wrappery v PHP (`php.ini`).                                     |
| **Restrikce adresářů**        | `open_basedir = /var/www` nebo Docker                              | Zamyká aplikaci výhradně do určeného webrootu. Znemožní čtení systémových souborů jako `/etc/passwd`.                           |
| **Vypnutí rizikových modulů** | Vypnutí rozšíření `expect`                                         | Zamezí spouštění systémových příkazů přes wrapper `expect://`.                                                                  |
| **Detekce (WAF)**             | ModSecurity (Permissive / Blocking mode)                           | Analýza příchozích požadavků a včasná detekce probíhajícího skenování nebo exploatace.                                          |

**1. Aplikace Whitelistu (Nejúčinnější obrana)**

Místo přímého předávání dynamického parametru do funkce `include($_GET['page'])` se vstup porovná vůči seznamu povolených souborů:

PHP

```
$allowed_pages = [
    'home' => 'views/home.php',
    'about' => 'views/about.php',
    'contact' => 'views/contact.php'
];

$page = $_GET['page'] ?? 'home';

if (array_key_exists($page, $allowed_pages)) {
    include($allowed_pages[$page]);
} else {
    include('views/404.php');
}
```

**2. Bezpečná sanitizace cest**

Pokud aplikace musí přijímat názvy souborů, je nutné vynutit použití vestavěných systémových funkcí:

- Používejte **`basename()`**, která odřízne jakoukoliv adresářovou cestu před názvem souboru.
    
- Pro rekurzivní odstranění `../` využijte cyklus:
    
    PHP
    
    ```
    while(substr_count($input, '../', 0)) {
        $input = str_replace('../', '', $input);
    }
    ```
    
- Vyhýbejte se psaní vlastních regulárních výrazů pro ošetření cest – často nepočítají s krajními případy (edge-cases) na úrovni shellu a interpretu.
    

**3. Hardening a filozofie Defense in Depth**

Cílem hardeningu není vytvořit stoprocentně neprolomitelný systém, ale výrazně ztížit postup útočníka, omezit dopad případné zranitelnosti (např. zamezit zisku RCE přes RFI) a generovat dostatek záznamů v logách. To zkracuje průměrnou dobu detekce útoku ze strany bezpečnostního týmu.