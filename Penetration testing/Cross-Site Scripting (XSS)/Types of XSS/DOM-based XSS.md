## 📌 Co je DOM-based XSS?

**DOM-based XSS** je druhý typ nepřetrvávaného (Non-Persistent) XSS. Na rozdíl od Reflected XSS **vůbec neodesílá data na backendový server**. Celé zpracování vstupu probíhá výhradně v prohlížeči uživatele pomocí JavaScriptu, který manipuluje se strukturou stránek – **Document Object Model (DOM)**.

- **Absence HTTP provozu:** Vstup se často předává přes URL s využitím znaku `#` (anchor/fragment), např. `[http://example.com/#task=vstup](http://example.com/#task=vstup)`. Vše za `#` prohlížeč serveru neodesílá.
    
- **Zdrojový kód stránky (`CTRL+U`):** Vložení textu ve zdrojovém kódu zjištěném ze serveru **nenajdete**, protože se do stránky vykresluje až dynamicky na klientské straně.
    
- **Inspekce prvků (`CTRL+SHIFT+C`):** Pro zobrazení vygenerovaného kód musíte použít vývojářské nástroje (Web Inspector / DOM Explorer).
    

## 🔗 Pojmy Source & Sink

Pro odhalení DOM XSS je klíčové porozumět relaci mezi zdrojem vstupu a jeho vykreslením:

- **Source (Zdroj):** Místo, odkud JavaScript načítá uživatelský vstup (např. `document.URL`, `location.search`, `location.hash`, formulářové pole).
    
- **Sink (Cíl / Vykreslení):** JavaScriptová funkce nebo vlastnost, která načtený vstup zapíše do DOM struktury.
    

### Často zranitelné Sinks (funkce)

- **Přímo v JavaScriptu:** `document.write()`, `DOM.innerHTML`, `DOM.outerHTML`
    
- **V knihovně jQuery:** `.add()`, `.after()`, `.append()`, `.html()`
    

Pokud Sink přijme nedostatečně ošetřený vstup ze Source, dochází k DOM XSS zranitelnosti.

## ⚡ Realizace útoku (DOM Attacks)

Při manipulaci skrze vlastnost `innerHTML` prohlížeče z bezpečnostních důvodů **nevykonají** klasický tag `<script>`. Je proto nutné použít alternativní vektory, které spoléhají na události (event handlers) jiných HTML prvků.

### Obvyklý payload

HTML

```
<img src="" onerror="alert(window.origin)">
```

- **Princip:** Vytvoří obrázek s neplatným/prázdným zdrojem (`src=""`). Při selhání načtení se okamžitě vyvolá událost `onerror`, která spustí zadaný JavaScript.
    

## 📊 Porovnání typů XSS

| **Vlastnost**                 | **Stored XSS** | **Reflected XSS** | **DOM-based XSS**                      |
| ----------------------------- | -------------- | ----------------- | -------------------------------------- |
| **Ukládání do DB**            | Ano            | Ne                | Ne                                     |
| **Zpracování serverem**       | Ano            | Ano               | **Ne (pouze klient)**                  |
| **Přítomnost v HTTP provozu** | Ano            | Ano               | **Ne** (při použití `#`)               |
| **Využitelný vektory**        | Všechny        | Všechny           | HTML události (`onerror`, `onload`...) |
