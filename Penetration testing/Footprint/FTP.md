# FTP (File Transfer Protocol) – Port 21

FTP je **client-server protokol pro přenos souborů**. Používá dvě spojení:

- **Port 21 (Control Channel)**
    - příkazy: `USER`, `PASS`, `LIST`, `GET`, `PUT`
    - zůstává otevřený
- **Port 20 / náhodný port (Data Channel)**
    - přenos souborů

FTP **nešifruje komunikaci**, takže hesla mohou být odposlechnuta.

---

# FTP Modes

## Active Mode (PORT)

Server se připojuje zpět ke klientovi.

```
Client ---> Server :21
Client <--- Server :20
```

Problém:

- firewall/NAT blokace

---

## Passive Mode (PASV)

Klient se připojuje na port serveru.

```
Client ---> Server :21
Client ---> Server :random port
```

Dnes nejčastější.

---

# Enumerace

## Zjištění verze

```
nmap -sV -p21 TARGET
```

---

## Anonymous Login

Kontrola:

```
nmap --script ftp-anon -p21 TARGET
```

Ruční test:

```
ftp TARGET
```

Login:

```
anonymous
anonymous
```

---

## Banner Grab

```
nc -nv TARGET 21
```

Ukáže:

- verzi FTP serveru
- možné exploity

---

# FTP Commands

Připojení:

```
ftp TARGET
```

Soubory:

```
ls -la      # seznam souborů
pwd         # aktuální složka
cd folder   # změna složky
get file    # stáhnout
put file    # nahrát
binary      # binární režim
```

---

# Exploitace

## Anonymous Access

Možnosti:

- čtení citlivých souborů
- nalezení hesel
- upload souborů

---

## Writable FTP Directory

Scénář:

```
FTP Upload
     |
Web Server
     |
Reverse Shell
```

Test:

```
put shell.php
```

---

## Brute Force

Hydra:

```
hydra -l admin -P rockyou.txt ftp://TARGET
```

---

## Sniffing

FTP posílá hesla plaintext:

```
USER admin
PASS password
```

Nástroje:

- Wireshark
- tcpdump

Filtr:

```
tcp.port == 21
```

---

# Známé exploity

## vsFTPd 2.3.4 Backdoor

Verze:

```
vsFTPd 2.3.4
```

Historický exploit:

```
username:)
```

Výsledek:

- root shell
- port 6200

---

# FTP Pentest Checklist

☐ Port 21 otevřený  
☐ Verze FTP serveru  
☐ Anonymous login  
☐ Přístupné soubory  
☐ Upload práva  
☐ Slabá hesla  
☐ Citlivé soubory (`config`, `backup`, `id_rsa`)  
☐ Staré verze + CVE