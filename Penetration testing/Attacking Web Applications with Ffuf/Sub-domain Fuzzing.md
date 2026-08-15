### Co je fuzzing poddomén?

Poddoména (sub-domain) je jakýkoliv web běžící pod hlavní doménou (např. `photos.google.com` nebo `blog.inlanefreight.com`). Při fuzzingu poddomén se dotazujeme přímo skrze URL na veřejný DNS server a ověřujeme, zda pro danou veřejnou poddoménu existuje platný DNS záznam, který směřuje na fungující IP adresu.

### 1. Slovníky pro DNS fuzzing

V repozitáři **SecLists** naleznete specializované slovníky pro vyhledávání poddomén v adresáři: `/opt/useful/seclists/Discovery/DNS/`

- **Pro běžný / rychlý scan:** `subdomains-top1million-5000.txt`
    
- **Pro hloubkový scan:** Můžete zvolit větší slovníky z této složky (např. top100000 nebo full).
    

### 2. Příkaz pro vyhledávání veřejných poddomén

Zástupný symbol `FUZZ` se umístí na začátek doménového jména na pozici poddomény:

Bash

```
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/
```

- **Úspěšné výsledky:** Vrátí status kódy jako `200 OK` nebo `301 Moved Permanently` (např. `admin`, `blog`, `support`, `www`).
    

### 3. Proč nefunguje DNS fuzzing na lokálních / interních doménách?

Pokud zkusíte stejný příkaz na interní nebo HTB doménu (např. `[http://FUZZ.academy.htb/](http://FUZZ.academy.htb/)`), vrací `ffuf` 100% chyb a žádné nalezené výsledky.

- **Důvod:** `ffuf` při tomto typu dotazu posílá požadavky na **veřejné DNS servery**.
    
- Pokud jste přidali `academy.htb` pouze do lokálního souboru `/etc/hosts`, veřejné DNS servery o existenci interních poddomén nevědí.
    
- Pro vyhledávání poddomén na lokálních IP adresách nebo v interních sítích je nutné použít techniku **VHost Fuzzing** (přes HTTP hlavičku `Host`), což probírá následující kapitola.