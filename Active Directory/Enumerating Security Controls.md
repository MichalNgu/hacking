# 🛡️ AD Post-Exploitation: Security Controls Checklist

## 🛡️ 1. Antivirus & EDR (Windows Defender)

- [ ] **Ověřit stav Windows Defenderu**
    
    - _Příkaz:_ `Get-MpComputerStatus`
        
    - _Co sledovat:_ Parametr `RealTimeProtectionEnabled`. Pokud je `True`, Defender je aktivní a bude blokovat známé nástroje jako PowerView.
        
    
    - [ ] Poznamenat si verze signatur (`AntivirusSignatureVersion`) a stáří (`AntivirusSignatureAge`) pro případný custom bypass.
        

## 🛑 2. Aplikační Whitelisting (AppLocker)

- [ ] **Vytáhnout efektivní politiku AppLockeru**
    
    - _Příkaz:_ `Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections`
        
    
    - [ ] **Zkontrolovat restrikce pro PowerShell / CMD**
        
        - Je blokována standardní 64-bitová cesta `%SYSTEM32%\WINDOWSPOWERSHELL\V1.0\POWERSHELL.EXE`?
            
    - [ ] **Vyzkoušet alternativní cesty pro bypass (pokud existuje restrikce):**
        
        - [ ] SysWOW64 verze: `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe`
            
        - [ ] PowerShell ISE: `PowerShell_ISE.exe`
            
    - [ ] **Zkontrolovat výjimky složek (Default Rules):**
        
        - Pouští politika spouštění čehokoliv z `C:\Windows\*` nebo `C:\Program Files\*` pro skupinu `Everyone`? (Ideální místa pro zápis payloadu).
            

## 💻 3. PowerShell Language Mode

- [ ] **Ověřit režim spouštění PowerShellu**
    
    - _Příkaz:_ `$ExecutionContext.SessionState.LanguageMode`
        
    - _Vyhodnocení:_
        
        - `FullLanguage` 🟢 -> Plný přístup, fungují všechny post-ex skripty, COM objekty a .NET třídy.
            
        - `ConstrainedLanguage` 🟡 (CLM) -> Omezený režim. Blokuje pokročilé útočné nástroje. Nutno kombinovat s AppLocker bypassem.
            

## 🔑 4. Microsoft LAPS (Local Administrator Password Solution)

Pokud je LAPS v síti nasazen, rotuje hesla lokálních adminů. Cílem je zjistit, kdo je může v AD číst.

- [ ] **Vyhledat delegované skupiny (kdo má právo číst LAPS přes GPO)**
    
    - _Příkaz (LAPSToolkit):_ `Find-LAPSDelegatedGroups`
        
    - _Co sledovat:_ Které specifické skupiny (mimo Domain Admins) mají delegované právo pro konkrétní Organizační Jednotky (OU).
        
- [ ] **Najít uživatele s "All Extended Rights"**
    
    - _Příkaz (LAPSToolkit):_ `Find-AdmPwdExtendedRights`
        
    - _Význam:_ Uživatelé, kteří např. připojili počítač do domény, mohou mít implicitní právo číst LAPS heslo tohoto stroje, i když nejsou v oficiální admin skupině.
        
- [ ] **Pokusit se o dump LAPS hesel (pokud náš aktuální kontext má právo číst)**
    
    - _Příkaz (LAPSToolkit):_ `Get-LAPSComputers`
        
    - _Úlovky:_ Poznamenat si vypsaná hesla v čistém textu (`Password`) a čas jejich expirace.
        

## 📝 Poznámky z aktuálního hosta:

Plaintext

```
Hostnaše: 
Aktuální uživatel: 
Defender: [ Aktivní / Neaktivní ]
Language Mode: [ Full / Constrained ]
Nalezené LAPS účty/hesla:
```