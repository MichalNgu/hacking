
Linux je **monolitické jádro**, což znamená, že celé běží v jednom privilegovaném paměťovém prostoru. Je ale **modulární** – ovladače můžeš nahrávat za běhu (LKM - Loadable Kernel Modules), aniž bys musel systém restartovat.

---

### 1. Architektura: User Space vs. Kernel Space

Linux striktně odděluje, kdo co může dělat pomocí tzv. **Protection Rings**:

- **User Space (Ring 3):** Tady běží tvoje aplikace (prohlížeč, terminál). Mají omezený přístup k HW. Pokud aplikace spadne, nezhroutí se celý systém.
    
- **Kernel Space (Ring 0):** Tady má jádro absolutní moc nad CPU a pamětí.
    

Most mezi nimi: System Calls (Syscalls)

Když chce program v User Space něco udělat (třeba otevřít soubor), nemůže to udělat přímo. Musí zavolat „systémové volání“ (např. open(), read(), fork()). CPU se v tu chvíli přepne do privilegovaného režimu, jádro požadavek provede a vrátí výsledek zpět.

---

### 2. Správa procesů (Process Management)

V Linuxu je proces v podstatě struktura v paměti zvaná `task_struct`.

- **Plánovač (Scheduler):** Linux používá **CFS (Completely Fair Scheduler)**. Ten se snaží spravedlivě rozdělit čas procesoru mezi všechny procesy pomocí „červeno-černého stromu“ (datová struktura), aby systém působil plynule.
    
- **Fork & Exec:** Nové procesy nevznikají z ničeho. Existující proces se „rozdvojí“ (`fork()`) a pak se jeho obsah nahradí novým programem (`exec()`).
    

### 3. Správa paměti (Memory Management)

Linux používá **Virtuální paměť**. Žádný program nevidí skutečnou fyzickou RAM.

- Každý proces si „myslí“, že má pro sebe celých $2^{48}$ (u 64-bit) adresního prostoru.
    
- **MMU (Memory Management Unit):** Hardwarová součástka, která za běhu překládá virtuální adresy na ty skutečné v RAM.
    
- **Paging:** Paměť je rozdělena na malé bloky (stránky, typicky 4 KB). Pokud dojde RAM, jádro odloží nepoužívané stránky na disk (**Swap**).
    

---

### 4. VFS – Virtuální souborový systém

Jedna z nejúžasnějších věcí na Linuxu je filozofie: „Všechno je soubor.“

Aby jádro nemuselo řešit, jestli čteš z disku (ext4), USB (FAT32) nebo ze sítě (NFS), používá vrstvu VFS. Ta sjednocuje ovládání.

- **Speciální soubory:** V `/dev/` najdeš hardware (např. `/dev/sda` je tvůj disk).
    
- **Pseudo-souborové systémy:** `/proc/` a `/sys/` nejsou na disku. Jsou to „okna“ přímo do vnitřností běžícího kernelu. Když napíšeš `cat /proc/cpuinfo`, kernel ti ty informace vygeneruje za běhu.
    

---

### 5. Přerušení a ovladače (Interrupts)

Hardware komunikuje s jádrem pomocí **přerušení (interrupts)**. Když stiskneš klávesu, klávesnice pošle elektrický signál CPU. CPU okamžitě zastaví to, co dělá, a spustí **ISR (Interrupt Service Routine)** v kernelu, která stisk zpracuje.

### Jak to vidět v praxi (pro tvůj CPTS mindset):

Pokud chceš vidět, jak se aplikace „baví“ s kernelem, zkus tyto nástroje:

1. `strace <příkaz>`: Zobrazí všechna systémová volání (syscalls), která program provádí.
    
2. `ltrace <příkaz>`: Zobrazí volání knihoven.
    
3. `lsmod`: Ukáže ti seznam aktuálně nahraných modulů (LKM) v jádře.


# 🔑 Správa práv (Permission Management)

V Linuxu má každý soubor a adresář jasně definováno, kdo s ním může co dělat. Tato práva se dělí do tří kategorií uživatelů:

1. **Owner (u)**: Vlastník souboru (zpravidla ten, kdo ho vytvořil).
    
2. **Group (g)**: Skupina uživatelů, která má k souboru přístup.
    
3. **Others (o)**: Všichni ostatní uživatelé v systému.
    


### 🏗️ Anatomie výpisu `ls -l`

Když napíšeš `ls -l`, uvidíš řetězec jako `-rwxr-xr--`. Co to znamená?

