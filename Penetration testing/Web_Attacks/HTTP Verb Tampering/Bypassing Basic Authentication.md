Tato sekce popisuje praktickou exploataci špatně nakonfigurované základní autentizace (HTTP Basic Auth) na webovém serveru pomocí změny HTTP metody na **HEAD**.

**Mechanismy a průběh obcházení:**

- **Příčina zranitelnosti:** Správce serveru omezil přístup k adresáři `/admin/` pomocí direktivy `<Limit GET POST>`. Server tak vyžaduje přihlášení pouze v případě, že požadavek přijde jako `GET` nebo `POST`.
    
- **Průběh obcházení:**
    
    1. **Identifikace:** Přístup na `/admin/reset.php` přes `GET` i `POST` selže s chybou `401 Unauthorized`.
        
    2. **Průzkum metod:** Pomocí `curl -i -X OPTIONS http://SERVER_IP:PORT/` zistíme podporované metody (`Allow: POST,OPTIONS,HEAD,GET`).
        
    3. **Provedení:** Změnou metody na `HEAD` obcházíme autentizační filtr serveru. Metoda `HEAD` funguje stejně jako `GET` (spustí backendový skript a provede reset souborů), ale v odpovědi nevrací tělo (body), pouze HTTP hlavičky `200 OK`.
        

**Správná oprava v konfiguraci webového serveru (Apache):**

Pro zabezpečení adresáře vyžadujte autentizaci **bez omezení metod**, případně použijte direktivu `<LimitExcept>`:

XML

```
<!-- ❌ ZRANITELNÁ KONFIGURACE -->
<Directory "/var/www/html/admin">
    <Limit GET POST>
        Require valid-user
    </Limit>
</Directory>

<!-- 1. BEZPEČNÁ KONFIGURACE (Doporučeno: Autentizace pro všechny metody) -->
<Directory "/var/www/html/admin">
    Require valid-user
</Directory>
```