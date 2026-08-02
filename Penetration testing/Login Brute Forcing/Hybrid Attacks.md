## 🔀 Hybridní útoky (Hybrid Attacks)

> **Hlavní myšlenka:** Kombinují sílu slovníkových útoků a útoku hrubou silou. Využívají lidskou tendenci provádět předvídatelné změny v heslech (např. přidání roku nebo speciálního znaku při vynucené změně hesla – `Summer2023` ➔ `Summer2024!`).

### 💡 Jak hybridní útok funguje

1. **Fáze slovníku:** Zkusebně se otestuje základní slovník (wordlist) běžných slov či firemních termínů.
    
2. **Fáze pravidlové modifikace (Brute-Force mod):** Ke slovům ze slovníku se automatizovaně přidávají čísla, speciální znaky nebo se mění velikost písmen na základě vzorů (mutace/pravidla).
    

### 🛠️ Praktické filtrování wordlistu podle politiky hesel

Pokud známe politiku hesel cílové organizace, můžeme rozsáhlý slovník ofiltrovat pomocí regulárních výrazů (`grep` / `regex`). Tím výrazně zmenšíme prostor pro vyhledávání a zrychlíme útok.

#### Příklad politiky hesel:

- **Minimální délka:** 8 znaků
    
- **Obsahuje:** Nejméně 1 velké písmeno, 1 malé písmeno a 1 číslo
    

#### Postup ofiltrování v Linuxu (Řetězení `grep`):

Bash

```
# 1. Stáhnutí rozsáhlého wordlistu
wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/darkweb2017_top-10000.txt

# 2. Filtr na minimální délku 8 znaků
grep -E '^.{8,}$' darkweb2017_top-10000.txt > darkweb2017-minlength.txt

# 3. Filtr na alespoň jedno velké písmeno [A-Z]
grep -E '[A-Z]' darkweb2017-minlength.txt > darkweb2017-uppercase.txt

# 4. Filtr na alespoň jedno malé písmeno [a-z]
grep -E '[a-z]' darkweb2017-uppercase.txt > darkweb2017-lowercase.txt

# 5. Filtr na alespoň jedno číslo [0-9]
grep -E '[0-9]' darkweb2017-lowercase.txt > darkweb2017-number.txt

# Výsledek: Z 10 000 hesel zůstalo pouze 89 hesel splňujících politiku!
wc -l darkweb2017-number.txt
```

> 🎯 **Výsledek:** Zúžení slovníku z **10 000** na **89** vyhovujících kandidátů radikálně šetří výpočetní čas a síťové zdroje.

## 🔑 Credential Stuffing & Recyklace hesel

> **Hlavní myšlenka:** Útočníci využívají fakt, že uživatelé často používají **stejná hesla na více službách**.

### Jak útok probíhá:

1. **Získání dat z úniků:** Útočník získá seznam uniklých dvojic `jméno:heslo` nebo `email:heslo` (z data breachů, phishingu či veřejných databází).
    
2. **Automatizované testování:** Pomocí skriptů nebo specializovaných nástrojů masově zkouší tyto uniklé údaje přihlašovat do jiných populárních služeb (e-shopy, bankovnictví, sociální sítě).
    
3. **Kompromitace účtu:** Pokud uživatel zrecykloval heslo, útočník získá neoprávněný přístup k dalším službám bez nutnosti louskat heslo.