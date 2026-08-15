## 📌 Co je to Cross-Site Scripting (XSS)?

**Cross-Site Scripting (XSS)** je zranitelnost webových aplikací, která vzniká v důsledku **nedostatečné sanitizace a ošetření uživatelského vstupu**.

Pokud aplikace správně neošetří data vložená uživatelem (např. v komentáři, vyhledávacím poli nebo profilu), umožní útočníkovi vložit do stránek **vlastní JavaScriptový kód**. Ten se následně spustí v prohlížeči každého uživatele, který danou stránku navštíví.

```
[ Útočník ] ──( Vložení škodlivého JS )──> [ Webová aplikace ]
                                                  │
                                          ( Uložení / Odraz )
                                                  │
[ Oběť ] <──( Načtení stránky a spuštění JS )──────┘
```

> ⚙️ **Klientská zranitelnost:** XSS se vykonává výhradně na straně klienta (v prohlížeči). Nepředstavuje přímé ohrožení pro samotný backendový server, ale představuje **střední až vysoké riziko pro uživatele aplikace**.

## 💥 Typické dopady a příklady útoků

Spuštěný JavaScript má v rámci prohlížeče oběti široké možnosti:

- **Krádež relace (Session Hijacking):** Odeslání relačních cookies (`document.cookie`) na server útočníka.
    
- **Neautorizované akce:** Vykonání nechtěných API požadavků v jménu oběti (např. změna hesla, e-mailu nebo převod financí).
    
- **Sociální inženýrství:** Zobrazení falešných přihlašovacích formulářů (Phishing) přímo na legitimním webu.
    
- **XSS Červi (Self-propagating worms):** Útoky, které automaticky šíří škodlivý kód dál (např. _Samy Worm_ na MySpace v roce 2005 nebo incident na Twitter/TweetDeck v roce 2014).
    

## 📑 3 základní typy XSS zranitelností

| **Typ XSS**                               | **Popis mechanismu**                                                                                     | **Příznaky a výskyt**                                |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| **Stored (Trvalé / Persistent)**          | **Nejnebezpečnější typ.** Vstup je uložen do databáze a následně se načítá všem uživatelům.              | Komentáře, příspěvky ve fóru, profilové informace.   |
| **Reflected (Odražené / Non-Persistent)** | Vstup se neukládá do DB, ale server jej ihned vrátí v odpovědi (odrazí).                                 | Výsledky vyhledávání, chybové hlášky, URL parametry. |
| **DOM-based (Klientské)**                 | Vstup je zpracován **výhradně na straně klienta** (v prohlížeči) pomocí JavaScriptu, bez účasti serveru. | Práce s `window.location`, parametry za `#`          |