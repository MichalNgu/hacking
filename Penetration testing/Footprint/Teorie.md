# Teorie Enumerace

## 📋 Základní principy

|Princip|Co si pamatovat|
|---|---|
|**1. Je toho víc, než vidíš.**|Pokud port vypadá zavřeně, zkus jinou techniku skenování, časování nebo další zdroje informací.|
|**2. Rozlišuj viděné a skryté.**|Firewall, filtrování nebo absence odpovědi často poskytují stejně cenné informace jako otevřený port.|
|**3. Vždy je cesta dál.**|Pokud se neposouváš technicky, hledej další zdroje informací (dokumentace, veřejné informace, lidský faktor v rámci autorizovaného testu).|

---

# 🛡️ 6 Vrstev Enumerace

## 🌐 Vrstva 1 – Internet Presence (Kde jsou?)

**Role:** Digitální detektiv

### Cíl

Vytvořit co nejúplnější seznam domén, subdomén a IP adres.

### Hledáš

|Prvek|Proč je důležitý|
|---|---|
|Domény|Hlavní vstupní body organizace|
|Subdomény|Často odhalí testovací nebo interní systémy|
|ASN|Napoví rozsah infrastruktury|
|Cloud služby|Mohou obsahovat další aktiva organizace|

### Výstup

- Seznam domén
    
- Seznam IP adres
    
- Mapa internetové infrastruktury
    

---

## 🚧 Vrstva 2 – Gateway (Jak se chrání?)

### Cíl

Zjistit, jaké bezpečnostní prvky stojí mezi tebou a cílem.

### Hledáš

|Technologie|Účel|
|---|---|
|Firewall|Filtruje provoz|
|WAF|Chrání webové aplikace|
|VPN|Omezuje přístup|
|Reverse Proxy|Skrývá backendové služby|
|CDN / Cloud ochrana|Chrání infrastrukturu|

### Otázka

> Komunikuješ skutečně s cílovým serverem, nebo pouze s ochrannou vrstvou?

---

## ⚙️ Vrstva 3 – Accessible Services (Co nabízejí?)

### Cíl

Zjistit, které služby jsou dostupné a jaké verze používají.

### Hledáš

|Informace|Příklad|
|---|---|
|Otevřené porty|22, 80, 443|
|Služby|SSH, HTTP, SMB|
|Verze|Apache 2.4.41|
|Operační systém|Linux, Windows|

### Myšlenka

Každá služba existuje z nějakého důvodu. Pokud pochopíš její účel, lépe pochopíš i data, která zpracovává.

---

## 🔄 Vrstva 4 – Processes (Co se tam děje?)

### Cíl

Pochopit tok dat mezi službami.

### Hledáš

|Otázka|Příklad|
|---|---|
|Odkud přichází data?|Webový formulář|
|Kam putují?|Databáze|
|Jak spolu komunikují služby?|HTTP → API → SQL|

### Výstup

Mapování vztahů:

```text
Uživatel
   ↓
Web Server
   ↓
API
   ↓
Databáze
```

---

## 🔑 Vrstva 5 – Privileges (Kdo to ovládá?)

### Cíl

Zjistit, pod jakými účty běží jednotlivé služby.

### Hledáš

|Účet|Riziko|
|---|---|
|root|Maximální oprávnění|
|SYSTEM|Maximální oprávnění ve Windows|
|Administrator|Vysoká oprávnění|
|www-data|Omezený servisní účet|
|service account|Specifická oprávnění|

### Zaměření

- Oprávnění souborů
    
- Chybné konfigurace
    
- Přístupová práva
    
- Delegace oprávnění
    

---

## 💻 Vrstva 6 – OS Setup (Jak je to postavené?)

### Cíl

Pochopit konfiguraci systému.

### Hledáš

|Oblast|Příklad|
|---|---|
|Kernel|Verze jádra|
|Konfigurace|Konfigurační soubory|
|Síť|Routy, DNS, firewall|
|Historie|Historie příkazů|
|Logy|Provoz a chyby|
|Automatizace|Cron, Scheduled Tasks|

### Výstup

Kompletní přehled o fungování systému a jeho správě.

---

# Připojení k systémům

## SSH pomocí hesla

```bash
ssh uzivatel@IP_ADRESA
```

## SSH pomocí privátního klíče

```bash
chmod 600 id_rsa
ssh -i id_rsa uzivatel@IP_ADRESA
```

## RDP (xfreerdp)

```bash
xfreerdp /v:IP_ADRESA /u:uzivatel /p:heslo /dynamic-resolution +clipboard
```