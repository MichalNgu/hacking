**Zranitelnost File Inclusion (LFI / RFI)** vzniká v momentě, kdy aplikace používá uživatelsky řízené parametry k dynamickému načítání lokálních nebo vzdálených souborů na serveru bez řádné sanitizace vstupu.

**Rozdíl mezi čtením obsahu a spouštěním kódu**

Riziko zranitelnosti se odvíjí od toho, jakým způsobem použitá funkce se souborem nakládá:

- **Čtení obsahu (Read):** Umožňuje útočníkovi zobrazit zdrojové kódy, konfigurační soubory, klíče nebo systémová data (např. `/etc/passwd`).
    
- **Spouštění kódu (Execute):** Může vést k okamžitému vykonání vzdáleného kódu (RCE) na serveru.
    
- **Vzdálená URL (Remote URL / RFI):** Umožňuje načíst a spustit kód ze serveru útočníka.
    

|**Jazyk / Framework**|**Funkce**|**Čtení obsahu (Read)**|**Spuštění kódu (Execute)**|**Vzdálená URL (RFI)**|
|---|---|---|---|---|
|**PHP**|`include()` / `include_once()`|✅|✅|✅|
|**PHP**|`require()` / `require_once()`|✅|✅|❌|
|**PHP**|`file_get_contents()`|✅|❌|✅|
|**PHP**|`fopen()` / `file()`|✅|❌|❌|
|**NodeJS**|`fs.readFile()` / `fs.sendFile()`|✅|❌|❌|
|**NodeJS**|`res.render()`|✅|✅|❌|
|**Java**|`<jsp:include>`|✅|❌|❌|
|**Java**|`<c:import>`|✅|✅|✅|
|**.NET**|`@Html.Partial()` / `Response.WriteFile()`|✅|❌|❌|
|**.NET**|`<!--#include file="..."-->`|✅|✅|✅|

**Zranitelné kódové vzory napříč technologiemi**

- **PHP:** `include($_GET['language']);` – Bezpečné filtrování chybí, jakákoliv cesta zadaná v parametru `language` se načte do stránky.
    
- **NodeJS (Express):** `res.render(`/${req.params.language}/about.html`);` – Přímé skládání řetězce z URL parametru umožňuje manipulaci se směrováním.
    
- **Java (JSP):** `<jsp:include file="<%= request.getParameter('language') %>" />` – Přímé předání vstupu do include direktivy.
    
- **.NET:** `Response.WriteFile(HttpContext.Request.Query['language']);` – Vypíše obsah libovolného souboru zadaného v GET parametru do HTTP odpovědi.