
## 🗺️ Mapa souborového systému Windows (C:)

Základem je disk **C:** (root). Na rozdíl od Linuxu, kde je vše pod `/`, Windows dělí systém do několika pevně daných bloků:

### 1. Jádro a systém (Možek stroje)

- **`C:\Windows`**: Hlavní složka operačního systému.
    
    - **`System32\`**: Nejdůležitější složka. Obsahuje DLL knihovny, ovladače a spustitelné soubory (jako `cmd.exe`). Pokud jsi 64-bit, jsou tu 64-bitové soubory.
        
    - **`SysWOW64\`**: Tady jsou uloženy 32-bitové soubory pro kompatibilitu (název znamená _Windows on Windows 64_).
        
    - **`Fonts\`**: Systémová písma.
        
    - **`Logs\`**: Systémové záznamy (např. o instalacích).
        

### 2. Uživatelská data (Cíl útoku)

- **`C:\Users`**: Domov všech uživatelských profilů.
    
    - **`Default\`**: Šablona pro nové uživatele.
        
    - **`Public\`**: Sdílené soubory mezi všemi uživateli.
        
    - **`<JménoUživatele>\`**:
        
        - **`Desktop`, `Documents`, `Downloads`**: Klasika, kde lidi nechávají data.
            
        - **`AppData\` (Skrytá)**: Klíčová složka. Obsahuje hesla prohlížečů, lokální databáze aplikací a nastavení. Dělí se na `Local`, `LocalLow` a `Roaming` (data, co se stěhují v doméně).
            

### 3. Aplikace (Kde hledat zranitelnosti)

- **`C:\Program Files`**: Instalace 64-bitových aplikací.
    
- **`C:\Program Files (x86)`**: Instalace 32-bitových aplikací.
    
- **`C:\ProgramData` (Skrytá)**: Sdílená data aplikací. Často zde bývají chybně nastavená práva (vhodné pro _Privilege Escalation_).
    

---

## 🛠️ Porovnávací tabulka pro rychlou orientaci

Pokud přecházíš z Linuxu, tahle tabulka ti pomůže se zorientovat "kde je co":

| **Co hledáš**          | **Linux ekvivalent** | **Windows cesta**                              |
| ---------------------- | -------------------- | ---------------------------------------------- |
| **Kořen**              | `/`                  | `C:\`                                          |
| **Domovská složka**    | `/home/user`         | `C:\Users\<user>`                              |
| **Administrátor**      | `/root`              | `C:\Users\Administrator`                       |
| **Binárky / Programy** | `/bin`, `/usr/bin`   | `C:\Windows\System32`, `C:\Program Files`      |
| **Konfigurace**        | `/etc`               | **Registry** (v souborech v `System32\config`) |
| **Dočasné soubory**    | `/tmp`               | `C:\Windows\Temp` nebo `%TEMP%`                |
| **Logy**               | `/var/log`           | `C:\Windows\System32\winevt\Logs`              |
|                        |                      |                                                |

## 🔍 Pentest tipy: Co v této mapě sledovat?

1. **`C:\pagefile.sys`**: Soubor virtuální paměti. Může obsahovat citlivá data v prostém textu, která zbyla v RAM.
    
2. **`C:\Windows\Panther\`**: Často obsahuje soubor `unattend.xml` s hesly v čitelném textu po automatizované instalaci.
    
3. **`C:\Users\<user>\.ssh\`**: Pokud uživatel používá SSH (např. přes Git), najdeš tu jeho soukromé klíče.


### 🔍 Průzkum z příkazové řádky (CMD)

Když se dostaneš do systému (např. přes reverzní shell), grafické rozhraní nemáš. Musíš si vystačit s příkazy:

- **`dir /a`**: Vypíše vše včetně skrytých souborů. Všimni si ve svém výpisu souborů jako `pagefile.sys` (virtuální paměť) nebo `hiberfil.sys` (data z hibernace). Tyto soubory mohou obsahovat zbytky hesel z RAM!
    
- **`tree /f`**: Graficky znázorní strukturu. Přepínač `/f` vypíše i konkrétní soubory. To je skvělé pro rychlou orientaci v nainstalovaném softwaru.
    
- **`type soubor.txt`**: Ekvivalent linuxového `cat`. Slouží k zobrazení obsahu souboru.
    

---

### ⚙️ Rozdíl mezi 32-bit a 64-bit

Windows používají zajímavý mechanismus pro zpětnou kompatibilitu:

- **`Program Files`**: Zde jsou čistě 64-bitové aplikace.
    
- **`Program Files (x86)`**: Zde jsou starší 32-bitové aplikace.
    
- **`SysWOW64`**: Nenech se zmást názvem – tato složka obsahuje 32-bitové knihovny běžící na 64-bitovém systému (Windows-on-Windows64).

### 📂 Souborové systémy: FAT32 vs. NTFS

- **FAT32:** Starší, kompatibilní se vším (Linux, Mac, foťáky), ale má limit **4 GB na soubor** a **chybí mu zabezpečení**. Neexistují v něm žádná práva – kdo má disk v ruce, ten čte vše.
    
- **NTFS:** Standard od dob Windows NT.
    
    - **Journaling:** Zapisuje si změny, takže po pádu proudu se systém snadno obnoví.
        
    - **Metadata:** Ukládá podrobné info o souborech.
        
    - **Zabezpečení:** Klíčová vlastnost. Umožňuje nastavit, kdo smí co dělat.
        

---

### 🔐 NTFS Oprávnění (Permissions)

V Linuxu máme `rwx`, ve Windows je to mnohem jemnější. Nejdůležitější typy jsou:

- **Full Control (F):** Můžeš všechno, včetně mazání a měnění práv ostatním.
    
- **Modify (M):** Čtení, zápis, mazání.
    
- **Read & Execute (RX):** Můžeš soubor číst a spustit ho (např. `.exe` nebo skript).
    
- **Traverse Folder:** Velmi zajímavé právo. Umožňuje ti "proskočit" složkou, do které nesmíš vidět (nemáš list), abys ses dostal k podadresáři hlouběji, na který práva máš.


### ⚙️ Windows Services (Služby)

Služby běží na pozadí, často pod velmi vysokými privilegii (`SYSTEM`), a startují dříve, než se uživatel vůbec přihlásí.

- **Správa:** Graficky přes `services.msc` nebo v PowerShellu pomocí `Get-Service`.
    
- **Kategorie:**
    
    - **Local System:** Má nejvyšší práva na lokálním stroji.
        
    - **Network Service:** Má práva přistupovat k síťovým prostředkům pod identitou počítače.
        
    - **Local Service:** Omezená práva pro běžné úkoly.
        

#### Kritické systémové procesy

Tyto procesy jsou pilíři Windows. Pokud je ukončíš, systém se zhroutí (BSOD):

- **`lsass.exe`**: Klíčový cíl! Spravuje přihlašování a uchovává v paměti autentizační údaje (hashe/hesla).
    
- **`services.exe`**: „Šéf“ všech služeb.
    
- **`svchost.exe`**: Univerzální hostitel pro služby běžící z DLL knihoven. Často jich uvidíš desítky najednou.