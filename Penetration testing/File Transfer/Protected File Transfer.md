# 🔐 Protected File Transfers – Doplnění pro CPTS

## 1. Nejlepší volba: použít šifrovaný protokol

Pořadí preferencí:

|Priorita|Metoda|Šifrování|
|---|---|---|
|🥇|SFTP|SSH/TLS|
|🥈|SCP|SSH|
|🥉|HTTPS|TLS|
|4|SMB3|Encryption možné|
|5|FTP + šifrování souboru|žádné TLS nutně|
|6|HTTP + AES soubor|ruční ochrana|

---

# 2. SSH / SCP (nejčistší řešení)

## Upload

```
scp secret.txt user@10.10.10.10:/tmp/
```

## Download

```
scp user@10.10.10.10:/tmp/secret.txt .
```

Výhody:

- šifrovaný přenos
- autentizace
- integrita dat

---

# 3. SFTP

Interaktivní přenos:

```
sftp user@10.10.10.10
```

Příkazy:

```
ls
cd /tmp
put file.txt
get secret.txt
```

---

# 4. OpenSSL – lepší workflow

## Zašifrování

```
openssl enc \
-aes-256-cbc \
-pbkdf2 \
-iter 100000 \
-salt \
-in secret.txt \
-out secret.enc
```

Parametry:

|Parametr|Význam|
|---|---|
|aes-256-cbc|AES 256bit|
|pbkdf2|ochrana proti brute-force|
|iter|počet derivací hesla|
|salt|náhodná hodnota proti rainbow tabulkám|

---

## Dešifrování

```
openssl enc \
-d \
-aes-256-cbc \
-pbkdf2 \
-iter 100000 \
-in secret.enc \
-out secret.txt
```

---

# 5. Kontrola integrity pomocí hashů

Před přenosem:

```
sha256sum secret.txt
```

Výstup:

```
a8f23ab932.... secret.txt
```

Po přenosu:

```
sha256sum secret.txt
```

Musí být stejný.

---

# 6. Kombinace šifrování + HTTP

Situace:

- není SSH
- není SMB
- pouze port 80

Řešení:

## Útočník:

```
openssl enc -aes-256-cbc -pbkdf2 \
-in loot.zip \
-out loot.zip.enc
```

Server:

```
python3 -m http.server 8000
```

Cíl:

```
wget http://10.10.10.10/loot.zip.enc
```

Potom:

```
openssl enc -d -aes-256-cbc -pbkdf2 \
-in loot.zip.enc \
-out loot.zip
```

---

# 7. GPG alternativa

Na Linuxu velmi časté:

## Šifrování:

```
gpg -c secret.txt
```

Výsledek:

```
secret.txt.gpg
```

## Dešifrování:

```
gpg -d secret.txt.gpg
```

Výhoda:

- jednoduché
- standardní
- podporuje asymetrickou kryptografii

---

# 8. Windows – certifikáty a ZIP ochrana

Pokud nemáš AES skript:

## PowerShell ZIP:

```
Compress-Archive data.txt data.zip
```

Ale pozor:

- ZIP není automaticky šifrovaný

Lepší:

7-Zip:

```
7z a -pStrongPassword -mhe archive.7z data.txt
```

Výhody:

- AES-256
- šifrování názvů souborů (`-mhe`)

---

# 9. Exfiltrace přes DNS (nouzová metoda)

Pokud je blokovaný HTTP/SSH:

Schéma:

```
Soubor
 ↓
gzip
 ↓
base64
 ↓
DNS queries
 ↓
Útočník
```

Nástroje:

- dnscat2
- iodine

Použití:

- malé množství dat
- obejití firewall pravidel

---

# 10. CPTS rozhodovací strom

```
Potřebuji přenést data
          |
          |
          +-- SSH dostupné?
          |       |
          |       +-- SCP/SFTP
          |
          +-- HTTPS?
          |       |
          |       +-- HTTPS upload/download
          |
          +-- SMB?
          |       |
          |       +-- SMB transfer
          |
          +-- Pouze HTTP?
          |       |
          |       +-- AES encrypt + HTTP
          |
          +-- Nic?
                  |
                  +-- Base64
                  +-- DNS exfil
```

---

# 🛠️ Protected Transfer Checklist (CPTS)

```
[ ] Použil jsem SSH/SFTP místo HTTP?
[ ] Pokud HTTP, zašifroval jsem soubor?
[ ] Použil jsem silné unikátní heslo?
[ ] Ověřil jsem SHA256 hash?
[ ] Odstranil jsem dočasné kopie?
[ ] Neobsahují data reálné PII?
```