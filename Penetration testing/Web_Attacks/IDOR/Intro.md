# Insecure Direct Object References (IDOR)
### 🔑 Podstata a příčina zranitelnosti IDOR

- **Co je přímý odkaz na objekt (Direct Object Reference):** Aplikace vystaví interní identifikátor databázového záznamu nebo souboru přímo v parametrů požadavku (např. `download.php?file_id=123` nebo `/api/users/1045`).
    
- **Kdy vzniká zranitelnost:** Pouhé vystavení ID v URL nebo v těle požadavku _není_ zranitelností samo o sobě. Zranitelnost vzniká až v momentě, kdy **na backendu chybí kontrola oprávnění (Access Control)**, která by ověřila, zda přihlášený uživatel skutečně vlastní daný zdroj nebo má právo k němu přistupovat.
    
- **Proč je IDOR tak běžný:** Vývojáři se často spoléhají na to, že uživatelské rozhraní (front-end) zobrazí pouze tlačítka a odkazy na uživatelova vlastní data. Pokud však uživatel požadavek ručně upraví (změní ID na `124`), nefunkční kontrola přístupu na backendu mu data bez ověření vydá.
    

### 💥 Dopady zranitelností IDOR

1. **IDOR Information Disclosure (Únik informací):**
    
    - Čtení souborů, faktur, osobních údajů nebo platebních karet ostatních uživatelů pouhou změnou/hádáním sekvenčních čísel nebo UUID.
        
2. **Neoprávněné úpravy / smazání dat:**
    
    - Pokud rozhraní umožňuje modifikaci (`PUT`/`POST`) nebo mazání (`DELETE`), útočník může přepsat či smazat účty nebo data jiných uživatelů.
        
3. **IDOR Insecure Function Calls (Zvýšení oprávnění):**
    
    - Zneužití volání administrativních funkcí nebo API, ke kterým běžný uživatel nemá mít přístup (např. `/api/v1/admin/change_role?user_id=10&role=admin`). If backend neuplatňuje striktní RBAC (Role-Based Access Control), dojde k převzetí kontroly nad aplikací.