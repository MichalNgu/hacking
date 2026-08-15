Útoky přes nahrávání souborů se neomezují pouze na přímé nahrání webshellu. Zneužít lze i samotný název souboru, chování operačního systému (např. Windows) nebo zranitelnosti v knihovnách pro zpracování médií.

| **Typ útoku**                 | **Vektor / Payload**                                    | **Princip zranitelnosti**                                                                                           |
| ----------------------------- | ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Command Injection v názvu** | `file$(whoami).jpg`<br><br>  <br><br>`file.jpg\|whoami` | Backend předává název souboru do systémového příkazu (např. `mv` nebo `cp`) bez sanitizace.                         |
| **Stored XSS v názvu**        | `<script>alert(1)</script>.jpg`                         | Název souboru se po nahrání nesanitovaně vykresluje v HTML kódu aplikace.                                           |
| **SQL Injection v názvu**     | `file';SELECT sleep(5);--.jpg`                          | Název souboru se vkládá přímo do SQL dotazu při ukládání metadat do databáze.                                       |
| **Windows Reserved Names**    | `CON`, `COM1`, `LPT1`, `NUL`                            | Windows zakáže vytvoření souboru s tímto názvem, což vyvolá aplikací neošetřenou výjimku a odhalí cestu k adresáři. |
| **Windows 8.3 Short Names**   | `WEB~1.CON`                                             | Využití Tilde konvence (`~`) k přístupu nebo přepsání konfiguračních souborů (např. `web.config`).                  |

**Techniky odhalení upload adresáře (Upload Directory Disclosure)**

- **Vyvolání chyby souborového systému:**
    
    - **Extrémně dlouhé názvy:** Nahrání souboru s názvem přesahujícím 255 až 5000 znaků způsobí selhání zápisu a výpis cesty v chybovém hlášení.
        
    - **Souběžný přístup / Duplicity:** Nahrání dvoch identických souborů současně nebo nahrání souboru s již existujícím názvem.
        
    - **Vyhrazené znaky:** Vložení zakázaných znaků (`|`, `<`, `>`, `*`, `?`), které způsobí I/O chybu v OS.
        

**Pokročilé útoky na knihovny (Processing Libraries)**

Při automatickém zpracování nahraných souborů na pozadí (konverze videa, změna velikosti obrázků, generování náhledů PDF) dochází k předávání dat do externích knihoven:

- **ffmpeg:** Zranitelnosti zpracování formátu AVI/HLS vedoucí k XXE nebo čtení lokálních souborů.
    
- **ImageMagick / GraphicsMagick:** Historické i novější RCE zranitelnosti při parsování škodlivě upravených obrázků.