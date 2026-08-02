## 🧮 Matematika útoků hrubou silou

Úspěšnost a časová náročnost prolomení hesla nebo PINu závisí na celkovém počtu všech možných kombinací. Ten je definován vzorcem:

$$\text{Počet kombinací} = N^L$$

- $N$ = Velikost abecedy / množiny znaků (Character Set Size)
    
- $L$ = Délka hesla (Password Length)
    

### 📊 Porovnání složitosti hesel

S rostoucí délkou a rozšiřováním množiny znaků roste prostor pro vyhledávání **exponenciálně**:

| **Délka (L)** | **Množina znaků (N)**                 | **Vzorec** | **Celkový počet kombinací**                     |
| ------------- | ------------------------------------- | ---------- | ----------------------------------------------- |
| **6**         | Malá písmena (`a-z`) → $N=26$         | $26^6$     | **308 915 776** (~300 miliónů)                  |
| **8**         | Malá písmena (`a-z`) → $N=26$         | $26^8$     | **208 827 064 576** (~200 miliard)              |
| **8**         | Malá + velká písmena (`a-Z`) → $N=52$ | $52^8$     | **53 459 728 531 456** (~53 bilionů)            |
| **12**        | Písmena, čísla, symboly → $N=94$      | $94^{12}$  | **475 920 493 781 698 549 504** (~475 trilionů) |

### ⚡ Vliv výpočetního výkonu

Rychlost prolomení závisí na tom, kolik pokusů za sekundu dokáže útočník provést:

- **Běžný počítač (~1 milion pokusů/s):**
    
    - Stačí na rychlé prolomení jednoduchých hesel.
        
    - Prolomení 8místného hesla z písmen a čísel by trvalo cca **6,9 roku**.
        
- **Superpočítač / Grafický klaster (~1 trilion pokusů/s):**
    
    - Dramaticky zkracuje čas pro kratší a střední hesla.
        
    - Přesto prolomení 12místného kompletního ASCII hesla trvá i superpočítači cca **15 000 let**.
        

## 🐍 Praktický příklad: Brute-forcing 4místného PINu

U 4místného číselného PINu je počet kombinací pouze $10^4 = 10\,000$ (od `0000` do `9999`). Takto malý prostor lze velmi snadno projít jednoduchým skriptem v Pythonu.

### Princip fungování skriptu:

1. Skript prochází čísla od `0` do `9999`.
    
2. Každé číslo formátuje na 4místný řetězec s úvodními nulami (např. `7` ➔ `"0007"`).
    
3. Odesílá HTTP GET požadavek na endpoint (např. `/pin?pin=0007`).
    
4. Kontroluje odpovídající stavový kód a JSON odpověď – při nalezení shody vypíše PIN a získanou vlajku (flag).