|**Pozice**|**Význam**|
|---|---|
|**1. znak**|Typ: `-` (soubor), `d` (adresář), `l` (odkaz)|
|**2.-4. znak**|Práva **Vlastníka** (rwx)|
|**5.-7. znak**|Práva **Skupiny** (r-x)|
|**8.-10. znak**|Práva **Ostatních** (r--)|

### 🔢 Oktálová (číselná) notace

Práva se často vyjadřují čísly. Každé právo má svou hodnotu:

- **Read (r) = 4**
    
- **Write (w) = 2**
    
- **Execute (x) = 1**
    

Sečtením těchto čísel získáš výsledné právo pro jednu kategorii:

- `7` (4+2+1) = **rwx** (všechna práva)
    
- `5` (4+0+1) = **r-x** (čtení a spouštění)
    
- `4` (4+0+0) = **r--** (pouze čtení)
    

**Příklad:** `chmod 754 soubor` znamená: Vlastník (7 - rwx), Skupina (5 - r-x), Ostatní (4 - r--).

### 🚀 Speciální práva: SUID, SGID a Sticky Bit

Tady začíná ta pravá hackerská zábava.

#### 1. SUID (Set User ID)

- **Značení:** `s` místo `x` u vlastníka (např. `-rwsr-xr-x`).
    
- **Funkce:** Program se spustí s právy **vlastníka** souboru, ne uživatele, který ho spustil.
    
- **Pentest:** Pokud má program jako `nmap` nebo `python` nastavený SUID bit a vlastní ho `root`, můžeš se přes ně stát rootem.
    

#### 2. SGID (Set Group ID)

- **Značení:** `s` místo `x` u skupiny.
    
- **Funkce:** Soubor běží s právy skupiny souboru. U adresářů zajistí, že nové soubory uvnitř budou patřit stejné skupině.
    

#### 3. Sticky Bit

- **Značení:** `t` na konci (např. `drwxrwxrwt`).
    
- **Funkce:** I když mají všichni právo zápisu do složky (např. `/tmp`), soubory může mazat **pouze jejich majitel**.
    
- **Velké T vs. malé t:**
    
    - `t` (malé): Ostatní mají právo `x` (mohou do složky vstoupit).
        
    - `T` (velké): Ostatní **nemají** právo `x` (do složky nemohou).


# 👤 User Management (Správa uživatelů)

V Linuxu je uživatel definován svým **UID** (User ID). Root má vždy `UID 0`. Ostatní uživatelé začínají (obvykle) od `UID 1000` výše.

### 1. Přepínání uživatelů (`sudo` vs `su`)

To jsou dva hlavní způsoby, jak změnit identitu v terminálu:

- **`sudo` (SuperUser Do):** Spustíš jeden konkrétní příkaz s právy root (nebo jiného uživatele). Systém se tě zeptá na **TVOJE** heslo.
    
    - _Příklad:_ `sudo cat /etc/shadow`
        
- **`su` (Substitute User):** Přepne tě úplně do jiného uživatelského profilu. Systém se tě zeptá na **HESLO TOHO UŽIVATELE**, na kterého se přepínáš.
    
    - _Příklad:_ `su - mrb3n` (přepne tě k uživateli mrb3n a načte jeho prostředí).

### 📂 Kde Linux bere software? (Sources)

Všechny adresy, odkud si tvůj systém stahuje programy, jsou zapsány v souboru: `/etc/apt/sources.list`

Pokud ti `apt install` hlásí, že balíček neexistuje, pravděpodobně ti v tomto souboru chybí správný repozitář.

---

### 🛡️ SSH (Secure Shell)

SSH je dnes standardem pro bezpečnou vzdálenou správu. Nahradilo starý Telnet, protože veškerou komunikaci **šifruje**.

- **Instalace:** `sudo apt install openssh-server`
    
- **Konfigurace:** `/etc/ssh/sshd_config` (zde můžeš například zakázat přihlášení uživatele root, což je základní bezpečnostní pravidlo).
    
- **Pentest tip:** Vždy kontroluj verzi SSH. Starší verze mohou mít zranitelnosti, nebo mohou umožňovat hádání uživatelských jmen (username enumeration).
    

---

### 📂 NFS (Network File System)

NFS ti dovolí připojit si složku ze vzdáleného serveru tak, jako by byla na tvém vlastním disku.

- **Konfigurace:** Soubor `/etc/exports` definuje, kdo a s jakými právy může ke složkám přistupovat.
    
- **Klíčové nastavení `no_root_squash`:** Pokud uvidíš toto nastavení v `/etc/exports`, zbystři! Znamená to, že pokud se k NFS připojíš jako root ze svého stroje, budeš mít **root práva i na serveru**. To je jedna z nejčastějších cest k ovládnutí celého serveru (Privilege Escalation).
    

---

