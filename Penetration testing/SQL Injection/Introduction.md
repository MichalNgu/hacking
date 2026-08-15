## 🗄️ Úvod do SQL Injekce (SQLi)

> **Co to je:** SQL injection (SQLi) je jedna z nejběžnějších a nejnebezpečnějších zranitelností webových aplikací. Nastává ve chvíli, kdy aplikace nebezpečně spojuje uživatelský vstup s databázovým dotazem, což útočníkovi umožní "vystoupit" z původní logiky a spustit vlastní SQL příkazy.

![[Pasted image 20260802123619.png]]

### ⚙️ Jak SQLi funguje

1. **Únik z uživatelského vstupu:** Útočník vloží speciální znaky – nejčastěji jednoduchou (`'`) nebo dvojitou (`"`) uvozovku. Tím přeruší datový řetězec v SQL dotazu.
    
2. **Změna logiky dotazu:** Po úniku přidá vlastní SQL kód (např. pomocí operátoru `UNION`, vrstvených dotazů / stacked queries nebo logických podmínek typu `OR 1=1`).
    
3. **Získání výstupu:** Výsledek pozměněného dotazu se zobrazí na front-endu aplikace nebo se projeví v její reakci.
    

> ℹ️ Zatímco **SQLi** cílí na relagční databáze (MySQL, PostgreSQL, MSSQL), útoky na nerelační databáze (např. MongoDB) se označují jako **NoSQL injection**.

## 🎯 Dopady a případy použití (Impact & Use Cases)

SQL injection může mít pro organizaci devastační následky:

- **Únik citlivých dat:** Získání uživatelských jmen, hesel, osobních údajů či dat o platebních kartách.
    
- **Obcházení autentizace:** Přihlášení k účtům bez znalosti hesla (přeskočení přihlašovacího formuláře).
    
- **Eskalace oprávnění:** Přístup k administrátorským funkcím a chráněným sekcím webu.
    
- **Čtení a zápis souborů:** Přímý přístup k souborovému systému serveru (`LOAD_FILE()`, `INTO OUTFILE`), což může vést k nahrání **Web Shellu** a úplnému převzetí kontroly nad serverem (RCE).
    

## 🛡️ Prevence

Zranitelnostem typu SQLi lze předcházet především na úrovni vývoje aplikace a konfigurace databáze:

1. **Prepared Statements (Parametrizované dotazy):** Zajišťují, že vstup uživatele je vždy zpracován čistě jako _data_, nikoli jako spustitelný _kód_.
    
2. **Sanitizace a validace vstupů:** Striktní kontrola formátu a typu přijímaných dat (white-listing).
    
3. **Princip nejmenších privilegií (Least Privilege):** Databázový uživatel aplikace by měl mít pouze minimální nutná oprávnění (např. zákazy zápisu do souborového systému).