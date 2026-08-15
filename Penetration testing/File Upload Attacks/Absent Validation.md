Absence validace nahrávaných souborů (_Absent Validation_) znamená, že webová aplikace neprovádí žádné kontroly přípon ani obsahu na klientské ani serverové straně. Tento stav umožňuje přímé nahrání spustitelného skriptu a získání vzdáleného přístupu k serveru (RCE).

**Postup identifikace a ověření zranitelnosti**

| **Krok**                  | **Cíl**                                       | **Metoda / Technika**                                                                                                                                                            |
| ------------------------- | --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Určení technologií** | Zjištění backendového jazyka (PHP, ASPX, JSP) | • Testování přípon v URL (např. zobrazení `/index.php`)<br><br>  <br><br>• Rozšíření Wappalyzer v prohlížeči<br><br>  <br><br>• Analýza HTTP hlaviček (`Server`, `X-Powered-By`) |
| **2. Testovací nahrání**  | Ověření absence filtrů na serveru             | Nahrání souboru `test.php` s neškodným kódem:<br><br>  <br><br>`<?php echo "Hello HTB"; ?>`                                                                                      |
| **3. Ověření spouštění**  | Potvrzení vykonání kódu backendem             | Přístup k nahranému souboru (např. `/uploads/test.php`). Zobrazení textu _"Hello HTB"_ potvrdí vykonání PHP kódu.                                                                |

**Indikátory absence filtrů v aplikaci:**

- Dialogové okno pro výběr souboru v prohlížeči zobrazuje výchozí filtr **Všechny soubory (_._)**.
    
- Rozhraní přijme jakoukoliv spustitelnou příponu bez chybového hlášení.
    
- Aplikace po nahrání vrací přímý odkaz na nahraný soubor (např. v adresáři `/uploads/` nebo `/files/`).