### 🌐 Webové servery (Apache & Python)

Webové servery nejsou jen pro webové stránky. Pro pentestera jsou to nástroje na **přenos souborů**.

- **Apache (`apache2`):** Velký, robustní server. Složka `/var/www/html` je místo, kam dáváš soubory, které chceš sdílet.
    
- **Python HTTP Server:** Nejrychlejší cesta, jak něco nasdílet. Stačí jeden příkaz v adresáři, kde máš soubory: `python3 -m http.server 80` Tím okamžitě vytvoříš webový server na portu 80. Skvělé pro stahování exploitů do cílového systému pomocí `wget` nebo `curl`.
    

---

### 🔒 VPN (OpenVPN)

VPN vytváří bezpečný tunel skrze nezabezpečený internet. Pro tebe je to brána do vnitřní sítě klienta (např. v HTB nebo při reálném testu).

- **Soubor `.ovpn`:** Obsahuje veškeré certifikáty a nastavení pro připojení.
    
- **Spuštění:** `sudo openvpn jmeno_souboru.ovpn`
    

---

### ⚠️ Bezpečnostní varování: FTP vs. SFTP

V textu jsi četl o FTP. Pamatuj, že **FTP posílá hesla v čistém textu**. Kdokoliv na stejné síti (např. v kavárně nebo na kompromitovaném switchi) může tvoje heslo vidět pomocí nástrojů jako Wireshark. Vždy upřednostňuj **SFTP** (běží přes SSH).



### 🚀 Rsync: Král synchronizace

Rsync je extrémně rychlý, protože přenáší pouze **změněné části** souborů (tzv. delta transfer).

**Klíčové přepínače:**

- **`-a` (archive):** Nejdůležitější přepínač. Zachová práva, vlastníky a časová razítka.
    
- **`-v` (verbose):** Ukazuje ti, co se právě děje.
    
- **`-z` (compress):** Komprimuje data během přenosu (šetří šířku pásma).
    
- **`-e ssh`:** Zajistí, že se data nepřenášejí čitelně, ale skrz zašifrovaný SSH tunel.
    

---

### 🔐 Bezpečnost a šifrování

Samotná záloha nestačí, musí být i soukromá.

- **Duplicity:** Nástroj, který kombinuje sílu Rsyncu s šifrováním (GPG). Pokud ti někdo ukradne zálohu z cloudu, bez hesla/klíče ji nepřečte.
    
- **SSH klíče:** Aby automatizace (přes Cron) fungovala, nemůže se tě skript pokaždé ptát na heslo. Proto se používá `ssh-keygen` a `ssh-copy-id`, které vytvoří "důvěru" mezi stroji.
    

---

### 🤖 Automatizace: Spojení Rsync + Cron

Toto je nejčastější způsob, jakým administrátoři (i útočníci pro exfiltraci dat) udržují data aktuální na jiném stroji.

**Postup v kostce:**

1. Vytvoříš SSH klíče bez hesla (**Passphrase: None**).
    
2. Zkopíruješ veřejný klíč na server.
    
3. Napíšeš jednoduchý skript `.sh` s příkazem `rsync`.
    
4. Přidáš skript do `crontab -e`.


### 🌐 Správa síťových rozhraní (Interfaces)

V Linuxu už nehledáme „ovládací panely“. Vše ovládáme příkazy, které nám dají mnohem přesnější kontrolu.

- **`ip addr` (nebo starší `ifconfig`)**: Tvůj první pohled na síť. Zjistíš zde svou IP adresu a MAC adresu.
    
- **Aktivace rozhraní**: Pokud tvoje síťovka „spí“, probudíš ji příkazem `sudo ip link set eth0 up`.
    
- **Statická konfigurace**: Soubor `/etc/network/interfaces` je místo, kde definuješ, že tvůj stroj bude mít vždy stejnou adresu (např. v labu).
    

---

### 🛡️ Network Access Control (NAC)

NAC rozhoduje o tom, kdo může do sítě vstoupit a co v ní smí dělat. Jako pentester narazíš na tři hlavní modely:

|**Model**|**Princip**|**Analogie**|
|---|---|---|
|**DAC** (Discretionary)|Vlastník souboru určuje práva.|Majitel bytu ti půjčí klíče.|
|**MAC** (Mandatory)|Systém (kernel) vynucuje pravidla.|Bankovní trezor – ani ředitel ho neotevře bez splnění protokolu.|
|**RBAC** (Role-Based)|Práva máš podle své funkce (Role).|Sestra v nemocnici vidí karty pacientů, ale účetní ne.|

---

### 🧱 Hardening: SELinux vs. AppArmor

