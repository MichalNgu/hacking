### Rozdíl mezi GET a POST fuzzingem

- **GET parametry:** Odesílají se jako součást URL adresy (za otazníkem `?param=hodnota`).
    
- **POST parametry:** Nejsou viditelné v URL adrese, ale přenášejí se v **těle HTTP požadavku** (v těle dat / data field).
    

> **💡 Důležitá poznámka pro PHP:** V prostředí PHP vyžaduje zpracování POST dat správně nastavenou HTTP hlavičku `Content-Type: application/x-www-form-urlencoded`. Bez této hlavičky server POST data nemusí vůbec zpracovat.

### Jak provádět POST Parameter Fuzzing v `ffuf`

Pro posílání POST požadavků a fuzzing těla dat používáme přepínače:

- **`-X POST`**: Určuje HTTP metodu.
    
- **`-d 'FUZZ=hodnota'`**: Specifikuje data posílaná v těle požadavku, kde zástupný symbol `FUZZ` představuje název hledaného parametru.
    
- **`-H 'Content-Type: application/x-www-form-urlencoded'`**: Přidává potřebnou hlavičku pro PHP aplikaci.
    

#### Příkaz pro vyhledávání POST parametrů:

Bash

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs <VELIKOST_DEFAULTNI_ODPOVEDI>
```

### Ověření nalezeného parametru pomocí `curl`

Pokud `ffuf` odhalí nový parametr (např. `id`), můžete jeho funkčnost snadno ověřit ručně z příkazové řádky pomocí nástroje `curl`:

Bash

```
curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'
```

- **Vyhodnocení odpovědi:**
    
    - Odpověď typu `<p>Invalid id!</p>` potvrzuje, že parametr `id` backend skutečně přijímá a zpracovává (na rozdíl od ignorovaných parametrů).
        
    - Nyní zbývá v dalším kroku zjistit správnou hodnotu pro tento parametr (Value Fuzzing).