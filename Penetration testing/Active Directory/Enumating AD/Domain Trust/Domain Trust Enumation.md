### Domain Trusts

Trust (důvěra) umožňuje autentizaci mezi doménami nebo lesy. Pro útočníka je to klíčová cesta k eskalaci privilegií ("end-around" útoky).

- **Transitive (Tranzitivní):** Důvěra se přenáší dál (pokud A důvěřuje B a B důvěřuje C, A důvěřuje C). Typické pro vnitřní vztahy (parent-child, forest).
    
- **Non-Transitive:** Důvěra platí jen mezi dvěma konkrétními subjekty.
    
- **Směr:**
    
    - **One-way:** Uživatelé z důvěryhodné domény (trusted) přistupují k prostředkům v doméně, která důvěřuje (trusting).
        
    - **Bi-directional:** Obousměrná autentizace.
        
- **Riziko:** Často špatně nakonfigurované nebo zapomenuté vztahy (M&A). Umožňují útočit na "měkčí" cíl v přidružené doméně a získat práva v hlavní doméně.
    

### Příkazy pro enumeraci

#### 1. Built-in Active Directory PowerShell

Základní enumerace pomocí nativních nástrojů (vyžaduje `RSAT-AD-PowerShell`).

PowerShell

```
Import-Module activedirectory
Get-ADTrust -Filter *c v
```

#### 2. PowerView (SharpView)

Efektivnější pro mapování vztahů a rychlý přehled.

PowerShell

```
# Výpis trustů
Get-DomainTrust

# Detailní mapování všech vztahů (včetně obousměrných)
Get-DomainTrustMapping

# Enumerace uživatelů v cizí (např. child) doméně
Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName
```

#### 3. Netdom

Starší, ale nativně dostupný nástroj v systémech Windows.

PowerShell

```
# Zjištění trustů
netdom query /domain:inlanefreight.local trust

# Seznam DC v doméně
netdom query /domain:inlanefreight.local dc

# Seznam stanic a serverů v doméně
netdom query /domain:inlanefreight.local workstation
```

#### 4. BloodHound

Nejlepší pro vizualizaci. Využij v rámci post-exploatace předpřipravený query:

- **Map Domain Trusts** (vizualizace cesty a typu vztahů).
- ![[Pasted image 20260720135828.png]]