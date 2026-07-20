# 🔐 Cracking Protected Archives & Volumes

---

# 📚 Úvod

Při post-exploitation se často setkáme s chráněnými:

- ZIP archivy
    
- 7z soubory
    
- RAR archivy
    
- KeePass databázemi
    
- virtuálními disky (VHD/VHDX)
    
- BitLocker svazky
    

Cílem je získat přístup k datům, která byla zabezpečena heslem nebo šifrováním.

Typický scénář:

```text
Chráněný soubor
        |
        ▼
Identifikace ochrany
        |
        ▼
Extrakce hashe
        |
        ▼
Offline cracking
        |
        ▼
Odemčení obsahu
```

---

# 🔒 1. Proč se Používají Archivy a Šifrované Disky?

## Organizace

Seskupení velkého množství souborů:

- ZIP
    
- RAR
    
- 7z
    
- TAR
    

Výhody:

- jednodušší přenos
    
- menší velikost
    
- lepší správa dat
    

---

## Bezpečnost

Použití ochrany:

- hesla
    
- šifrování
    
- přístupové klíče
    

Typicky chráněná data:

- hesla
    
- SSH klíče
    
- osobní údaje
    
- zálohy
    
- dokumenty
    

---

## Firemní Použití

Příklady:

- BitLocker ve Windows
    
- šifrované zálohy
    
- chráněné archivy
    

Důvody:

- ochrana citlivých dat
    
- bezpečnostní politika
    
- legislativa (např. GDPR)
    

---

# 📦 2. Typy Archivů a Ochrany

Ne všechny formáty mají vlastní šifrování.

---

# 🔐 Nativně Šifrované Formáty

|Formát|Ochrana|
|---|---|
|ZIP|AES / ZipCrypto|
|7z|AES-256|
|RAR|AES|
|KDBX|KeePass encryption|

---

# 🔒 Externě Šifrované Formáty

Některé soubory potřebují další vrstvu ochrany.

Příklady:

```text
TAR
 |
 +--> openssl
 |
 +--> gpg
```

---

# 💽 Virtuální Disky

Používané například:

- VHD
    
- VHDX
    
- VMX
    

Mohou být chráněny:

- BitLocker
    
- heslem
    
- klíčem
    

---

# ⚙️ 3. Workflow: Hash Extraction

Každé prolomení ochrany má podobný postup.

---

# A) Extrakce Hashu

John the Ripper obsahuje nástroje typu:

```text
x2john
```

které převedou ochranu souboru na hash.

---

## ZIP

```bash
zip2john file.zip > zip.hash
```

---

## RAR

```bash
rar2john file.rar > rar.hash
```

---

## 7-Zip

```bash
7z2john.pl file.7z > 7z.hash
```

---

## BitLocker

```bash
bitlocker2john -i disk.vhd > bitlocker.hash
```

Výsledek:

```text
$bitlocker$0$...
```

---

# B) Offline Cracking

Místo útoku přímo na soubor se útočí na extrahovaný hash.

Výhody:

- rychlejší
    
- není potřeba pracovat se souborem
    
- možné využít GPU
    

---

# Hashcat

Výhody:

- GPU akcelerace
    
- vysoký výkon
    

---

## BitLocker Mode

```bash
hashcat -m 22100 bitlocker.hash rockyou.txt
```

---

# John the Ripper

Jednodušší alternativa:

```bash
john --wordlist=rockyou.txt bitlocker.hash
```

---

# C) Mounting a Přístup k Datům

Po získání hesla je potřeba data připojit.

Typický proces:

```text
Hash
 |
 ▼
Heslo
 |
 ▼
Dešifrování
 |
 ▼
Mount
 |
 ▼
Čtení dat
```

---

# 💽 Mount Workflow pro VHD + BitLocker

## 1. Vytvoření Loop Zařízení

Nastavení virtuálního zařízení:

```bash
losetup
```

Kontrola:

```bash
losetup -a
```

---

## 2. Dešifrování

Použití:

```bash
dislocker
```

Příklad:

```bash
dislocker -V disk.vhd \
-uHeslo \
-- /mnt/bitlocker
```

---

## 3. Připojení Filesystemu

```bash
mount
```

Například:

```bash
mount /mnt/bitlocker/dislocker-file /mnt/windows
```

---

# 🚩 4. Praktický BitLocker Workflow

## 1. Identifikace

Zjistíme:

```text
Private.vhd
        |
        ▼
BitLocker protected
```

---

## 2. Extrakce Hashu

```bash
bitlocker2john -i Private.vhd > bitlocker.hash
```

Výstup:

```text
$bitlocker$0$...
```

---

## 3. Cracking

Hashcat:

```bash
hashcat -m 22100 bitlocker.hash \
/usr/share/wordlists/rockyou.txt
```

Výsledek:

```text
Password: franxxxxxxxx
```

---

## 4. Dešifrování

Vytvoření loop zařízení:

```bash
losetup
```

Výsledek:

```text
loop0p1
```

---

Použití dislocker:

```bash
dislocker
```

---

## 5. Mount Disku

Obsah je dostupný například:

```text
/run/media/kali/New Volume
```

---

## 6. Získání Dat

Příklad:

```bash
cat "/run/media/kali/New Volume/flag.txt"
```

---

# 🧰 Užitečné Nástroje

|Nástroj|Účel|
|---|---|
|John the Ripper|CPU cracking|
|Hashcat|GPU cracking|
|zip2john|ZIP hash extraction|
|rar2john|RAR hash extraction|
|7z2john|7z hash extraction|
|bitlocker2john|BitLocker hash extraction|
|dislocker|BitLocker decryption|
|losetup|Loop zařízení|
|mount|Připojení filesystemu|

---

# 🧠 Kompletní Workflow

```text
Najdi chráněný soubor
          |
          ▼
Zjisti typ ochrany
          |
          ▼
Použij x2john nástroj
          |
          ▼
Získej hash
          |
          ▼
Hashcat / John
          |
          ▼
Získej heslo
          |
          ▼
Dešifruj
          |
          ▼
Mount
          |
          ▼
Extrahuj data
```

---

# 📌 Název souboru

```text
11_cracking_archives_and_volumes.md
```