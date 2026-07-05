# 📑 18. ENUMERACE A ZÍSKÁVÁNÍ POLITIK HESEL

## 🔐 1. Získání politiky s platnými údaji (Credentialed)

Pokud už máš k dispozici alespoň jeden ověřený účet (např. z předchozího kroku), zjištění politiky hesel je triviální záležitost.

### Z Linuxu přes CrackMapExec (CME) / NetExec

Nástroj CrackMapExec se dotáže doménového kontroleru přes protokol SMB a vytáhne kompletní konfiguraci:

Bash

```
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```

### Z Windows přes vestavěné příkazy (net.exe)

Pokud získáš přístup k Windows stanici v doméně, nemusíš stahovat žádné nástroje. Stačí použít vestavěnou binárku:

DOS

```
net accounts
```

## 🕵️‍♂️ 2. Získání politiky bez přihlašovacích údajů (Unauthenticated)

Pokud začínáš zcela bez přístupu, existují dvě starší konfigurační chyby, které ti politiku hesel prozradí: **SMB NULL Session** a **LDAP Anonymous Bind**.

### A) SMB NULL Sessions (Anonymní SMB relace)

Tato zranitelnost vzniká nejčastěji při in-place upgradech starých doménových kontrolerů, kdy se zachová zpětná kompatibilita s historickým (a nebezpečným) nastavením Windows Serveru. Umožňuje útočníkovi připojit se k sdílenému prostředku `IPC$` s prázdným jménem i heslem.

#### Dotaz přes rpcclient:

Bash

```
rpcclient -U "" -N 172.16.5.5
rpcclient $> querydominfo       # Ověření NULL session a zjištění počtu uživatelů
rpcclient $> getdompwinfo       # Výpis minimální délky hesla a komplexnosti
```

#### Komplexní enumerace přes enum4linux-ng:

Moderní Python alternativa k původnímu Perlovému skriptu dokáže automatizovat rpcclient dotazy a uložit čistý JSON/YAML výstup.

Bash

```
enum4linux-ng -P 172.16.5.5 -oA ilfreight
```

### B) LDAP Anonymous Bind (Anonymní LDAP dotaz)

Od Windows Server 2003 je anonymní dotazování do LDAPu defaultně zakázáno. Správci ho však občas manuálně povolují kvůli integraci starších aplikací třetích stran, čímž nechtěně vystaví celou AD strukturu komukoliv v síti.

K vytažení politiky hesel přes anonymní LDAP dotaz lze použít klasický `ldapsearch`:

Bash

```
ldapsearch -H ldap://172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -i -E "pwdHistoryLength|lockoutThreshold|minPwdLength"
```

## 📊 3. Analýza politiky domény INLANEFREIGHT.LOCAL

Získaná data z výše uvedených skenů nám poskytují detailní pohled na obranu cíle:

|**Parametr v politice**|**Zjištěná hodnota**|**Co to znamená pro útočníka? 🤔**|
|---|---|---|
|**Minimum password length**|8 znaků|Velmi slabé. Uživatelé budou používat jednoduchá hesla typu `Welcome1`.|
|**Account Lockout Threshold**|**5 pokusů**|**Zásadní hodnota!** Máš maximálně 4 neúspěšné pokusky. Aby byla zachována stoprocentní bezpečnost, při Password Sprayingu smíš zkusit **pouze 1 až 2 hesla** na jedno kolo.|
|**Lockout Duration**|30 minut|Pokud účet omylem uzamkneš, za půl hodiny se automaticky odemkne.|
|**Reset Lockout Counter**|30 minut|Čítač špatných pokusů se vynuluje po 30 minutách.|
|**Password Complexity**|Enabled (1)|Heslo musí obsahovat kombinaci velkých/malých písmen, čísel nebo znaků (např. `Klmcargo2` splňuje).|
|**Maximum password age**|Unlimited / Not Set|Hesla uživatelům nikdy neexpirují. Nemají motivaci je měnit, takže staré úniky dat (Breach Data) budou vysoce funkční.|

## 🏁 Strategie pro nadcházející Password Spraying

Na základě této analýzy sestavíme bezpečný plán útoku:

1. Vygenerujeme čistý seznam uživatelských jmen (Target List).
    
2. Vybereme **jedno jediné** vysoce pravděpodobné heslo (např. `Sezona2026!` nebo `Inlanefreight1`).
    
3. Proženeme toto jedno heslo napříč všemi uživateli (tzv. horizontální útok).
    
4. **Další pokus s jiným heslem spustíme nejdříve za 31 minut**, abychom bezpečně obešli limit `Account Lockout Threshold` a čítač na doménovém kontroleru se mezitím vynuloval.