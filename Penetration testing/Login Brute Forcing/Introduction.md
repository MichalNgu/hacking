## 🔑 Brute Forcing (Útok hrubou silou)

**Co to je:** Metoda pokus-omyl, při které útočník systematicky zkouší všechny možné kombinace znaků, hesel nebo klíčů, dokud nenajde ten správný.

### 🛠️ Typy útoků hrubou silou

Brute forcing není jedna jediná metoda. Podle situace a dostupných dat volíme různé přístupy:

|**Metoda**|**Jak funguje**|**Kdy se nejlépe hodí**|
|---|---|---|
|**Simple Brute Force**|Systematicky zkouší úplně všechny kombinace znaků v dané délce (např. `aaa` až `zzz`).|Nemáme žádné informace o heslu, ale máme obrovský výpočetní výkon.|
|**Dictionary Attack**|Používá předem připravený seznam častých hesel (např. `rockyou.txt`).|Cíl pravděpodobně používá slabé nebo běžné heslo.|
|**Hybrid Attack**|Kombinuje slovník s pravidly (např. přidává čísla/znaky na konec slova ze slovníku).|Uživatel použil mírně upravené běžné heslo (např. `Heslo123!`).|
|**Credential Stuffing**|Využívá uniklá jména a hesla z jiných databází (předpokládá opakování hesel).|Máme k dispozici únik dat (Data Breach) a cíl recykluje hesla.|
|**Password Spraying**|Zkouší jedno velmi časté heslo (např. `Summer2024!`) proti obrovskému množství účtů.|Systém má nastavené blokování účtů po 3 špatných pokusech (vyhne se detekci).|
|**Rainbow Table Attack**|Porovnává zachycené hashe s předpočítanou tabulkou hashů.|Potřebujeme prolomit velké množství hashů a máme dostatek místa na disku.|
|**Reverse Brute Force**|Známe jedno heslo a zkoušíme ho proti různým uživatelským jménům.|Podezření na masové používání jednoho konkrétního hesla.|
|**Distributed Brute Force**|Rozděluje zátěž zkoušení hesel mezi více počítačů/strojů současně.|Heslo je velmi složité a jediný stroj by ho louskal příliš dlouho.|
