# 🪟 SocksOverRDP – RDP Pivoting přes Dynamic Virtual Channels

## Základní princip

**SocksOverRDP** je technika pivotingu, která zneužívá funkci **RDP Dynamic Virtual Channels (DVC)**.

RDP běžně používá virtuální kanály pro:

- Clipboard (kopírování)
    
- Zvuk
    
- Tiskárny
    
- Diskové jednotky
    

SocksOverRDP dokáže tyto legitimní RDP kanály využít k přenosu **SOCKS proxy provozu**.

Výsledek:

- RDP spojení vypadá jako běžná administrátorská práce.
    
- Uvnitř RDP relace může proudit další TCP komunikace.
    
- Vytvoříš SOCKS proxy do hlubší části sítě.
    

---

# Kdy použít SocksOverRDP

Použití:

- Máš kompromitovaný Windows stroj.
    
- Máš RDP přístup na další interní server.
    
- Nemůžeš použít:
    
    - SSH
        
    - Chisel
        
    - Socat
        
    - Meterpreter pivoting
        

Typický scénář:

```
KALI
 |
 |
RDP 3389
 |
 |
Windows Jump Host
 |
 |
RDP Session
 |
 |
Internal Server
```

Pomocí SocksOverRDP uděláš:

```
KALI
 |
 |
RDP
 |
 |
Windows Pivot
 |
 SOCKS5
 |
 |
Internal Network
```

---

# Jak SocksOverRDP funguje

Skládá se ze dvou částí:

## 1. Client plugin

Běží na stroji, odkud spouštíš RDP klienta.

Například:

```
Windows Foothold
```

Registruje se do RDP klienta.

---

## 2. Server komponenta

Běží na vzdáleném RDP serveru.

Například:

```
Internal Windows Server
```

Přijímá SOCKS provoz přes RDP Virtual Channel.

---

# Postup konfigurace

## 1. Registrace RDP pluginu

Na prvním Windows stroji:

CMD jako Administrator:

```cmd
regsvr32.exe SocksOverRDP-Plugin.dll
```

Výsledek:

RDP klient začne podporovat SocksOverRDP.

---

# 2. Připojení přes RDP

Spustíš klasické RDP:

```cmd
mstsc.exe
```

Připojíš se například:

```
172.16.5.19
```

Po úspěšném připojení:

- vytvoří se Dynamic Virtual Channel
    
- otevře se lokální SOCKS proxy
    

Typicky:

```
127.0.0.1:1080
```

---

# 3. Aktivace serveru na cílovém Windows

Na vzdáleném serveru:

Nahraješ:

```
SocksOverRDP-Server.exe
```

Spustíš:

```cmd
SocksOverRDP-Server.exe
```

Nutná práva:

```
Administrator
```

---

# 4. Kontrola SOCKS proxy

Na pivot Windows:

```cmd
netstat -ano | findstr 1080
```

Výsledek:

```
TCP 127.0.0.1:1080 LISTENING
```

SOCKS proxy běží.

---

# 5. Proxy přes Proxifier

Windows nemá proxychains.

Použiješ:

```
Proxifier
```

Konfigurace:

Proxy Server:

```
Address:
127.0.0.1

Port:
1080

Type:
SOCKS5
```

---

# Použití v praxi

Po nastavení můžeš směrovat:

- RDP
    
- SMB
    
- WinRM
    
- LDAP
    
- webové aplikace
    
- skenery
    

Například:

Další RDP server:

```
172.16.6.155
```

Spustíš:

```cmd
mstsc.exe
```

Traffic:

```
mstsc
 |
Proxifier
 |
SOCKS5 127.0.0.1:1080
 |
RDP Virtual Channel
 |
Internal Server
```

---

# Praktické využití

## 1. Double Hop Pivoting

Nejčastější použití.

Situace:

```
Internet

 |
 |
Foothold PC

 |
 |
Jump Server

 |
 |
DC / Database
```

Nemáš přímý přístup na DC.

Řešení:

```
RDP
 +
SocksOverRDP
```

Uděláš z Jump Serveru proxy.

---

# 2. Obcházení firewallu

Firewall vidí:

```
TCP 3389
```

Normální RDP komunikace.

Nevidí:

```
SMB
LDAP
RDP další servery
```

protože jsou uvnitř šifrovaného RDP kanálu.

---

# 3. Pivoting bez instalace síťových nástrojů

Výhoda:

Nepotřebuješ:

- chisel.exe
    
- socat.exe
    
- plink.exe
    

Používáš:

- mstsc.exe
    
- RDP protokol
    

To snižuje šanci detekce.

---

# Nevýhody

## 1. Pomalejší

RDP není vytvořené pro tunelování velkého množství dat.

Problémy:

- vysoká latence
    
- pomalé skeny
    
- pomalejší přenosy
    

---

## 2. Potřebuje RDP

Musíš mít:

- RDP přístup
    
- povolený port 3389
    
- možnost vytvořit relaci
    

---

## 3. Windows only

Nejde použít pro Linux pivoty.

---

# Optimalizace výkonu

RDP spotřebuje hodně bandwidth.

Před připojením:

```
mstsc.exe
```

Options → Experience

Nastavit:

```
Connection speed:
Modem (56 kbps)
```

Vypnout:

- Desktop background
    
- Font smoothing
    
- Animations
    
- Themes
    

Výsledek:

Více bandwidth pro SOCKS tunel.

---

# SocksOverRDP vs Chisel

||SocksOverRDP|Chisel|
|---|---|---|
|Protokol|RDP DVC|HTTP/WebSocket|
|OS|Windows|Linux/Windows|
|Potřebuje účet|RDP účet|Ne|
|SOCKS5|Ano|Ano|
|Detekce|Nízká|Střední|
|Rychlost|Nižší|Vyšší|
|Typické použití|AD pivot|Obecný pivot|

---

# CPTS / HTB zapamatovat

Nejdůležitější:

```
SocksOverRDP =
SOCKS proxy uvnitř RDP Dynamic Virtual Channel
```

Používá se když:

```
Mám RDP přístup
+
potřebuji další pivot
+
nemohu použít klasické tunely
```

Workflow:

```
regsvr32 plugin
        ↓
mstsc RDP
        ↓
SocksOverRDP server
        ↓
SOCKS5 :1080
        ↓
Proxifier
        ↓
Internal network
```