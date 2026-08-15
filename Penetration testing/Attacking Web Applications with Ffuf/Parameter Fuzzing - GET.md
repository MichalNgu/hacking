### Proč vyhledávat skryté parametry?

Pokud stránka (např. `admin.php`) zobrazuje zprávu o neoprávněném přístupu nebo chybějícím klíči, ale aplikace nepoužívá přihlašovací formulář ani cookies, často očekává vstup předaný přes parametr (např. klíč, token, ID).

> **💡 Pentesting tip:** Skryté nebo nezdokumentované parametry vývojáři často méně testují a zabezpečují. Jsou skvělým cílem pro obcházení autorizace nebo hledání dalších zranitelností (SQLi, LFI apod.).

### Jak funguje GET Parameter Fuzzing v `ffuf`

GET parametry se předávají přímo v URL adrese za otazníkem (`?param=hodnota`). Zástupný symbol `FUZZ` se umístí na pozici **názvu parametru**, zatímco hodnota se nastaví na libovolný řetězec (např. `key` nebo `1`).

#### 1. Doporučený slovník

SecLists obsahuje specializovaný slovník s běžnými názvy parametrů: `/opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt`

#### 2. Příkaz pro vyhledávání GET parametrů

Bash

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs <VELIKOST_DEFAULTNI_ODPOVEDI>
```

### Postup při filtrování:

1. Nejprve spusťte příkaz bez filtru `-fs`, abyste zjistili výchozí velikost odpovědi pro neexistující parametry.
    
2. Odfiltrujte tuto velikost pomocí `-fs <velikost>` (stejně jako u VHostů).
    
3. Jakýkoliv parametr, který vrátí **jinou velikost odpovědi**, znamená, že backend na daný parametr reaguje (i kdyby vrátil chybovou hlášku typu _"Deprecated"_ nebo _"Invalid key"_).