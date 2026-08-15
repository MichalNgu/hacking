Detekce **OS Command Injection** spočívá v testování, zda aplikace odesílá uživatelský vstup do systémového shellu bez dostatečné sanitizace. Útočník se pokouší připojit vlastní příkaz k původnímu systémovému volání (např. k příkazu `ping`) pomocí **řídících operátorů shellu**.

### 📌 Přehled řídících operátorů (Command Injection Operators)

Při posílání payloadů v HTTP požadavcích (GET/POST) je nutné speciální znaky **URL-encodovat**, aby je webový server správně interpretoval a nezpůsobily chybu v syntaxi HTTP.

| **Operátor** | **URL-Encoded** | **Logika vykonání**                                                       | **OS Kompatibilita**                           |
| ------------ | --------------- | ------------------------------------------------------------------------- | ---------------------------------------------- |
| `;`          | `%3b`           | Vykoná oba příkazy postupně bez ohledu na výsledek prvního.               | Linux, Windows PowerShell _(ne funguje v CMD)_ |
| `\n`         | `%0a`           | Odřádkování – vykoná oba příkazy jako samostatné řádky skriptu.           | Linux, Windows                                 |
| `&`          | `%26`           | Spustí první příkaz na pozadí a ihned vykoná druhý.                       | Linux, Windows                                 |
| `&&`         | `%26%26`        | Vykoná druhý příkaz **pouze při úspěchu** prvního (návratový kód `0`).    | Linux, Windows                                 |
| `\|`         | `%7c`           | Přesměruje výstup (STDOUT) prvního příkazu do vstupu (STDIN) druhého.     | Linux, Windows                                 |
| `\|`         | `%7c%7c`        | Vykoná druhý příkaz **pouze při selhání** prvního (návratový kód `!= 0`). | Linux, Windows                                 |
| `` `cmd` ``  | `%60cmd%60`     | Vykoná příkaz v podřazeném shellu (Sub-shell execution).                  | Pouze Linux / Bash                             |
| `$()`        | `%24%28...%29`  | Vykoná příkaz v podřazeném shellu (Sub-shell execution).                  | Pouze Linux / Bash                             |
|              |                 |                                                                           |                                                |

### 💡 Klíčové postřehy pro detekci

1. **Injekce do očekávaného vstupu:** Pokud aplikace očekává např. IP adresu `127.0.0.1`, otestujte vložení `127.0.0.1; whoami` nebo `127.0.0.1 && id`.
    
2. **Rozdíly mezi Windows a Linuxem:**
    
    - Středník `;` nefunguje v klasickém prostředí Windows Command Line (`cmd.exe`), ale v PowerShellu ano.
        
    - Operátory `$()` a zpětné uvozovky `` ` `` jsou dostupné výhradně na systémech typu Unix/Linux.
        
3. **URL Encoding v Burp Suite / cURL:** Při posílání příkazů přes HTTP POST nebo GET parametry nezapomeňte zakódovat znaky jako `&` (`%26`) nebo `;` (`%3b`), jinak je webový server vyhodnotí jako oddělovače HTTP parametrů.