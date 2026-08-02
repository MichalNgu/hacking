### VHost (Virtual Host) vs. Klasická Poddoména

- **Klasická poddoména:** Spoléhá na veřejné DNS záznamy. Pokud nemá záznam na DNS serveru, prohlížeč ani běžný DNS scan ji nenajde.
    
- **VHost (Virtual Host):** Běží na **stejné IP adrese a portu** jako hlavní web. Webový server (např. Apache/Nginx) rozhoduje o tom, jakou stránku vám zobrazí, na základě HTTP hlavičky `Host:`, kterou pošle váš prohlížeč.
    

> **Proč VHost Fuzzing?** Umožňuje odhalit interní, neveřejné poddomény i na cílech, které nemají veřejné DNS záznamy (např. v labech HTB nebo interních firemních sítích).

### Jak funguje VHost Fuzzing v `ffuf`

Místo měnění domény přímo v URL adrese posíláme dotazy na známou IP/adresu serveru a dynamicky měníme HTTP hlavičku `Host:` pomocí přepínače **`-H`**.

#### Základní příkaz:

Bash

```
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'
```

### ⚠️ Problém s falešně pozitivními výsledky (False Positives)

Jak je vidět z ukázky v textu, při spuštění tohoto příkazu vrátí **všechny dotazy ze slovníku HTTP stav `200 OK`**.

- **Důvod:** Webový server při přijetí neznámé hlavičky `Host:` nevrátí chybu `404`, ale zobrazí svou **výchozí (defaultní) stránku** pro danou IP adresu.
    
- **Řešení:** Všechny neexistující VHosty vracejí výchozí stránku, která má **stále stejnou velikost (Size)**. Skutečně existující VHost vrátí odlišný obsah, a tedy i **jinou velikost odpovědi**.
    
- V následujícím kroku se proto používá filtrování podle velikosti (`-fs`), aby se skryly tyto falešné výsledky se stejnou velikostí odpovědi. například: **-fs 950**