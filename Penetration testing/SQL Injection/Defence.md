## 🛡️ Defenzivní strategie proti SQL Injection

Ochrana před SQL Injection spočívá v oddělení uživatelských dat od vykonávané řídicí logiky SQL dotazu. Zde jsou základní techniky zabezpečení:

### 1. Parametrizované dotazy (Prepared Statements) – **Nejúčinnější obrana**

Místo přímého skládání řetězců se používají zástupné symboly (placeholdery `?`). Databázový ovladač pošle strukturu dotazu a uživatelská data odděleně, čímž zcela zabrání interpretaci vstupu jako kódu.

#### Příklad v PHP (`mysqli`):

PHP

```
$username = $_POST['username'];
$password = $_POST['password'];

// Použití placeholderů '?'
$query = "SELECT * FROM logins WHERE username = ? AND password = ?";
$stmt = mysqli_prepare($conn, $query);

// Bezpečné navázání parametrů ('ss' určuje dva stringy)
mysqli_stmt_bind_param($stmt, 'ss', $username, $password);
mysqli_stmt_execute($stmt);
$result = mysqli_stmt_get_result($stmt);
```

### 2. Validace vstupů (Input Validation)

Kontrola, zda zadaná data odpovídají očekávanému formátu (např. e-mail, číslo, alfabetické znaky). Jakýkoliv neuspořádaný nebo neočekávaný znak je okamžitě odmítnut.

#### Příklad použití regulárního výrazu (`preg_match`):

PHP

```
$pattern = "/^[A-Za-z\s]+$/"; // Povoluje pouze písmena a mezery
$code = $_GET["port_code"];

if (!preg_match($pattern, $code)) {
    die("Invalid input!"); // Skript se ukončí dříve, než dotaz dosáhne databáze
}
```

### 3. Sanitizace vstupů (Input Sanitization / Escaping)

Pokud nelze použít parametrizované dotazy, musí se nebezpečné řídicí znaky (jako `'` nebo `"`) ošetřit pomocí escapovacích funkcí, které zruší jejich speciální význam v SQL.

- **MySQL/PHP:** `mysqli_real_escape_string($conn, $input)`
    
- **PostgreSQL/PHP:** `pg_escape_string($conn, $input)`
    

> ⚠️ _Poznámka:_ Sanitizace/escaping je považována za méně spolehlivou než Prepared Statements, protože může dojít k chybě při špatném kódování znaků.

### 4. Princip nejnižších oprávnění (Principle of Least Privilege)

Uživatel, pod kterým se webová aplikace připojuje k databázi, by měl mít uděleny **pouze minimální nutné pravomoci** (např. pouze `SELECT` nad konkrétní tabulkou).

- Webová aplikace by **nikdy** neměla přistupovat k databázi pod účtem `root` nebo `admin`.
    
- Zamezí se tím čtení citlivých tabulek (`credentials`) nebo zápisu souborů na server (`INTO OUTFILE`), i kdyby zranitelnost v aplikaci existovala.
    

SQL

```
CREATE USER 'reader'@'localhost' IDENTIFIED BY 'heslo';
GRANT SELECT ON ilfreight.ports TO 'reader'@'localhost';
```

### 5. Web Application Firewall (WAF)

WAF (např. _ModSecurity_, _Cloudflare_) analyzuje příchozí HTTP požadavky na aplikaci a blokuje známé vzorce SQL injection (např. řetězce `UNION SELECT` nebo `INFORMATION_SCHEMA`). Slouží jako doplňková vrstva ochrany (_Defense in Depth_).