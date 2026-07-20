## 🏗️ Architektura a klíčové procesy

Když se uživatel přihlašuje, rozjede se řetězec událostí, kde figurují tito hlavní hráči:

- WinLogon: Systémový proces, který má na starosti interakci s uživatelem (přihlašovací obrazovka, zamykání PC). Jako jediný odchytává stisky kláves při loginu.
- LogonUI: Grafické rozhraní, které mi zobrazí políčko pro heslo.
- LSASS (Local Security Authority Subsystem Service): Tohle je můj hlavní cíl. Je to strážce brány. Ověřuje uživatele, vynucuje bezpečnostní politiky a posílá logy do Event Logu. Běží jako %SystemRoot%\System32\Lsass.exe.

📦 Autentizační balíčky (DLL v rámci LSASS)

LSASS používá různé knihovny podle toho, o jaký typ přihlášení jde:

- msv1_0.dll: Používá se pro lokální přihlášení (mimo doménu).
- kerberos.dll: Používá se pro doménové přihlášení v Active Directory.
- ntdsa.dll: Tato knihovna běží pouze na Domain Controlleru a spravuje databázi NTDS.dit.

🗄️ Kde jsou uloženy credentials?

Jako útočník musím vědět, kde přesně fyzicky leží data, která chci dumpnout:

### 1. SAM (Security Accounts Manager)

- Umístění: %SystemRoot%\system32\config\SAM (v registrech pod HKLM\SAM).
- Obsah: Lokální uživatelé a jejich NTLM hashe.
- Přístup: Potřebuji práva SYSTEM.

### 2. Credential Manager (Locker)

Windows si sem ukládá hesla pro weby, síťové disky a certifikáty.

- Umístění: C:\Users\[Username]\AppData\Local\Microsoft\Credentials\
- Využití: Pokud ovládnu uživatelský session, můžu odtud vytáhnout hesla k Outlooku nebo prohlížečům (často přes DPAPI).

### 3. NTDS.dit (Svatý grál AD)

Tato databáze existuje pouze na Domain Controllerech.

- Umístění: %SystemRoot%\ntds.dit
- Obsah: Úplně všechno o doméně – všichni uživatelé, skupiny a hlavně všechny doménové hashe.
- Cíl: Pokud získám tento soubor, ovládl jsem celou doménu.

### 🔐 Kerberos vs. NTLM

- NTLM: Starší protokol založený na výzvě a odpovědi (Challenge-Response). LSASS porovnává hash hesla s tím, co je v SAMu.
- Kerberos: Moderní a složitější. Používá systém "lístků" (Tickets). Pokud jsem v doméně, LSASS komunikuje s KDC (Key Distribution Center), což je typicky Domain Controller.