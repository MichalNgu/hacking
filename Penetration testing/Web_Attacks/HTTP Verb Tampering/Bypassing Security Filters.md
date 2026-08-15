Tato sekce vysvětluje druhý typ **HTTP Verb Tampering**, který pramení z nepozornosti při psaní zdrojového kódu (Insecure Coding). Vniká v momentě, kdy vývojář aplikuje bezpečnostní filtr pouze na konkrétní typ požadavku (např. `POST`), ale samotná logika aplikace zpracovává univerzální vstup.

### 🔍 Podstata zranitelnosti v kódu

K této chybě dochází nejčastěji v jazycích jako PHP, pokud vývojář použije superglobální pole `$_REQUEST`, které v sobě kombinuje parametry z `$_GET`, `$_POST` i `$_COOKIE`.

#### Zranitelný vzor (Vulnerable Pattern):

PHP

```
// ❌ CHYBA: Filtr kontroluje pouze pole $_POST
if (isset($_POST['filename'])) {
    if (preg_match('/[;&|]/', $_POST['filename'])) {
        die("Malicious Request Denied!");
    }
}

// ❌ CHYBA: Příkaz vykoná hodnota z $_REQUEST (přijme i GET)
system("touch " . $_REQUEST['filename']);
```

Pokud útočník pošle požadavek jako `POST`, filtr škodlivé znaky zachytí a požadavek zablokuje. Pokud však změní HTTP metodu na `GET`, podmínka `isset($_POST['filename'])` vrátí `false` a filtr se zcela přeletí. Funkce `system()` následně přečte nefiltrovaný parametr z `$_REQUEST['filename']`, čímž dojde k vykonání příkazu (**Command Injection**).

### 🛡️ Správné ošetření v kódu (Secure Coding)

Pro úplnou nápravu je nutné kombinovat striktní kontrolu HTTP metod s řádnou sanitizací a vyhýbáním se univerzálním proměnným typu `$_REQUEST`.

PHP

```
// 1. Kontrola povolené HTTP metody
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    http_response_code(405);
    die("Method Not Allowed");
}

// 2. Použití výhradně specifického pole $_POST
if (!isset($_POST['filename'])) {
    die("Missing parameter");
}

// 3. Sanitizace vstupu (Whitelisting)
$filename = preg_replace('/[^A-Za-z0-9_.-]/', '', $_POST['filename']);

// 4. Bezpečné zpracování rodnou PHP funkcí bez shellu
touch("/var/www/uploads/" . $filename);
```

### 📊 Srovnání typů HTTP Verb Tampering

| **Vlastnost**        | **Typ 1: Insecure Server Config**                                 | **Typ 2: Insecure Coding**                                                      |
| -------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Kde chyba vzniká** | Konfigurace webového serveru (`.htaccess`, `httpd.conf`).         | Zdrojový kód aplikace (PHP, JS, Python).                                        |
| **Příčina**          | Omezení autentizace na vybrané metody (např. `<Limit GET POST>`). | Filtrování vstupu v jednom poli (např. `$_POST`), ale zpracování v `$_REQUEST`. |
| **Typický dopad**    | Obcházení HTTP Basic Auth (přístup k chráněným stránkám).         | Obcházení WAF/filtrů (vedoucí k SQLi, Command Injection atd.).                  |
| **Detekce nástroji** | Snadno detekovatelné automatickými skenery.                       | Obtížně detekovatelné (vyžaduje aktivní testování logiky).                      |