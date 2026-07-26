# 📖Pokročilé Techniky Auditing Active Directory (AD Auditing)

Při provádění bezpečnostních posouzení Active Directory a pentestů je klíčové nejen odhalit cesty k převzetí domény, ale také poskytnout zákazníkovi podrobné podklady pro nápravu. Použití specializovaných auditovacích nástrojů umožňuje získat přesná data, vizualizace a vyhodnocení rizik podle uznávaných standardů (např. CMMI).

## 🔍 1. Sysinternals AD Explorer (Active Directory Explorer)

**AD Explorer** je pokročilý prohlížeč a editor Active Directory z balíčku Sysinternals.

### Hlavní funkce a využití:

- **Offline prohlížení (AD Snapshots):** Umožňuje vytvořit kompletní snímek AD databáze v daném čase a analyzovat jej offline během fáze psaní reportu.
    
- **Porovnávání stavu (Diffing):** Porovnání dvou snímků (před a po) k odhalení změn v objektech, atributování a bezpečnostních oprávněních (DACL).
    
- **Vyhledávání:** Komplexní vyhledávací dotazy nad atributy objektů bez nutnosti složitého psaní LDAP filtrů.
    

### Postup vytvoření snapshotu:

1. Spustit `ADExplorer.exe` a přihlásit se pomocí běžných doménových kredenčních údajů.
    
2. V horním menu zvolit **File ➔ Create Snapshot**.
    
3. Zadat název souboru a uložit `.dat` snímek pro offline analýzu.
    

## 🏰 2. PingCastle

**PingCastle** vyhodnocuje bezpečnostní profil AD prostředí na základě matice rizik a metodiky Capability Maturity Model Integration (**CMMI**). Na rozdíl od BloodHoundu se zaměřuje na vyhodnocení rizikovosti a zralosti zabezpečení domény.

> ⚠️ **Tip pro lab / zkoušku:** Pokud nástroj selže při spuštění z důvodu expirace licenčního/podpůrného data, nastavte systémové datum v Control Panelu na datum před **31. červencem 2023**.

### Hlavní funkce a nabídka TUI:

- `1 - healthcheck`: Vytvoří základní zprávu o rizicích domény (Healthcheck Report v HTML).
    
- `2 - conso`: Agreguje více reportů z různých domén do jednoho.
    
- `3 - carto`: Vytvoří mapu všech propojených domén a důvěrností.
    
- `4 - scanner`: Spustí specifické bezpečnostní prověrky na stanicích/serverech.
    

### Skenerové moduly (`scanner`):

|Modul|Popis auditované oblasti|
|---|---|
|`aclcheck`|Kontrola nebezpečných ACL oprávnění pro uživatele a skupiny.|
|`laps_bitlocker`|Skenování stavu nasazení LAPS a BitLockeru.|
|`nullsession` / `nullsession-trust`|Detekce anonymního přístupu a null sessions.|
|`spooler`|Kontrola běžící služby Print Spooler na doménových řadičích.|
|`zerologon` / `remote` / `smb`|Detekce kritických zranitelností a nastavení SMB.|

## 🛠️ 3. Group3r (Audit Group Policy Objects)

**Group3r** je specializovaný CLI nástroj určený k vyhledávání zranitelností a chyb v nastavení Group Policy (GPO). Musí být spuštěn z doménového stroje pod doménovým účtem (nevyžaduje admin práva).

### Základní syntaxe:

```
# Výstup do souboru
group3r.exe -f gpo_audit.log

# Výstup na standardní výstup (stdout)
group3r.exe -s
```

### Interpretace výstupu:

Struktura výstupu využívá odsazení pro přehlednost:

- **Bez odsazení:** Název konkrétní GPO.
    
- **1. úroveň odsazení:** Konkrétní politika / nastavení uvnitř GPO.
    
- **2. úroveň odsazení:** Nalezená zranitelnost / nález (Finding) s odůvodněním rizikovosti.
    

## 📊 4. ADRecon

**ADRecon** je komplexní PowerShell skript, který získá rozsáhlé množství dat z Active Directory a vygeneruje přehledné výstupy ve formátu HTML a CSV (případně Excel `.xlsx`, pokud je na stroji nainstalován MS Excel).

### Spuštění:

```
PS C:\htb> .\ADRecon.ps1
```

### Co ADRecon automaticky sbírá:

- Doménovou strukturu, lesy, doménové vztahy (Trusts), doménové řadiče (DCs).
    
- Uživatele, SPN účty, heslovou politiku (Default i Fine-Grained Password Policies).
    
- Skupiny a změny v jejich členství.
    
- Strukturu OU, GPO reporty (`GPOReport.html`), DNS zóny a záznamy.
    
- Informace o LAPS a BitLockeru (vyžaduje privilegovaný účet).
    

> 💡 **Tip k reportování:** Pokud na cílovém stroji není nainstalován Microsoft Excel, ADRecon vygeneruje pouze CSV soubory a HTML zprávu pro GPO. Excelový výstup lze dodatečně vygenerovat na jiném stroji pomocí parametru `-GenExcel -ImportDir <cesta_k_CSV_slozce>`.

# 📋 CPTS Summary Checklist: AD Auditing Tools

### 🔲 Fáze 1: Rychlý přehled a rizika (PingCastle)

- [ ] **Spustit Healthcheck:** `PingCastle.exe` -> varianta `1-healthcheck`.
    
- [ ] **Spustit specifické skenery:** Prověřit Spooler (`spooler`), Null Sessions (`nullsession`) a LAPS (`laps_bitlocker`).
    
- [ ] **Uložit HTML report:** Zkontrolovat celkové skóre domény a sekci "Anomalies".
    

### 🔲 Fáze 2: Hloubkový audit GPO (Group3r)

- [ ] **Spustit Group3r:** `group3r.exe -f group3r_report.log`.
    
- [ ] **Prohledat log:** Zaměřit se na hesla v GPO, nadbytečná práva v Local Group Policy a slabé konfigurace.
    

### 🔲 Fáze 3: Offline záloha a rozsáhlý sběr dat (AD Explorer & ADRecon)

- [ ] **AD Explorer Snapshot:** Vytvořit `.dat` snímek přes **File ➔ Create Snapshot** pro pozdější offline analýzu.
    
- [ ] **ADRecon Sběr:** Spustit `.\ADRecon.ps1` pro kompletní export konfiguračních dat do CSV a HTML.
    
- [ ] **Generování Excelu:** V případě potřeby zkonvertovat CSV výstupy do `.xlsx` na analytickém stroji.