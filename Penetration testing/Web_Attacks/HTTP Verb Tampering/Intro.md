### 🔑 Hlavní koncepty a typy zranitelností

- **Princip útoku:** Útočník zamění očekávanou HTTP metodu za jinou (např. `GET` za `HEAD` nebo `POST`), aby obešel bezpečnostní mehanismy, které kontrolují pouze určité vybrané metody.
    
- **Typ 1: Insecure Web Server Configuration (Špatná konfigurace serveru)**
    
    - Vzniká např. použitím direktivy `<Limit GET POST>` v konfiguračních souborech Apache (`.htaccess`), která vyžaduje autentizaci pouze pro metody `GET` a `POST`.
        
    - Pokud útočník pošle požadavek metodou `HEAD` nebo neznámým slovesem, server autentizaci nevyžaduje, ale aplikace požadavek přesto zpracuje.
        
- **Typ 2: Insecure Coding (Chybná logika v aplikaci)**
    
    - Vzniká v situaci, kdy vývojář sanituje vstup pouze z jednoho pole (např. `$_GET['code']`), ale k vykonání logiky nebo SQL dotazu použije obecné pole `$_REQUEST['code']`.
        
    - Odesláním parametrů přes `POST` útočník zcela obejde validaci v `$_GET`, zatímco backend data z `$_REQUEST` zpracuje.
        

### 📊 Přehled nejběžnějších HTTP metod

| **Metoda**  | **Popis**                          | **Riziko při špatné konfiguraci**                    |
| ----------- | ---------------------------------- | ---------------------------------------------------- |
| **GET**     | Získání obsahu ze serveru          | Standardní metoda, často jediná chráněná filtry      |
| **HEAD**    | Vrací pouze HTTP hlavičky bez těla | Často obchází autentizační filtry určité pro GET     |
| **POST**    | Odeslání dat ke zpracování         | Může obejít filtry kontrolující pouze GET parametry  |
| **PUT**     | Zápis/nahrazení souboru na serveru | Umožňuje přímé nahrání škodlivých souborů (webshell) |
| **DELETE**  | Smazání zdroje na serveru          | Umožňuje neoprávněné mazání dat nebo souborů         |
| **OPTIONS** | Vrací seznam podporovaných metod   | Pomáhá útočníkům mapovat dostupné metody serveru     |
