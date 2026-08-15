Zabezpečení proti **HTTP Verb Tampering** vyžaduje odstranění omezení na konkrétní metody v konfiguraci webového serveru a důslednou konzistenci při načítání a sanitizaci parametrů na úrovni zdrojového kódu.

**Oprava konfigurace webových serverů**

Nejčastější chybou v konfiguraci je explicitní definování autorizace pouze pro vybrané HTTP metody (`GET`, `POST`), čímž zůstávají ostatní metody (`HEAD`, `OPTIONS`, `PUT`) zcela nechráněné.

|**Server / Framework**|**Zranitelná konfigurace**|**Bezpečná konfigurace**|
|---|---|---|
|**Apache**|`<Limit GET>` uvnitř `<Directory>`|Odstranění direktivy `<Limit>` (uplatní se na vše) nebo použití `<LimitExcept>`|
|**Tomcat**|Vynechání metod v `<http-method>`|Použití `<http-method-omission>` pro pokrytí všech zbývajících metod|
|**ASP.NET**|Omezení rozsahu v `verbs="GET"`|Povolení/zakázání bez specifikace atributu `verbs`|

_Doporučení:_ Pokud webová aplikace nevyžaduje metody jako `HEAD` nebo `TRACE`, měly by být na úrovni serveru globálně zakázány.

**Bezpečný vývoj v kódu (Secure Coding)**

V kódu zranitelnost vzniká nekonzistentním přístupem k parametrům — např. když filtr ověřuje pouze pole `$_POST['param']`, ale samotná funkce vykonává příkaz z unifikovaného pole `$_REQUEST['param']`.

PHP

```
// ❌ ZRANITELNÝ KÓD: Filtr ověřuje pouze POST, ale příkaz spouští cokoliv z $_REQUEST
if (isset($_REQUEST['filename'])) {
    if (!preg_match('/[^A-Za-z0-9. _-]/', $_POST['filename'])) {
        system("touch " . $_REQUEST['filename']);
    }
}

// 1. BEZPEČNÝ KÓD: Striktní použití stejné metody pro filtraci i vykonání
if (isset($_POST['filename'])) {
    if (!preg_match('/[^A-Za-z0-9. _-]/', $_POST['filename'])) {
        system("touch " . $_POST['filename']);
    }
}

// 2. BEZPEČNÝ KÓD: Pokrytí všech metod v bezpečnostním filtru
if (isset($_REQUEST['filename'])) {
    if (!preg_match('/[^A-Za-z0-9. _-]/', $_REQUEST['filename'])) {
        system("touch " . $_REQUEST['filename']);
    }
}
```

Proměnné pro globální zachycení vstupů napříč různými HTTP metodami v dalších jazycích:

- **PHP:** `$_REQUEST['param']`
    
- **Java:** `request.getParameter('param')`
    
- **C# (.NET):** `Request['param']`