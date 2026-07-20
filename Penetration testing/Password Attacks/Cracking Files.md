# 🔐 Cracking Protected Files & SSH Keys

---

# 📚 Úvod

Při post-exploitation často narazíme na:

- šifrované archivy
    
- chráněné dokumenty
    
- SSH privátní klíče
    
- databáze hesel
    
- zálohy
    

Uživatelé často považují tyto soubory za bezpečné, ale slabá hesla mohou umožnit jejich získání.

Typický workflow:

```text
Najít soubor
      |
      ↓
Extrahovat hash
      |
      ↓
Cracknout hash
      |
      ↓
Použít získané heslo
```

---

# 🔎 1. Vyhledávání Cílů (Enumeration)

První krok je najít zajímavé soubory.

---

# 📄 Vyhledávání Dokumentů

Hledání:

- Microsoft Office
    
- PDF
    
- OpenDocument
    

```bash
for ext in .xls .xlsx .doc .docx .pdf .odt .ods .odp; do
    echo -e "\n[+] Extension: $ext"
    find / -name "*$ext" 2>/dev/null | grep -v "lib\|fonts\|share"
done
```

---

# 🔑 Vyhledávání SSH Privátních Klíčů

SSH klíče často nemají žádnou příponu.

Hledáme podle hlavičky:

```bash
grep -rnE '^\-{5}BEGIN [A-Z0-9 ]+ PRIVATE KEY\-{5}$' /home 2>/dev/null
```

---

Příklady nalezených klíčů:

```text
-----BEGIN RSA PRIVATE KEY-----
```

nebo:

```text
-----BEGIN OPENSSH PRIVATE KEY-----
```

---

## Ověření ochrany SSH klíče

```bash
ssh-keygen -yf <soubor_klice>
```

Pokud se objeví:

```text
Enter passphrase:
```

klíč je chráněný heslem.

---

# ⚙️ 2. X2John Workflow

John the Ripper neumí přímo crackovat soubory.

Proces má vždy dva kroky:

---

## 1. Extrakce hashe

Převede šifrování souboru na crackovatelný hash.

```text
Soubor
   |
   ↓
x2john nástroj
   |
   ↓
Hash
```

---

## 2. Crackování

Hash se následně útočí pomocí:

- John the Ripper
    
- Hashcat
    

---

# 🧰 Přehled X2John Nástrojů

Umístění v Kali:

```text
/usr/share/john/
```

---

|Nástroj|Formát|
|---|---|
|`office2john.py`|Microsoft Office|
|`pdf2john.py`|PDF|
|`zip2john`|ZIP|
|`7z2john.pl`|7-Zip|
|`rar2john`|RAR|
|`ssh2john.py`|SSH klíče|
|`keepass2john`|KeePass databáze|

---

# 🛠️ 3. Praktické Příklady

---

# 📊 A) Microsoft Office

Podporované:

- `.docx`
    
- `.xlsx`
    
- `.pptx`
    

Moderní Office používá:

```text
AES-256
```

---

## Extrakce hashe

```bash
python3 /usr/share/john/office2john.py protected.xlsx > office.hash
```

---

## Crackování

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt office.hash
```

---

# 🔑 B) SSH Privátní Klíč

---

## Extrakce

```bash
python3 /usr/share/john/ssh2john.py id_rsa > ssh.hash
```

---

## Crackování

```bash
john --wordlist=rockyou.txt ssh.hash
```

---

# 📄 C) PDF Dokument

Síla ochrany záleží na verzi PDF.

Starší:

- slabší šifrování
    
- rychlejší crack
    

Novější:

- silnější algoritmy
    
- výrazně pomalejší
    

---

## Extrakce

```bash
pdf2john.py document.pdf > pdf.hash
```

---

## Crackování

```bash
john --wordlist=rockyou.txt pdf.hash
```

---

# 🚀 4. Pro Tips

---

# ⚡ Hashcat GPU Cracking

John je velmi dobrý na CPU.

Pro moderní šifrování:

- Office 2013+
    
- PDF AES
    
- silné archivy
    

je často lepší Hashcat s GPU.

---

## Workflow

```text
Soubor
   |
   ↓
office2john
   |
   ↓
Hash
   |
   ↓
Hashcat GPU
```

---

# Hashcat Módy

|Typ|Hashcat Mode|
|---|---|
|Office 2010|`-m 9500`|
|Office 2013|`-m 9600`|
|PDF 1.7+|`-m 10700`|

---

# 🧹 Čištění Hashů

Některé nástroje přidají do výstupu název souboru.

Například:

```text
file.zip:$zip2$....
```

Hashcat potřebuje pouze:

```text
$zip2$....
```

Jinak může zobrazit:

```text
Signature unmatched
```

---

# 🧠 Vlastní Wordlisty

Pokud běžné slovníky nestačí, vytvoř vlastní.

---

## CeWL

Generování slovníku z webu:

```bash
cewl -w company_wordlist.txt https://www.target-company.com
```

Použití:

- názvy produktů
    
- zaměstnanci
    
- technologie
    
- interní názvy
    

---

# 📋 5. John the Ripper Reference

|Příkaz|Funkce|
|---|---|
|`john hash.txt`|Spustí základní útok|
|`john --show hash.txt`|Zobrazí nalezená hesla|
|`john --list=formats`|Seznam podporovaných formátů|
|`john --rules hash.txt`|Aktivuje pravidla|
|`john --incremental`|Brute-force režim|
|`john --restore`|Obnoví přerušený útok|

---

# 🔄 Kompletní Workflow

```text
1. Najít citlivý soubor
          |
          ↓
2. Identifikovat typ ochrany
          |
          ↓
3. Použít x2john nástroj
          |
          ↓
4. Získat hash
          |
          ↓
5. Zvolit John / Hashcat
          |
          ↓
6. Použít nalezené heslo
          |
          ↓
7. Ověřit přístup
```

---

# 🧰 Nejčastější Nástroje

|Nástroj|Účel|
|---|---|
|John the Ripper|CPU cracking|
|Hashcat|GPU cracking|
|CeWL|Tvorba vlastních slovníků|
|hashid|Identifikace hashů|
|ssh-keygen|Kontrola SSH klíčů|
|*2john|Extrakce hashů|
