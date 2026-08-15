## ⚙️ 1. Nastavení a Základy (Proxy Setup)

- **Adresa lokální proxy:** `127.0.0.1:8080` (lze změnit v `Proxy` ➔ `Proxy settings` ➔ `Tools` ➔ `Proxy` ➔ `Proxy listeners`).
    
- **Zabudovaný prohlížeč (Burp Browser):** Najdeš v `Proxy` ➔ `Intercept` ➔ **Open Browser** (nevyžaduje ruční nastavování proxy ani certifikátu).
    
- **Instalace CA certifikátu (pro vlastní prohlížeč):**
    
    1. Zapni proxy a v prohlížeči navštiv adresu `http://burpsuite`.
        
    2. Vpravo nahoře klikni na **CA Certificate** a stáhni soubor `cacert.der`.
        
    3. V nastavení prohlížeče (např. Firefox: `about:preferences#privacy` ➔ _View Certificates_) importuj certifikát do záložky _Authorities_ a zaškrtni důvěru pro webové stránky.
        

## 🛑 2. Odchytávání požadavků a odpovědí (Intercepting Web Requests & Responses)

- **Zapnutí / Vypnutí odchytávání:** Tlačítko `Intercept is on` / `Intercept is off` v záložce `Proxy` ➔ `Intercept`.
    
- **Práce se zachyceným požadavkem:**
    
    - **Forward:** Odesle zachycený požadavek dále na server.
        
    - **Drop:** Zahodí požadavek (nedorazí na server).
        
    - **Action:** Nabízí pokročilé akce (předání do jiných nástrojů, úprava atd.).
        

### Odchytávání odpovědí serveru (Intercepting Responses):

- **Jednorázově:** V menu `Action` u zachyceného požadavku zvol **Do intercept** ➔ **Response to this request**.
    
- **Trvale (Pravidlo):** V `Proxy settings` ➔ `Response interception rules` zaškrtni _Intercept responses based on the following rules_.
    

## 🔄 3. Automatické úpravy (Automatic Modification - Match and Replace)

> **Kde to najdeš:** `Proxy` ➔ `Proxy settings` ➔ sekce **Match and Replace** ➔ **Add**.

- **Změna User-Agenta (Obcházení filtrů):**
    
    - **Item:** `Request header`
        
    - **Match:** `^User-Agent.*$` _(zapnout `Regex match`)_
        
    - **Replace:** `User-Agent: HackTheBox Agent 1.0`
        
- **Obcházení HTML formulářů (Úprava odpovědi):**
    
    - **Item:** `Response body`
        
    - **Match:** `type="number"` ➔ **Replace:** `type="text"`
        
    - **Match:** `disabled` ➔ **Replace:** _(nechat prázdné)_
        
- **Automatická úprava těla požadavku (POST body):**
    
    - **Item:** `Request body`
        
    - **Match:** `ip=1` ➔ **Replace:** `ip=1;ls;`
        

## 🔁 4. Opakované odesílání požadavků (Repeating Requests - Repeater)

- **Odevzdání do Repeateru:**
    
    - Označ požadavek v `HTTP history` nebo `Interceptu` a stiskni **`CTRL + R`**.
        
    - Přejdi do záložky **Repeater** stisknutím **`CTRL + SHIFT + R`**.
        
- **Práce v Repeateru:**
    
    - Tlačítko **Send** odesílá požadavek.
        
    - Pravé tlačítko ➔ **Change Request Method**: Rychle přepne metodu mezi `GET` a `POST` (případně automaticky přesune parametry mezi URL a tělem požadavku).
        
    - Přepínání záložek zobrazení odpovědi: **Pretty** (přehledný kód), **Raw** (čistý text), **Render** (grafické vykreslení stránky).
        

## 🔣 5. Kódování a Dekódování (Encoding/Decoding)

- **Rychlé URL kódování v textu:** Označ text a stiskni **`CTRL + U`** _(kóduje klíčové znaky jako mezeru, `&`, `#`)_.
    
- **Nástroj Decoder (Záložka v hlavní liště):**
    
    - Vlož text ➔ Zvol **Decode as...** nebo **Encode as...** (Base64, URL, HTML, Hex atd.).
        
    - Výstup lze řetězit (např. po dekódování z Base64 upravíš text a dole zvolíš _Encode as > Base64_).
        
- **Nástroj Inspector (Pravý panel v Proxy / Repeateru):**
    
    - Automaticky rozbalí a dekóduje označené parametry, hlavičky nebo cookies přímo při prohlížení požadavku. Dovoluje hodnotu upravit a automaticky ji zakóduje zpět.
        

## 🚀 6. Směrování provozu z příkazové řádky přes Burp (Proxying Tools)

- **Lokalní adresa Burp Proxy:** `127.0.0.1:8080`
    

### A. Směrování přes `proxychains` (Linux CLI):

1. V souboru `/etc/proxychains.conf` nastav na konec:
    
    Plaintext
    
    ```
    http 127.0.0.1 8080
    ```
    
2. Spusť libovolný příkaz:
    
    Bash
    
    ```
    proxychains -q curl http://TARGET_IP:PORT
    ```
    

### B. Směrování z Metasploitu (`msfconsole`):

1. V použitém modulu nastav proměnnou proxy:
    
    Plaintext
    
    ```
    msf6 > set PROXIES HTTP:127.0.0.1:8080
    msf6 > run
    ```
    

## ⌨️ Rychlé klávesové zkratky v Burp Suite

| **Zkratka**            | **Akce**                                                             |
| ---------------------- | -------------------------------------------------------------------- |
| **`CTRL + R`**         | Odeslat požadavek do Repeateru                                       |
| **`CTRL + SHIFT + R`** | Otevřít záložku Repeater                                             |
| **`CTRL + U`**         | URL-encode označeného textu                                          |
| **`CTRL + F`**         | Vyhledávání v požadavku / odpovědi                                   |
| **`CTRL + I`**         | Odeslat požadavek do Intruderu _(pro automatické skenování/fuzzing)_ |