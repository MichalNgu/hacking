| Cesta  | Popis                                                                                                                                                                                                                        |
| ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| /      | Nejvyšší adresář (root filesystem). Obsahuje všechny soubory nutné pro start operačního systému ještě před připojením ostatních filesystémů. Po naběhnutí systému jsou ostatní filesystémy připojeny jako podadresáře rootu. |
| /bin   | Obsahuje základní a nezbytné příkazy (binárky), které jsou potřeba pro běžný chod systému.                                                                                                                                   |
| /boot  | Obsahuje statický bootloader, jádro systému (kernel) a další soubory nutné pro zavedení Linuxu.                                                                                                                              |
| /dev   | Obsahuje soubory zařízení, které umožňují přístup ke všem hardwarovým zařízením připojeným k systému.                                                                                                                        |
| /etc   | Lokální konfigurační soubory systému. Jsou zde uloženy také konfigurace nainstalovaných aplikací.                                                                                                                            |
| /home  | Každý uživatel systému zde má vlastní podadresář pro ukládání osobních dat.                                                                                                                                                  |
| /lib   | Sdílené knihovny, které jsou nezbytné pro start systému a běh základních příkazů.                                                                                                                                            |
| /media | Sem se připojují externí vyměnitelná zařízení, například USB disky nebo CD/DVD.                                                                                                                                              |
| /mnt   | Dočasný připojovací bod pro běžné filesystémy.                                                                                                                                                                               |
| /opt   | Volitelné soubory, typicky nástroje třetích stran nebo externí aplikace.                                                                                                                                                     |
| /root  | Domovský adresář uživatele root (administrátora).                                                                                                                                                                            |
| /sbin  | Obsahuje spustitelné soubory určené pro správu systému (systémové binárky).                                                                                                                                                  |
| /tmp   | Systém a programy zde ukládají dočasné soubory. Obsah je většinou mazán při restartu a může být smazán kdykoliv bez varování.                                                                                                |
| /usr   | Obsahuje uživatelské programy, knihovny, manuálové stránky a další sdílené zdroje.                                                                                                                                           |
| /var   | Obsahuje proměnlivá data jako logy, e-maily, soubory webových aplikací, cron úlohy a další.                                                                                                                                  |

# 🧠 Jak číst tabulku Linux filesystemu (Pentester pohled)

Tahle tabulka není popis Linuxu pro začátečníky.  
Je to **mentální mapa útoku** – říká ti **kam se dívat jako první a proč**.

---

## 🔥 Hlavní myšlenka
Ne všechny složky mají stejnou hodnotu.
Pentester nehledá „co tam je“, ale **kde je chyba nebo slabé místo**.

---

## 📂 Význam jednotlivých částí

### /etc – nejvyšší priorita
- konfigurační soubory
- špatně nastavené služby
- sudo bez hesla
- cron joby běžící jako root

👉 Často vede k **privilege escalation bez exploitu**

---

### /home – lidský faktor
- hesla v souborech
- `.bash_history`, `.env`, `.config`
- znovupoužitá hesla

👉 Nejslabší článek je **uživatel**, ne systém

---

### /tmp – pracovní prostor útočníka
- obvykle writable pro každého
- místo pro:
  - reverse shell
  - exploit
  - dočasné skripty

👉 Když je `/tmp` omezené, hledá se `/dev/shm`

---

### /root – cíl
- nedíváš se tam jako první
- dostaneš se tam **až přes chybu jinde**

👉 Přístup do `/root` = **plná kontrola systému**

---

### /var/log – stopy
- zaznamenává přihlášení, chyby, příkazy
- modrý tým: analýza útoku
- červený tým: zametání stop

👉 Každá akce může být logovaná

---

## 🧭 Skrytá logika tabulky
Pořadí složek odpovídá reálnému postupu útoku:

1. Zorientuj se (`/`)
2. Najdi konfiguraci (`/etc`)
3. Najdi uživatele (`/home`)
4. Najdi místo pro práci (`/tmp`)
5. Eskaluj na root (`/root`)
6. Zkontroluj logy (`/var/log`)

---

## ✅ Shrnutí
Tabulka slouží jako:
- checklist při enumerační fázi
- mentální mapa při CTF
- rychlá orientace při pentestu

Není to reference.  
Je to **návod, jak přemýšlí útočník**.

### 📂 Klíčové soubory k zapamatování

Jako hacker se na tyhle soubory dívej jako na první cíle:

1. **`/etc/passwd`**: Seznam všech uživatelů. Je čitelný pro všechny (**r--**). Uvidíš tam jména, UID a domovské složky.
    
2. **`/etc/shadow`**: Obsahuje **hashe hesel**. Je čitelný jen pro roota. Pokud ho dokážeš přečíst bez sudo, systém je špatně nastavený a ty můžeš hesla zkusit "cracknout" (např. pomocí John the Ripper).


### 📂 Hlavní logovací soubory

Většina logů se nachází v adresáři `/var/log/`. Zde jsou ty nejdůležitější:

|**Typ logu**|**Umístění**|**Co v něm hledat?**|
|---|---|---|
|**Autentizace**|`/var/log/auth.log`|Úspěšná i neúspěšná přihlášení, použití `sudo`.|
|**Systém**|`/var/log/syslog`|Obecné zprávy systému a služeb, které nemají vlastní log.|
|**Kernel**|`/var/log/kern.log`|Hlášení jádra, chyby hardwaru, zásahy firewallu.|
|**Apache Web**|`/var/log/apache2/access.log`|Kdo přistupoval na web, jaké URL volal a s jakým výsledkem.|
|**MySQL**|`/var/log/mysql/error.log`|Chyby databáze (někdy obsahují úryvky SQL dotazů).|