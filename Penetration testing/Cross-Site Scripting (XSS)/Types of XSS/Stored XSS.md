## Stored XSS 

(Persistent XSS) je nejnebezpečnější variantou Cross-Site Scriptingu. Vzniká v momentě, kdy backend aplikace uloží uživatelský vstup do databáze a následně jej bez sanitizace vykreslí v prohlížeči každého návštěvníka dané stránky.

**Hlavní rizika a vlastnosti Stored XSS**

- **Široký zásah obětí:** Neútočí pouze na jednoho konkrétního uživatele přes upravený odkaz, ale zasáhne automaticky každého, kdo navštíví infikovanou část webu (fóra, profily, to-do listy, komentáře).
    
- **Trvalost (Persistence):** Škodlivý kód zůstává v aplikaci aktivní tak dlouho, dokud není manuálně odstraněn z backendové databáze.
    

**Základní testovací Payloady**

| **Payload**                                 | **Účel a výhody**                                                                                                                                   |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`<script>alert(window.origin)</script>`** | Zobrazí doménu, na které se kód vykonává. Zobrazení `window.origin` pomáhá ověřit, zda aplikace nepoužívá k izolaci vstupů cross-domain `<iframe>`. |
| **`<script>print()</script>`**              | Otevře tiskové okno prohlížeče. Výborná alternativa v případech, kdy moderní prohlížeče blokují funkci `alert()`.                                   |
| **`<plaintext>`**                           | Zastaví vykreslování zbývajícího HTML kódu na stránce a zobrazí jej jako prostý text. Snadno rozpoznatelný vizuální efekt.                          |

**Postup ověření zranitelnosti**

1. **Injekce:** Vložení testovacího payloadu do formulářového pole (např. přidaná položka v seznamu úkolů).
    
2. **Kontrola zdrojového kódu (`CTRL+U`):** Ověření, zda se zadané tagy `<script>` zobrazují v HTML struktuře v nezměněné (neescapované) podobě.
    
3. **Potvrzení persistence:** Obnovení stránky (`F5`) nebo její otevření v jiném prohlížeči. Pokud se kód spustí znovu, jedná se o Stored XSS.