Toto jsou „vyhazovači“ v Linuxovém jádře. Pokud se ti podaří nabourat do aplikace (např. webového serveru), tito strážci zabrání tomu, aby útočník ovládl celý zbytek systému.

- **SELinux (RedHat/CentOS)**: Extrémně silný, ale složitý. Kontroluje každý proces a každý soubor pomocí bezpečnostních štítků.
    
- **AppArmor (Ubuntu/Debian)**: Jednodušší na správu. Používá profily pro konkrétní aplikace. Pokud aplikace dělá něco, co nemá v profilu, AppArmor ji zastaví.
    
- **TCP Wrappers**: Jednoduchý filtr (soubory `/etc/hosts.allow` a `/etc/hosts.deny`), který pouští na služby jen určité IP adresy.
    

---

### 🔍 Diagnostika a Troubleshooting (Tvé zbraně)

Když něco nefunguje (nebo když hledáš cestu k cíli), použiješ tyto nástroje:

- **`ping`**: „Jsi tam?“ – Základní test dostupnosti.
    
- **`traceroute`**: „Kudy tam jdou data?“ – Ukáže ti všechny routery na cestě k cíli.
    
- **`netstat -tulpn`**: „Kdo u mě poslouchá?“ – Zobrazí všechny otevřené porty a služby, které na nich běží. To je pro pentestera zlatý důl.
    

---

### 🧩 DNS: Srdce internetu

Bez DNS by ses nikam nepřipojil přes jméno (např. `google.com`).

- **Soubor `/etc/resolv.conf`**: Zde Linux hledá adresy DNS serverů.
    
- **Pentest tip**: Vždy kontroluj, zda systém nepoužívá podezřelý DNS server. Útočník může pomocí falešného DNS přesměrovat tvůj provoz na svůj server (DNS Poisoning).


## RD (Remote Desktop)
### 🖥️ X11: Síťová transparentnost

Protokol X11 (nebo jednoduše X) je základním stavebním kamenem grafiky v Unixových systémech. Jeho největší výhodou je, že aplikace může běžet na serveru, ale její **okno se vykresluje u tebe na lokálním stroji**.

- **Porty:** Typicky **TCP 6000** (pro displej :0), 6001 atd.
    
- **Bezpečnost:** X11 je nativně **nešifrovaný**. Útočník na stejné síti může pomocí nástrojů jako `xwd` doslova „sledovat tvou obrazovku“ nebo snímat stisky kláves.
    
- **Řešení:** Vždy používej **SSH Tunneling** (`ssh -X`). Tím se grafický provoz zabalí do šifrovaného SSH spojení.
    

---

### 🔗 VNC (Virtual Network Computing)

VNC je v Linuxu nejpoužívanější metodou pro sdílení celé pracovní plochy (podobně jako TeamViewer).

- **Porty:** Začíná na **TCP 5900** (displej :0), pro displej :1 je to port **5901** atd.
    
- **Koncepty:**
    
    1. **Sdílení fyzické obrazovky:** Vidíš to samé, co člověk sedící u počítače.
        
    2. **Virtuální relace:** Vytvoříš si novou plochu „na pozadí“, která není vidět na monitoru serveru.
        

---

### 🛡️ Bezpečnostní rizika a Pentesting

Jako pentester se při skenování sítě (např. pomocí `nmap`) zaměřuješ na následující slabiny:

1. **Otevřené X11 (6000-6005):** Pokud najdeš otevřený port 6000 bez autentizace, můžeš se pokusit o screenshot vzdálené plochy nebo do ní posílat příkazy.
    
2. **Slabá hesla VNC:** VNC často používá pouze jedno heslo pro přístup (bez uživatelského jména). To je ideální cíl pro slovníkový útok (brute-force).
    
3. **XDMCP (UDP 177):** Starý, nešifrovaný protokol pro správu relací. Velmi náchylný k útoku Man-in-the-Middle (MITM).
    

---

### 🛠️ Praktické nastavení (TigerVNC)

Nastavení VNC v Linuxu vyžaduje několik kroků, protože musíš systému říct, které grafické prostředí (např. XFCE) má po připojení spustit:

1. **Instalace:** `sudo apt install tigervnc-standalone-server xfce4`
    
2. **Heslo:** `vncpasswd` (vytvoří soubor v `~/.vnc/passwd`).
    
3. **Spuštění:** `vncserver :1` (spustí server na portu 5901).
    
4. **Zabezpečení:** Protože VNC samo o sobě není vždy ideálně šifrované, doporučuje se vytvořit tunel: `ssh -L 5901:127.0.0.1:5901 user@server` Poté se ve svém prohlížeči (vieweru) připojuješ k `localhost:5901`.