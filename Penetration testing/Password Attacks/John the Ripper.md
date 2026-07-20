# 🔓 John the Ripper Cheat Sheet

---

# 🛠️ Co je John the Ripper?

John the Ripper (JtR) je nástroj určený k testování síly hesel a obnově hesel z hashů.

Podporuje:

- Linux hashy
    
- Windows hashy
    
- ZIP archivy
    
- PDF dokumenty
    
- SSH klíče
    
- KeePass databáze
    
- RAR archivy
    
- mnoho dalších formátů
    

---

# 🎯 1. Základní Cracking Modes

## Single Crack Mode

První režim, který je vhodné vyzkoušet.

Využívá:

- uživatelská jména
    
- GECOS informace
    
- jednoduché variace
    

```bash
john --single hash.txt
```

---

## Wordlist Mode

Slovníkový útok.

```bash
john --wordlist=dict.txt hash.txt
```

Použití:

- známý slovník
    
- běžná hesla
    
- firemní slovníky
    

---

## Wordlist + Rules

Použije slovník a automatické modifikace hesel.

Například:

```text
password
Password
password1
Password123
P@ssword
```

Příkaz:

```bash
john --wordlist=dict.txt --rules hash.txt
```

---

## Incremental Mode

Brute-force režim.

Zkouší všechny možné kombinace znaků.

```bash
john --incremental hash.txt
```

Vlastnosti:

- nejpomalejší
    
- nejdůkladnější
    
- vhodný jako poslední možnost
    

---

# 🔍 2. Identifikace Formátů

John často rozpozná hash automaticky.

U některých hashů je potřeba formát specifikovat ručně.

---

## Identifikace hashe

```bash
hashid -j <hash>
```

Příklad:

```bash
hashid -j 5f4dcc3b5aa765d61d8327deb882cf99
```

---

## Ruční určení formátu

```bash
john --format=raw-md5 hash.txt
```

Další příklady:

```bash
john --format=raw-sha1 hash.txt
```

```bash
john --format=nt hash.txt
```

---

## Zobrazení všech podporovaných formátů

```bash
john --list=formats
```

---

# 📦 3. Konverze Souborů pomocí *2john

Mnoho formátů je potřeba nejprve převést na hash.

---

## ZIP Archiv

```bash
zip2john archive.zip > archive.hash
```

---

## PDF Dokument

```bash
pdf2john document.pdf > document.hash
```

---

## SSH Klíč

```bash
ssh2john id_rsa > id_rsa.hash
```

---

## RAR Archiv

```bash
rar2john archive.rar > archive.hash
```

---

## KeePass Databáze

```bash
keepass2john Vault.kdbx > vault.hash
```

---

# 📊 4. Práce s Výsledky

John ukládá nalezená hesla do:

```text
~/.john/john.pot
```

---

## Zobrazení nalezených hesel

```bash
john --show hash.txt
```

---

## Zobrazení stavu útoku

Během běhu:

```text
ENTER
```

nebo libovolná klávesa.

---

## Ukončení útoku

```text
q
```

nebo

```text
CTRL + C
```

---

## Obnovení přerušeného útoku

```bash
john --restore
```

---

# ⚙️ 5. Pokročilé Tipy

## Firemní Slovník pomocí CeWL

Vytvoření slovníku z veřejného webu organizace:

```bash
cewl -w company_dict.txt https://www.firma.cz
```

Výsledkem je seznam slov nalezených na webových stránkách.

---

## Minimální Délka Hesla

Pokud znáš minimální délku:

```bash
john --wordlist=rockyou.txt --min-len=8 hash.txt
```

---

## Více CPU Jader

Spuštění více procesů:

```bash
john --fork=4 hash.txt
```

Příklad:

```text
4 procesy
4 CPU jádra
rychlejší testování
```

---

# 🚀 6. Praktický Workflow

Typický postup:

```text
Hash
  │
  ▼
Identifikace formátu
  │
  ▼
Single Mode
  │
  ▼
Wordlist
  │
  ▼
Wordlist + Rules
  │
  ▼
Incremental
```

---

## Krok 1 - Single Mode

```bash
john --single passwd.txt
```

---

## Krok 2 - Wordlist + Rules

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --rules passwd.txt
```

---

## Krok 3 - Výpis Výsledků

```bash
john --show passwd.txt
```

---

# 📌 Nejužitečnější Příkazy

|Účel|Příkaz|
|---|---|
|Single Mode|`john --single hash.txt`|
|Wordlist|`john --wordlist=dict.txt hash.txt`|
|Wordlist + Rules|`john --wordlist=dict.txt --rules hash.txt`|
|Brute Force|`john --incremental hash.txt`|
|Výpis hesel|`john --show hash.txt`|
|Obnovení session|`john --restore`|
|Formáty|`john --list=formats`|
|ZIP → Hash|`zip2john archive.zip > archive.hash`|
|PDF → Hash|`pdf2john document.pdf > document.hash`|
|SSH → Hash|`ssh2john id_rsa > id_rsa.hash`|
|KeePass → Hash|`keepass2john vault.kdbx > vault.hash`|

---

# 🧠 Doporučený Postup

```text
1. Identifikace hashe
        │
        ▼
2. Single Mode
        │
        ▼
3. Wordlist Attack
        │
        ▼
4. Wordlist + Rules
        │
        ▼
5. Vlastní slovník
        │
        ▼
6. Incremental Mode
```