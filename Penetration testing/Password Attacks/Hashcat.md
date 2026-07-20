# 🔥 Hashcat Cheat Sheet

---

# 🧠 Co je Hashcat?

Hashcat je jeden z nejrychlejších nástrojů pro testování síly hesel pomocí GPU akcelerace.

Používá se pro:

- audit hesel
    
- testování hashů
    
- bezpečnostní analýzu
    
- obnovu zapomenutých hesel
    

Podporuje:

- MD5
    
- SHA1
    
- SHA256
    
- NTLM
    
- bcrypt
    
- Kerberos hashy
    
- WPA/WPA2
    
- mnoho dalších algoritmů
    

---

# ⚔️ 1. Attack Modes

Hashcat určuje typ útoku parametrem:

```bash
-a <mode>
```

---

## -a 0 — Straight Attack

Klasický slovníkový útok.

Použití:

```bash
hashcat -a 0 -m 0 hash.txt dict.txt
```

Princip:

```text
dictionary
   |
   ↓
password
Password
admin123
welcome
```

---

## -a 1 — Combination Attack

Spojí dvě slovníkové databáze.

```bash
hashcat -a 1 -m 0 hash.txt d1.txt d2.txt
```

Příklad:

```text
Admin + 123
Winter + 2026
```

Výsledek:

```text
Admin123
Winter2026
```

---

## -a 3 — Mask Attack

Brute-force podle přesného vzoru.

```bash
hashcat -a 3 -m 0 hash.txt ?u?l?l?d
```

Příklad masky:

```text
Aaa1
Bbb2
Test5
```

Použití:

- znáš strukturu hesla
    
- chceš omezit prostor hledání
    

---

## -a 6 — Hybrid Attack

Slovník + maska na konci.

```bash
hashcat -a 6 -m 0 hash.txt dict.txt ?d?d
```

Příklad:

```text
Summer12
Admin99
Password01
```

---

## -a 7 — Hybrid Attack

Maska na začátku + slovník.

```bash
hashcat -a 7 -m 0 hash.txt ?d?d dict.txt
```

Příklad:

```text
12Summer
99Admin
01Password
```

---

# 🔤 2. Mask Character Sets

Masky jsou základ Hashcat brute-force útoků.

|Symbol|Význam|Obsah|
|---|---|---|
|`?l`|Malá písmena|abcdef...|
|`?u`|Velká písmena|ABCDEF...|
|`?d`|Číslice|012345...|
|`?s`|Symboly|!@#$%^...|
|`?a`|Vše|písmena + čísla + symboly|

---

## Příklad

Maska:

```bash
?u?l?l?l?d?d
```

Hledá:

```text
Test12
Abcd99
```

---

# 🔎 3. Identifikace Hashů

## Zjištění typu hashe

Pokud neznáš hodnotu `-m`:

```bash
hashcat --help | grep -i ntlm
```

---

Příklad:

```bash
hashcat --help | grep -i sha256
```

---

# 📋 Nejčastější Hash Modes

|Hash|Hashcat Mode|
|---|---|
|MD5|`-m 0`|
|SHA1|`-m 100`|
|SHA256|`-m 1400`|
|NTLM|`-m 1000`|
|bcrypt|`-m 3200`|
|WPA/WPA2|`-m 22000`|

---

# 📊 4. Správa Výsledků

## Zobrazení cracknutých hesel

```bash
hashcat -m 0 hash.txt --show
```

---

## Stav útoku

Během běhu:

|Klávesa|Funkce|
|---|---|
|`s`|Status|
|`p`|Pause|
|`r`|Resume|
|`q`|Quit|

---

## Potfile

Hashcat ukládá výsledky do:

```text
~/.hashcat/hashcat.potfile
```

Výhoda:

Pokud už bylo heslo nalezeno, Hashcat ho nemusí počítat znovu.

---

# 🧩 5. Custom Character Sets

Vlastní znaková sada.

Použití:

- znáš omezenou množinu znaků
    
- chceš zmenšit prostor hledání
    

---

## Příklad

Heslo obsahuje pouze:

```text
a b c 1 2 3
```

Délka:

```text
8 znaků
```

Příkaz:

```bash
hashcat -a 3 -m 0 hash.txt \
-1 abc123 ?1?1?1?1?1?1?1?1
```

---

# ⚡ 6. Pokročilé Techniky

---

# Workload Profile

Nastavení výkonu:

```bash
-w 3
```

Úrovně:

|Hodnota|Výkon|
|---|---|
|1|Nízký|
|2|Standard|
|3|Vysoký|
|4|Extrémní|

---

## Incremental Mask Attack

Pokud neznáš délku hesla:

```bash
hashcat -a 3 -m 0 hash.txt \
--increment \
--increment-min 4 \
--increment-max 8 \
?l?l?l?l?l?l?l?l
```

Hashcat bude zkoušet:

```text
aaaa
aaaaa
aaaaaa
aaaaaaa
aaaaaaaa
```

---

# Rules Attack

Pravidla upravují slova ze slovníku.

Příklad:

```bash
hashcat -a 0 -m 0 hash.txt \
wordlist.txt \
-r /usr/share/hashcat/rules/best64.rule
```

Transformace:

```text
password

Password
password123
PASSWORD!
```

---

# 🚀 Praktický Workflow

```text
Hash
 |
 ↓
Identifikace algoritmu
 |
 ↓
Volba -m režimu
 |
 ↓
Wordlist Attack
 |
 ↓
Rules
 |
 ↓
Hybrid Attack
 |
 ↓
Mask Attack
 |
 ↓
Výsledek
```

---

# 📝 Praktický Příklad

Cíl:

```text
Uživatel: Mark White
```

---

## 1. Vytvoření slovníku

Například:

```text
Mark
White
Baseball
Nexura
Bella
```

Soubor:

```text
mark.txt
```

---

## 2. Hybrid Attack

Slovník + 4 čísla + symbol:

```bash
hashcat -m 0 -a 6 mark.txt ?d?d?d?d?s
```

Testuje:

```text
Mark1234!
White2026#
Bella9999@
```

---

## 3. Rules Attack

Použití pravidel:

```bash
hashcat -m 0 -a 0 mark.txt \
-r /usr/share/hashcat/rules/best64.rule
```

---

# 🧰 Užitečné Příkazy

|Úkol|Příkaz|
|---|---|
|Slovník|`-a 0`|
|Kombinace|`-a 1`|
|Maska|`-a 3`|
|Slovník + konec|`-a 6`|
|Začátek + slovník|`-a 7`|
|Výsledek|`--show`|
|Výkon|`-w 3`|
|Pravidla|`-r rule.file`|
|Status|`s`|

---

# 🧠 John vs Hashcat

||John|Hashcat|
|---|---|---|
|CPU|✅|⚠️|
|GPU|✅|⭐⭐⭐|
|Rules|✅|⭐⭐⭐|
|Masky|Dobré|Výborné|
|Rychlost|Dobrá|Extrémní|
|Automatická detekce|Lepší|Horší|

---