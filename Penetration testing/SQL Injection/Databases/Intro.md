## 🗄️ Úvod do databází

Databáze slouží webovým aplikacím k ukládání a správě dat v reálném čase – od systémových souborů a obrázků přes uživatelské příspěvky až po citlivé přihlašovací údaje. Zatímco starší aplikace spoléhaly na souborové databáze (které byly při větším objemu dat pomalé), moderní systémy využívají **Systémy řízení báze dat (DBMS)**.

### ⚙️ Klíčové vlastnosti DBMS

Systém řízení báze dat (DBMS) zajišťuje vytváření, správu a zabezpečení databází. Zprostředkovává komunikaci mezi aplikací a uloženými daty pomocí jazyka **SQL (Structured Query Language)**.

|**Vlastnost**|**Popis a význam**|
|---|---|
|**Konkurence (Concurrency)**|Zajišťuje bezpečný paralelní přístup více uživatelů současně bez rizika poškození nebo ztráty dat.|
|**Konzistence (Consistency)**|Garantuje, že data zůstanou platná a v konzistentním stavu i při souběžných operacích.|
|**Bezpečnost (Security)**|Poskytuje řízení přístupu přes autentizaci a oprávnění, čímž brání neoprávněnému čtení či úpravám.|
|**Spolehlivost (Reliability)**|Umožňuje snadné zálohování a obnovu databází do předchozího stavu v případě výpadku nebo chyb.|
|**Jazyk SQL**|Poskytuje intuitivní syntaxi pro dotazování, vkládání, úpravu a mazání dat.|

### 🏛️ Dvouvrstvá architektura (Two-Tier Architecture)

Komunikace mezi uživatelem a databází typicky probíhá ve dvou hlavních vrstvách:

1. **Vrstva I (Client-Side Application):**
    
    - Klientská aplikace (webová stránka, GUI rozhraní).
        
    - Zpracovává uživatelské akce (např. přihlášení, vložení komentáře) a předává je dále přes API nebo HTTP požadavky.
        
2. **Vrstva II (Middleware & Application Layer):**
    
    - Meziaplikace přeloží požadavek klienta do databázového dotazu (SQL query).
        
    - Pomocí specifických ovladačů a knihoven komunikuje přímo s DBMS.
        
    - **DBMS** dotaz zpracuje (výběr, zápis, úprava, smazání) a vrátí data nebo chybový kód zpět aplikaci.
        

> ℹ️ Aplikace i DBMS mohou běžet na stejném serveru, ale u produkčních systémů s vysokou zátěží bývá databáze oddělena na samostatný stroj pro dosažení vyššího výkonu a škálovatelnosti.