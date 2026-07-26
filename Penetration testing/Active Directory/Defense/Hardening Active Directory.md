# 📖 Hardening Active Directory & Defensivní Ochrana

Cílem správného zabezpečení (hardenu) Active Directory je omezit rádius dopadu při kompromitaci, zabránit lateral movementu, eskalaci práv a neautorizovanému přístupu k citlivým datům. Místo spoléhání se výhradně na EDR/SIEM nástroje je klíčové vybudovat silný bezpečnostní základ (baseline).

## 🔍 1. Audit a dokumentace (Step One)

Před samotným nastavováním restrikcí je nutné mít detailní přehled o celém AD prostředí. Audit by měl probíhat minimálně jednou ročně (ideálně každých několik měsíců).

### Co je nutné evidovat a sledovat:

- **Konvence pojmenování:** Struktura OU, počítačů, uživatelů a skupin.
    
- **Síťová infrastruktura:** Konfigurace DNS, DHCP a IP rozsahů.
    
- **Skupinové politiky (GPO):** Detailní přehled všech GPO a objektů, na které jsou aplikovány.
    
- **FSMO Role:** Přesné přiřazení FSMO rolí konkrétním doménovým řadičům.
    
- **Inventář aplikací:** Kompletní přehled schválených a běžících aplikací.
    
- **Inventář zařízení:** Seznam všech podnikových strojů a jejich fyzické/logické umístění.
    
- **Doménové vztahy (Trusts):** Všechny interní i externí doménové důvěry.
    
- **Privilegovaní uživatelé:** Přesná evidence účtů s vyššími oprávněními.
    

## 🛡️ 2. Koncept PPT: People, Processes, Technology

### A. People (Lidský faktor)

Uživatelé a administrátoři zůstávají nejčastějším vstupním bodem útočníků.

- **Heslová politika:** Zavedení komplexních hesel s filtrem zakazujícím běžná slova (sezóny, jméno firmy, "Heslo123"). Používání podnikových správců hesel.
    
- **Správa servisních účtů:** Pravidelná rotace hesel pro všechny servisní účty.
    
- **Lokální administrátoři:** Zrušení práv lokálního admina na běžných stanicích.
    
- **RID-500 & LAPS:** Deaktivace výchozího lokálního admin účtu (RID-500). Vytvoření nového účtu spravovaného pomocí **LAPS**.
    
- **Tiered Administration Model:** Oddělení administrativních účtů (PAW / Jump hosts). Správci nesmí pracovat na běžných stanicích pod účtem Domain Admin.
    
- **Čištění skupin:** Redukce počtu členů ve skupinách `Domain Admins` a `Enterprise Admins` na absolutní minimum.
    
- **Kerberos Delegation:** Zakázání delegování u administrativních účtů.
    

### 🛡️ Hloubkový vhled: Protected Users Group

Skupina **Protected Users** (uvedená ve Windows Server 2012 R2) poskytuje zvýšenou ochranu pro privilegované účty na úrovni doménového řadiče i klienta.

```
Get-ADGroup -Identity "Protected Users" -Properties Name,Description,Members
```

#### Ochranné mechanismy skupiny Protected Users:

1. **Zákaz Kerberos Delegation:** Členové nemohou být delegováni (Constrained ani Unconstrained).
    
2. **CredSSP & Windows Digest:** Neukládají plaintextová hesla do paměti LSASS (ani při explicitním povolení v GPO).
    
3. **Restrikce autentizace:** Členové se **nemohou autentizovat přes NTLM**, DES ani RC4 šifrování.
    
4. **Kešování klíčů:** Dlouhodobé klíče a plaintext kredenční údaje se neukládají do paměti po získání TGT.
    
5. **Omezení TTL pro TGT:** Životnost TGT ticketu nelze obnovovat nad rámec počátečních **4 hodin**.
    

> ⚠️ **Upozornění:** Přidání uživatelů do `Protected Users` bez předchozího testování může způsobit blokaci účtů nebo selhání autentizace starších služeb!

### B. Processes (Procesy a Politiky)

- **Asset Management:** Pravidelné audity AD strojů, tagování zařízení a proces vyřazování.
    
- **Životní cyklus účtů:** Jasně definovaný Onboarding / Offboarding (okamžité mazání či správná deaktivace účtů bývalých zaměstnanců).
    
- **Decommissioning:** Bezpečné odstraňování zastaralých systémů a služeb (např. správná odinstalace Exchange při přechodu na M365 místo pouhého vypnutí VM).
    
- **Pravidelné audity:** Plánované audity uživatelů, skupin a práv.
    

### C. Technology (Technologická opatření)

- **Pravidelné skenování miskonfigurací:** Používání nástrojů `BloodHound`, `PingCastle` a `Grouper`.
    
- **Čištění textových dat:** Pravidelná kontrola atributu `Description` u AD objektů a skriptů v `SYSVOL` na přítomnost hesel.
    
- **gMSA / MSA:** Nahrazení klasických servisních účtů pomocí **Group Managed Service Accounts (gMSA)** k eliminaci Kerberoastingu.
    
- **Kvantita počítačových účtů (`ms-DS-MachineAccountQuota`):** Nastavení hodnoty atributu na **`0`** pro zamezení útokům typu `noPac` nebo `RBCD`.
    
- **Print Spooler:** Vypnutí služby `Spooler` na doménových řadičích (prevence `PrintNightmare`, `Printer Bug`).
    
- **Zabezpečení LDAP a SMB:** Vynucení SMB Signing a LDAP Signing/Channel Binding.
    
- **Omezení Null Sessions:** Nastavení registru `RestrictNullSessAccess` na `1` k blokování anonymního výčtu.
    
- **Restrikce NTLM:** Postupné zakazování NTLM ve prospěch Kerberos autentizace.
    

## 🎯 3. Mapping na MITRE ATT&CK & Defensivní Kontroly

| Útok / TTP                     | MITRE Tag   | Doporučená defensivní opatření                                                                                              |
| ------------------------------ | ----------- | --------------------------------------------------------------------------------------------------------------------------- |
| **External Reconnaissance**    | `T1589`     | Sanitizace veřejných dokumentů (odstranění metadat), úprava inzerátů práce (neuvádět přesný SW/HW stack), kontrola BGP/DNS. |
| **Internal Reconnaissance**    | `T1595`     | Detekce port skenování přes NIDS/SIEM, restrikce ICMP v Windows Firewall/EDR, síťová segmentace.                            |
| **Poisoning (LLMNR/NBT-NS)**   | `T1557`     | Vynucení SMB Message Signing, zakázání LLMNR/NBT-NS přes GPO, šifrování síťového provozu.                                   |
| **Password Spraying**          | `T1110.003` | Sledování Event ID `4624` a `4648`, zavedení lockout politik, MFA, využití password filtrů.                                 |
| **Credentialed Enumeration**   | `TA0006`    | Sledování anomálního chování CLI/PowerShellu, detekce masivního RDP/SMB provozu mezi stanicemi, segmentace.                 |
| **Living off the Land (LOTL)** | `N/A`       | Nasazení AppLocker / WDAC politik, baselining běžného chování uživatelů a procesů.                                          |
| **Kerberoasting**              | `T1558.003` | Vynucení AES256 šifrování (zakázat RC4), nasazení gMSA účtů, silná hesla pro SPN účty, audit skupin.                        |