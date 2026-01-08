
Windows je postaven na **objektově orientovaném přístupu** a používá tzv. **hybridní jádro (NT Kernel)**.

Tady je hloubkový pohled do architektury Windows, který tě jako pentestera bude zajímat:

---

### 1. User Mode vs. Kernel Mode (Hybridní jádro)

Windows dělí systém na dvě hlavní vrstvy, ale s jedním mezikusem navíc:

- **User Mode:** Tady běží tvé aplikace a **Subsystemy** (jako Win32, přes který běží většina programů, nebo WSL pro Linux).
    
- **Kernel Mode:** Zde běží samotné jádro (`ntoskrnl.exe`), ovladače a **HAL (Hardware Abstraction Layer)**.
    
- **HAL:** To je vrstva, která „schovává“ rozdíly v hardwaru. Jádro nekomunikuje přímo s procesorem, ale s HALem, což zajišťuje, že Windows běží na různých typech základních desek.
    

---

### 2. Všechno je OBJEKT (ne soubor)

V Linuxu čteš soubor v `/dev/`. Ve Windows komunikuješ s **objekty**.

- Vše (procesy, vlákna, soubory, klíče v registru) spravuje **Object Manager**.
    
- Když program chce k něčemu přistoupit, dostane **Handle** (ukazatel/token).
    
- **CPTS Souvislost:** Právě manipulace s "Handles" nebo "Tokeny" je základem pro **Privilege Escalation** (např. Token Impersonation).
    

### 3. Registr: Mozek systému

Místo tisíců konfiguračních souborů v `/etc/` (Linux) mají Windows jednu obří hierarchickou databázi – **Windows Registry**.

- Je to extrémně efektivní pro systém (rychlé čtení), ale nebezpečné pro stabilitu (pokud se poškodí, systém nenabootuje).
    
- Pro tebe jako lovce bugů jsou registry zlatý důl – hledáš tam špatně nastavená oprávnění, uložená hesla nebo cesty k binárkám, které můžeš přepsat.
    

---

### 4. Správa procesů a Vláken

Windows je extrémně zaměřený na **vlákna (threads)**.

- **Proces** ve Windows „nic nedělá“, je to jen kontejner (má svou paměť, tokeny a zdroje).
    
- **Vlákno** je to, co reálně vykonává kód.
    
- **EPROCESS:** V kernelu má každý proces svou strukturu `EPROCESS`. Pokud ji jako útočník dokážeš v paměti přepsat (např. změnit své PID na 4, což je System), získáš nejvyšší práva.
    

---

### 5. I/O a IRP (I/O Request Packets)

Když chceš ve Windows něco zapsat na disk, nepoužívá se jednoduchý stream jako v Linuxu. Vytvoří se datová struktura zvaná **IRP**.

- Tento paket cestuje dolů skrze „stoh“ ovladačů (Driver Stack).
    
- Každý ovladač si z paketu může něco vzít nebo ho změnit. Toho využívají **Rootkity**, které se do tohoto stohu vloží a filtrují pakety tak, aby schovaly soubory nebo procesy před antivirem.
    

---

### 6. Bezpečnostní subsystém (LSASS a SRM)

Tohle je pro pentestování nejdůležitější část:

- **SRM (Security Reference Monitor):** Část jádra, která při každém přístupu k objektu kontroluje **ACL (Access Control List)**. Ptá se: „Má tenhle uživatel právo číst tenhle soubor?“
    
- **LSASS (Local Security Authority Subsystem Service):** Proces v User Modu, který spravuje přihlašování a hesla.
    
    - _Fun fact:_ LSASS je terčem č. 1 (dumpování paměti přes Mimikatz), protože v něm v určité formě zůstávají přihlašovací údaje.
        

### Srovnání: Linux vs. Windows pod kapotou

|**Vlastnost**|**Linux**|**Windows**|
|---|---|---|
|**Filozofie**|Všechno je soubor|Všechno je objekt|
|**Konfigurace**|Textové soubory (`/etc`)|Registry (binární databáze)|
|**Volání**|Syscalls (přímo)|Windows API (přes `ntdll.dll`)|
|**Ovladače**|Často součástí jádra|Izolované binárky (.sys)|

### 📊 Task Manager (Správce úloh)

Základní nástroj pro rychlou kontrolu. Pro nás jsou nejdůležitější záložky:

1. **Details:** Tady vidíš **PID** a pod jakým **uživatelem** proces běží. Pokud vidíš proces běžící jako `SYSTEM`, který by neměl (např. webový prohlížeč), je to podezřelé.
    
2. **Startup:** Tady vidíš, co se spouští po startu. Útočníci sem rádi přidávají svůj malware, aby zajistili jeho spuštění po restartu (**Persistence**).

### 🏗️ Architektura WMI (**Windows Management Instrumentation**)

WMI funguje jako prostředník mezi vámi a hardwarem či softwarem. Můžete si ho představit jako obrovskou databázi informací o všem, co se v počítači děje.

- **WMI Repository:** Databáze (uložená v `C:\Windows\System32\wbem\Repository`), kde jsou uložena statická data.
    
- **WMI Providers:** Agenti, kteří monitorují specifické části (např. procesy, ovladače, služby) a posílají data dál.
    
- **Třídy (Classes):** Šablony pro objekty (např. `Win32_Process` reprezentuje všechny běžící procesy).
    
- **Metody (Methods):** Akce, které můžete s třídou provést (např. metoda `Create` u třídy `Win32_Process` spustí nový program).
    

