## 📖 Dictionary Attacks

> **Hlavní myšlenka:** Čistý brute-force útok je univerzální, ale extrémně náročný na čas a zdroje. Slovníkový útok využívá lidskou tendenci volit snadno zapamatovatelná hesla a testuje pouze předem připravený seznam potenciálních hesel (wordlist).

### 🆚 Brute Force vs. Dictionary Attack

|**Vlastnost**|**Slovníkový útok (Dictionary Attack)**|**Útok hrubou silou (Brute Force)**|
|---|---|---|
|**Efektivita**|Výrazně rychlejší a méně náročný na zdroje.|Extrémně časově a výpočetně náročný.|
|**Cílení**|Lze přesně přizpůsobit cíli (OSINT, zájmy, název firmy).|Nemá žádné cílení – zkouší všechny kombinace.|
|**Úspěšnost**|Mimořádně efektivní proti běžným a slabým heslům.|Teoreticky 100% úspěšnost, pokud je dostatek času a výkonu.|
|**Omezení**|Zcela selže proti náhodně vygenerovaným komplexním heslům.|V praxi neaplikovatelný na dlouhá a složitá hesla ($12+$ znaků).|

### 📚 Seznamy slov (Wordlists) a jejich zdroje

Úspěch slovníkového útoku přímo závisí na kvalitě a relevantnosti použitého slovníku.

1. **Veřejně dostupné slovníky:**
    
    - **`rockyou.txt`:** Jeden z nejznámějších seznamů obsahující miliony reálných hesel uniklých z databáze RockYou.
        
    - **SecLists:** Repozitář obsahující specializované seznamy pro různé scénáře (např. `2023-200_most_used_passwords.txt`, `top-usernames-shortlist.txt`).
        
2. **Výchozí přihlašovací údaje (Default Credentials):**
    
    - Seznamy továrních hesel pro routery, kamery a aplikovaný software (`default-passwords.txt`).
        
3. **Vlastní slovníky (Custom Wordlists):**
    
    - Vytvářené na míru na základě informací získaných během fáze průzkumu (OSINT) – např. jméno firmy, jména zaměstnanců, oborový slang nebo zájmy cílů.
        

### 🐍 Praktický příklad: Dictionary Attack v Pythonu

Na rozdíl od číselného řetězce (PINu) skript načítá slova z existujícího seznamu (např. ze SecLists) a zkouší je jako heslo proti formuláři.

#### Princip fungování skriptu:

1. Skript stáhne textový soubor se seznamem běžných hesel (např. `500-worst-passwords.txt`).
    
2. Prochází heslo po heslu a odesílá HTTP POST požadavek na endpoint `/dictionary` s parametrem `password=HESLO`.
    
3. Po každém požadavku vyhodnotí odpověď serveru. Pokud odpověď obsahuje klíč `flag`, útok je úspěšný a skript vypíše nalezené heslo i vlajku.