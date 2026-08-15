Klientská validace (JavaScript v prohlížeči, HTML atributy) slouží výhradně ke zlepšení uživatelského rozhraní (UX) a nepředstavuje žádnou bezpečnostní bariéru. Jelikož má uživatel plnou kontrolu nad klientským prostředím i odesílanými HTTP požadavky, lze jakoukoliv klientskou kontrolu snadno obejít.

**Přehled metod obcházení klientské validace**

| **Metoda**                       | **Princip fungování**                                                                   | **Postup a nástroje**                                                                                                                   |
| -------------------------------- | --------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Modifikace HTTP požadavku** | Vynechání klientských skriptů a úprava požadavku až po jeho zachycení na síťové vrstvě. | Burp Suite / OWASP ZAP $\rightarrow$ zachycení požadavku na `/upload.php` $\rightarrow$ úprava `filename="shell.php"` a obsahu souboru. |
| **2. Úprava DOM / JavaScriptu**  | Odstranění nebo úprava validační logiky přímo v paměti prohlížeče před odesláním.       | DevTools (F12) $\rightarrow$ smazání atributu `accept=".jpg..."` a event handleru `onchange="checkFile(this)"` v HTML kódu.             |

**Správná oprava na straně serveru (Server-Side Remediation)**

Aby bylo nahrávání souborů bezpečné, musí veškerá validace probíhat výhradně na backendu:

- **Striktní Whitelist přípon:** Kontrolujte příponu souboru vůči pevně definovanému seznamu povolených formátů (např. pouze `.jpg`, `.png`).
    
- **Validace obsahu (Magic Bytes & Content-Type):** Neověřujte pouze hlavičku `Content-Type` z HTTP požadavku (tu lze snadno změnit), ale kontolujte reálné bajty souboru (magic bytes) nebo soubor rekonfigurujte/překódujte pomocí grafické knihovny.
    
- **Přejmenování a náhodné ID:** Nahrávané soubory přejmenovávejte na náhodné řetězce (např. UUID) a odstraňte původní přípony nebo nebezpečné znaky.
    
- **Ukládání mimo Webroot a zákaz spouštění:** Soubory ukládejte do adresáře přístupného pouze aplikačnímu serveru (mimo klientsky přístupný `webroot`) a v adresáři pro nahrávaná data striktně zakažte vykonávání skriptů (např. pomocí `.htaccess` nebo konfigurace Nginx).