---

### 🛠️ WMIC vs. PowerShell

Tradičně se k WMI přistupovalo přes nástroj **WMIC** (WMI Command-line). Microsoft jej sice označil za „deprecated“ (zastaralý), ale v praxi se s ním stále setkáte, protože je jednoduchý a efektivní.

#### Příklad v WMIC (klasická řádka):

- `wmic process list brief` – Vypíše běžící procesy.
    
- `wmic useraccount get name,sid` – Vypíše uživatele a jejich SID.
    

#### Příklad v PowerShellu (moderní přístup):

PowerShell používá cmdlety `Get-WmiObject` nebo novější `Get-CimInstance`.

- `Get-WmiObject -Class Win32_Service | Select-Object Name, State` – Vypíše služby a jejich stav.

### 🧱 Koncept snap-inů (Zásuvné moduly)

MMC samo o sobě nic nedělá. Jeho síla spočívá v **snap-inech**. To jsou jednotlivé administrátorské nástroje, které do konzole "zasuneš".

**Mezi nejčastější snap-iny patří:**

- **Event Viewer:** Sledování systémových logů.
    
- **Services:** Správa běžících služeb.
    
- **Disk Management:** Formátování a správa oddílů disků.
    
- **Group Policy Editor:** Nastavení bezpečnostních pravidel (pouze ve verzích Pro a Enterprise).
    
- **Certificates:** Správa digitálních certifikátů v systému.
    

---

### 🛠️ Proč je to důležité pro administraci?

1. **Centralizace:** Můžeš si vytvořit soubor `.msc` (např. `MojeSprava.msc`), který po otevření načte všechna důležitá okna najednou.
    
2. **Vzdálená správa:** Při přidávání snap-inu se tě Windows zeptá, zda chceš spravovat **tento počítač** (Local computer), nebo **jiný počítač v síti** (Another computer). To umožňuje spravovat servery z jednoho místa.
    
3. **Delegování:** Můžeš vytvořit ořezanou konzoli s omezenými nástroji a poslat ji kolegovi, který má na starosti jen specifickou část systému.
    

---

### 🕵️ Pohled z hlediska bezpečnosti (Pentesting)

Jako útočník nebo bezpečnostní analytik využíváš MMC k rychlému auditu stroje:

- **Certificates snap-in:** Hledáš, zda v systému nejsou nainstalované podezřelé nebo útočné certifikáty, které by umožnily odposlech HTTPS provozu (MitM).
    
- **Local Users and Groups:** Rychlá kontrola, kdo všechno má práva administrátora.
    
- **Shared Folders:** Okamžitý přehled o tom, co počítač sdílí do sítě (včetně skrytých sdílení).


### 🆔 Security Identifier (SID)

SID je unikátní kód, který Windows přiděluje každému „bezpečnostnímu subjektu“ (uživateli, skupině, počítači). Jména uživatelů se mohou měnit, ale SID zůstává.

- **RID (Relative ID):** Poslední část SID. Například administrátor má vždy RID `500`, zatímco běžní uživatelé začínají na `1000+`.
    
- **Příkaz:** `whoami /user` ti ukáže tvůj SID, což je klíčové při auditu oprávnění v registrech.
    

---

### 🛡️ User Account Control (UAC)

UAC je „přestávka“, která brání malwaru v tichém získání práv správce. Když program vyžaduje vyšší privilegia, UAC vytvoří **Consent Prompt** (výzvu k potvrzení).

- **Admin Approval Mode:** I když jsi administrátor, Windows ti standardně vystaví „ořezaný“ přístupový token. Plný token získáš až po potvrzení UAC výzvy.
    
- **Pentest tip:** Existuje mnoho technik **UAC Bypass**, které umožňují tuto výzvu obejít, pokud je úroveň UAC nastavena na jinou hodnotu než „Always Notify“.
    

---

### 🗃️ Registry (Registry)

Registry jsou hierarchická databáze všeho – od tapety na ploše až po kritické ovladače jádra.

- **Persistence (Udržení se):** Klíče `Run` a `RunOnce` jsou nejoblíbenějším místem malwaru. Cokoliv je zde zapsáno, spustí se automaticky po přihlášení uživatele.
    
- ** hives:** Soubory registrů jsou uloženy v `C:\Windows\System32\config`. Pokud útočník získá soubory `SAM` a `SYSTEM`, může z nich offline vytáhnout hesla všech lokálních uživatelů.
    

---

### 🏛️ Group Policy (GPO) a AppLocker

- **Group Policy (gpedit.msc):** Umožňuje centrálně vynutit bezpečnostní pravidla (např. zakázat USB disky nebo vynutit složitost hesla).
    
- **AppLocker:** Pokročilé **bělení aplikací (Whitelisting)**. Namísto zakazování virů (Blacklisting) administrátor určí, že smí běžet jen programy podepsané Microsoftem nebo ty, které jsou ve složce `Program Files`.
    

---

### 🦠 Windows Defender Antivirus

Dnešní Defender je špičkový produkt. Obsahuje:

- **Tamper Protection:** Brání malwaru v tom, aby antivirus sám vypnul přes Registry nebo PowerShell.
    
- **Controlled Folder Access:** Ochrana proti Ransomware – blokuje neautorizovaným aplikacím měnit soubory v dokumentech.
    
- **Exclusions:** Jako pentester vždy kontroluj seznam výjimek (`Get-MpPreference`). Pokud admin vyloučil složku `C:\Temp`, je to tvé ideální místo pro uložení exploitů.