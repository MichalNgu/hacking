Klientská validace (JavaScript v prohlížeči, HTML5 atributy) slouží výhradně ke zlepšení uživatelského zážitku (UX). Pokud vývojáři zapomenou ošetřit vstup na backendu a spoléhají pouze na klientskou kontrolu, lze tuto ochranu snadno vynechat úpravou požadavku přímo přes proxy nástroj (např. **Burp Suite** nebo **OWASP ZAP**).

### 🔍 Postup obcházení klientské validace krok za krokem

```
[ Prohlížeč (JS Validace) ] --(Zachycuje)--> [ Burp Suite Proxy ] --(Upravený Payload)--> [ Backend Server ]
```

1. **Detekce klientského filtru:**
    
    - Po zadání nevalidního vstupu (např. `127.0.0.1; whoami`) stránka ihned zobrazí chybu.
        
    - V záložce **Network (F12)** vidíte, že se při kliknutí na tlačítko **neodoslal žádný HTTP požadavek**. To potvrzuje, že validace probíhá lokálně v prohlížeči.
        
2. **Zachycení legitimního požadavku:**
    
    - Do formuláře zadejte platnou IP adresu (např. `127.0.0.1`), která klientskou validací projde.
        
    - V nástroji **Burp Suite** zachyťte odlétající `POST` požadavek a pošlete jej do modulu **Repeater** (`CTRL + R`).
        
3. **Injektáž payloadu a URL Encoding:**
    
    - V záložce Repeater upravte hodnotu parametru (např. `ip=127.0.0.1; whoami`).
        
    - Znaky payloadu zakódujte pomocí URL-encodingu (`CTRL + U` v Burp Suite) – např. středník `;` přepište na `%3b` nebo mezeru na `+` / `%20`:
        
        HTTP
        
        ```
        POST /index.php HTTP/1.1
        Host: target_ip
        Content-Type: application/x-www-form-urlencoded
        
        ip=127.0.0.1%3b+whoami
        ```
        
4. **Kontrola výstupu:**
    
    - Po kliknutí na **Send** vrátí backend server odpověď, v níž naleznete jak výstup příkazu `ping`, tak výstup injektovaného příkazu `whoami` (např. uživatele `www-data`).
        

### ⚖️ Klientská vs. Backendová validace

| **Vlastnost**    | **Klientská validace (Client-Side)**                  | **Backendová validace (Server-Side)**                |
| ---------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| **Kde probíhá**  | V prohlížeči uživatele (JS / HTML).                   | Na serveru před předáním příkazu systému.            |
| **Účel**         | Rychlá odezva pro uživatele (UX).                     | Bezpečnost aplikace (Security).                      |
| **Spolehlivost** | ❌ Nulová (útočník má plnou kontrolu nad prohlížečem). | Striktní sanitizace / parametrizace chránící server. |