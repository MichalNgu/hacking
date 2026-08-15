## ⚙️ 1. Nastavení a Základy (Proxy Setup)

- **Adresa lokální proxy:** `127.0.0.1:8080` (nastavení v `Tools` ➔ `Options` ➔ `Network` ➔ `Local Servers/Proxies`).
    
- **Předkonfigurovaný prohlížeč:** Spustíš ho jedním kliknutím na ikonu Firefoxu v pravé části horní lišty (nevyžaduje ruční konfiguraci proxy ani certifikátu).
    
- **Instalace CA certifikátu (pro vlastní prohlížeč):**
    
    1. V ZAPu přejdi do `Tools` ➔ `Options` ➔ `Network` ➔ `Server Certificates`.
        
    2. Klikni na **Save** a ulož certifikát.
        
    3. V prohlížeči (např. Firefox: `about:preferences#privacy` ➔ _View Certificates_) importuj certifikát do záložky _Authorities_ a povol důvěryhodnost pro webové stránky.
        
- **ZAP HUD (Heads Up Display):** Rozhraní integrované přímo do prohlížeče, které umožňuje ovládat ZAP během proklikávání aplikace. Zapíná se ikonou v pravém horním rohu ZAPu.
    

## 🛑 2. Odchytávání požadavků a odpovědí (Intercepting Web Requests & Responses)

- **Zapnutí / Vypnutí odchytávání (Break):**
    
    - Klávesová zkratka **`CTRL + B`**.
        
    - Nebo kliknutím na ikonu zelené/červené kuličky v horní liště.
        
- **Ovládání zachyceného požadavku:**
    
    - **Step (Krok):** Přepošle aktuální požadavek/odpověď a automaticky se zastaví na dalším kroku.
        
    - **Continue (Pokračovat):** Přepošle požadavek a vypne pozastavování pro ostatní běžící provoz.
        
    - **Drop:** Zahodí požadavek.
        
- **Funkce v ZAP HUD:**
    
    - **Show/Enable (Ikona žárovky):** Automaticky zviditelní skrytá formulářová pole (`type="hidden"`) a zpřístupní zakázané prvky (`disabled`) přímo ve webové stránce.
        
    - **Display Comments:** Zvýrazní místa v HTML kódu, kde vývojáři zanechali komentáře.
        

## 🔄 3. Automatické úpravy (Automatic Modification - Replacer)

> **Kde to najdeš:** Klávesová zkratka **`CTRL + R`** nebo v nastavení `Options` ➔ **Replacer**.

- **Změna User-Agenta (Obcházení filtrů):**
    
    - **Match Type:** `Request Header (will add if not present)`
        
    - **Match String:** `User-Agent`
        
    - **Replacement String:** `HackTheBox Agent 1.0`
        
- **Úprava HTML prvků v odpovědi (Response Body):**
    
    - **Match Type:** `Response Body String`
        
    - **Match String:** `type="number"` ➔ **Replacement String:** `type="text"`
        
    - **Match String:** `maxlength="3"` ➔ **Replacement String:** `maxlength="100"`
        
- **Automatické vstřikování do těla požadavku (POST Body):**
    
    - **Match Type:** `Request Body String`
        
    - **Match String:** `ip=1` ➔ **Replacement String:** `ip=1;ls;`
        

## 🔁 4. Opakované odesílání požadavků (Repeating Requests - Request Editor)

- **Odevzdání do Request Editoru:** Pravé tlačítko na požadavek v historii ➔ **Open/Resend with Request Editor**.
    
- **Práce v Request Editoru:**
    
    - V rozbalovacím menu **Method** lze jednoduše měnit HTTP metody (`GET`, `POST`, `PUT` atd.).
        
    - Tlačítkem **Send** požadavek odešleš a zobrazíš odpověď.
        
- **Zobrazení v HUD (přímo v prohlížeči):**
    
    - **Replay in Console:** Zobrazí odpověď přímo v panelu HUD.
        
    - **Replay in Browser:** Renderuje odpověď přímo na plochu prohlížeče.
        

## 🔣 5. Kódování a Dekódování (Encoding/Decoding)

- **Nástroj Encoder/Decoder/Hash:**
    
    - Klávesová zkratka **`CTRL + E`**.
        
    - **Záložka Decode:** Automaticky zobrazí zadaný řetězec dekódovaný do více formátů současně (Base64, URL, HTML, Hex, Unicode atd.).
        
    - **Vlastní záložky:** Pomocí tlačítka **Add New Tab** si můžeš sestavit vlastní kombinaci konvertorů pro zrychlení práce.
        
- **Převod v textu požadavku:** Oproti automatickým tabulkám v ZAPu lze vybrané části parametrů před odesláním upravit manuálně nebo nechat ZAP automaticky kódovat URL na pozadí.
    

## 🚀 6. Směrování provozu z příkazové řádky přes ZAP (Proxying Tools)

- **Adresa ZAP Proxy:** `127.0.0.1:8080`
    

### A. Směrování přes `proxychains` (Linux CLI):

1. V souboru `/etc/proxychains.conf` nastav na konec:
    
    Plaintext
    
    ```
    http 127.0.0.1 8080
    ```
    
2. Spusť libovolný příkaz z terminálu:
    
    Bash
    
    ```
    proxychains -q curl http://TARGET_IP:PORT
    ```
    

### B. Směrování z Metasploitu (`msfconsole`):

1. V nastavení modulu nastav proměnnou `PROXIES`:
    
    Plaintext
    
    ```
    msf6 > set PROXIES HTTP:127.0.0.1:8080
    msf6 > run
    ```
    

## ⌨️ Rychlé klávesové zkratky v OWASP ZAP

|**Zkratka**|**Akce**|
|---|---|
|**`CTRL + B`**|Zapnout / vypnout odchytávání (Break / Intercept)|
|**`CTRL + R`**|Otevřít okno automatických úprav (Replacer)|
|**`CTRL + E`**|Otevřít nástroj pro kódování/dekódování (Encoder/Decoder)|
|**`CTRL + F`**|Vyhledávání v požadavcích a odpovědích|