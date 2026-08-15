Použití logických operátorů **AND (`&&`)** a **OR (`||`)** přináší při exploataci OS Command Injection velkou výhodu: umožňuje přesně řídit tok vykonávání příkazů na základě **exit kódů** (návratových hodnot) předchozích procesů.

### ⚙️ Chování operátorů na základě Exit Kódů

V prostředí Linuxu i Windows vrací úspěšně dokončený příkaz návratový kód `0`. Pokud příkaz selže (např. chybějící parametr), vrací kód ne-nulový (nejčastěji `1`).

| **Operátor**     | **Podmínka vykonání druhého příkazu** | **Příkaz v praxi**              | **Výsledek**                       |
| ---------------- | ------------------------------------- | ------------------------------- | ---------------------------------- |
| **AND (`&&`)**   | První příkaz vrací `0` (Úspěch)       | `ping -c 1 127.0.0.1 && whoami` | Vykoná se `ping` **i** `whoami`.   |
| **OR (  \|\| )** |                                       | ping -c 1 127.0.0.1 \|\| whoami | vykoná se pouze ping               |
| **OR ( \|\| )**  |                                       | ping -c 1 \|\| whoami           | ping hodí chybu a vykoná se whoami |

> 💡 **Hackerský tip pro „čistý výstup“ (Clean Output):**
> 
> Pokud záměrně rozbijete původní příkaz (např. předáte pouze `|| whoami`), ušetříte si nepřehledné balastní výstupy (jako jsou statistiky pingu). Aplikace vrátí pouze chybovou hlášku a hned za ní **přímo čistý výstup vašeho příkazu**.

### 📋 Srovnávací Cheat Sheet: Injekční znaky podle zranitelností

Speciální znaky a operátory slouží k opuštění původního datového kontextu i v mnoha dalších třídách zranitelností:

| **Typ zranitelnosti**       | **Klíčové řídící znaky a operátory**                    |
| --------------------------- | ------------------------------------------------------- |
| **OS Command Injection**    | `;`, `&&`, `\|`, `\|`, `&`, `\n` (`%0a`), `` ` `` `$()` |
| **SQL Injection (SQLi)**    | `'`, `"`, `;`, `--`, `/* */`, `UNION`                   |
| **Code Injection**          | `'`, `;`, `--`, `/* */`, `$()`, `${}`, `#{}`            |
| **Directory Traversal**     | `../`, `..\`, `%00` (Null Byte)                         |
| **LDAP Injection**          | `*`, `(`, `)`, `&`, `\|`                                |
| **XPath Injection**         | `'`, `or`, `and`, `not`, `substring()`, `concat()`      |
| **CRLF / Header Injection** | `\n`, `\r\n`, `%0d%0a`, `\t`                            |
