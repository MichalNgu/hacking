**Whitelist** je bezpečnější než **blacklist**, protože povoluje pouze definované přípony. Přesto ji lze obejít při špatně napsaném regulárním výrazu, chybné konfiguraci webového serveru nebo pomocí injektáže speciálních znaků.

| **Technika obcházení**                            | **Vzorec / Příklad názvu**                           | **Princip fungování zranitelnosti**                                                                                                                          |
| ------------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Dvojitá přípona (Double Ext)**                  | `shell.jpg.php`                                      | Špatný regex v aplikaci (chybí kotva `$`). Aplikace ověří výskyt `.jpg` v názvu, ale server soubor spustí podle finální přípony `.php`.                      |
| **Obrácená dvojitá přípona (Reverse Double Ext)** | `shell.php.jpg`                                      | Aplikace schválí příponu `.jpg`, ale webový server (např. Apache s chybou v `<FilesMatch ".+\.php">` bez `$`) vykoná PHP kód, protože název obsahuje `.php`. |
| **Null Byte Injection** _(PHP < 5.3.4)_           | `shell.php%00.jpg`                                   | Název projde kontrolou `.jpg`, ale Null Byte (`%00` / `\0`) ukončí řetězec a soubor se na disk uloží jako `shell.php`.                                       |
| **Windows NTFS / Colon**                          | `shell.aspx:.jpg`                                    | Znak dvojtečky na Windows/IIS serveru ořízne příponu za dvojtečkou a uloží soubor jako `shell.aspx`.                                                         |
| **Injektáž znaků / Lomítek**                      | `shell.php%20.jpg`<br><br>  <br><br>`shell.php/.jpg` | Vložení mezer (`%20`), nových řádků (`%0a`), teček nebo lomítek, které oklamou validátor nebo souborový systém.                                              |

**Generování Permutací pro Character Injection (Bash Script)**

Pro automatizované testování (např. přes Burp Intruder) lze připravit kustomizovaný wordlist se všemi kombinacemi injektovaných znaků:

Bash

```
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps' '.phtml'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```