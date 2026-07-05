# 🪟 17. LLMNR/NBT-NS POISONING Z WINDOWS (INVEIGH)

### 💡 Inveigh vs. Responder

Inveigh plní naprosto stejnou roli jako Responder (vytváří Man-in-the-Middle a podvrhuje odpovědi na síťové dotazy), ale je napsaný v **PowerShellu** a **C#**. To z něj dělá ideální zbraň, pokud nemůžeš nebo nechceš do sítě připojovat Linux.

- **Původní PowerShell verze (`Inveigh.ps1`)** je dnes považována za legacy a již se neaktualizuje.
    
- **Moderní C# verze (`Inveigh.exe` / InveighZero)** je aktivně vyvíjená, funguje jako samostatná binárka a lépe obchází některé bezpečnostní mechanismy.
    

## 🚀 Spuštění Inveigh v praxi

Spuštění C# verze s výchozím nastavením zachytávání (LLMNR, NBNS, SMB a HTTP/HTTPS) provedeme přímo z příkazové řádky:

PowerShell

```
# Spuštění Inveigh.exe na Windows útočném stroji
.\Inveigh.exe
```

Nástroj začne okamžitě naslouchat a posílat falešné odpovědi obětem. Během běhu můžeš stisknout klávesu **`ESC`**, čímž vstoupíš do **interaktivní konzole Inveigh**.

### 📊 Práce s úlovky v konzoli:

V interaktivním režimu máš k dispozici příkazy pro rychlé zobrazení výsledků:

- `GET NTLMV2UNIQUE` – Vypíše unikátní zachycené hashe (např. pro uživatele `backupagent` nebo `forend`).
    
- `GET NTLMV2USERNAMES` – Vypíše tabulku s IP adresou oběti, názvem počítače a uživatelským jménem, které se pokusilo autentizovat.
    

_V tomto bodě získáváš přesně ty NetNTLMv2 hashe, které následně odnášíš na svůj lámací stroj k offline crackování přes Hashcat._

## 🛠️ Mitigace a Remediance (Obrana)

Jako penetrační tester musíš klientovi ukázat nejen jak síť ovládnout, ale hlavně jak ji zabezpečit. Vypnutí těchto protokolů je jedním z nejlevnějších a nejefektivnějších kroků k zabezpečení vnitřní sítě.

### 1. Vypnutí LLMNR přes Group Policy (GPO)

LLMNR lze snadno zakázat centrálně pro celou doménu v editoru Group Policy:

> **Konfigurace počítače** (Computer Configuration) ➡️ **Administrativní šablony** (Administrative Templates) ➡️ **Síť** (Network) ➡️ **Klient DNS** (DNS Client) ➡️ Povolit pravidlo: **Vypnout vyhledávání názvů pomocí vícesměrového vysílání (Turn OFF Multicast Name Resolution)**.

### 2. Vypnutí NetBIOS (NBT-NS) pomocí PowerShell skriptu

NetBIOS nelze ve Windows vypnout kliknutím na jedno globální GPO tlačítko. Musí se zakázat na úrovni každého síťového adaptéru. Pro plošné nasazení se vytváří PowerShell skript spouštěný při startu počítače (Startup Script) přes GPO:

PowerShell

```
# Skript pro vyhledání všech síťových rozhraní v registru a vypnutí NetBIOS (Hodnota 2 = Disable)
$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
Get-ChildItem $regkey | foreach { 
    Set-ItemProperty -Path "$regkey\$($_.pschildname)" -Name NetbiosOptions -Value 2
}
```

Tento skript se umístí na sdílené úložiště Domain Controlleru (**`SYSVOL`**) a prováže se s GPO. Po restartu stanic je NetBIOS kompletně deaktivován.

## 🏷️ Zařazení do fází AD útoků a obran

Abychom propojili celý tento text s metodikou, zde je přehledné rozdělení jednotlivých kroků do fází:

|**Část textu / Technika**|**📍 Kam to patří v rámci útoku/obrany**|**Vysvětlení**|
|---|---|---|
|**Spuštění Inveigh a odchyt hashů**|**Initial Access (Prvotní přístup) / MITM**|Používáš techniku Man-in-the-Middle k získání přihlašovacích údajů reálných uživatelů ze sítě.|
|**Příkaz `GET NTLMV2USERNAMES`**|**Internal Reconnaissance (Vnitřní průzkum)**|Zjišťuješ, jaké účty v síti reálně existují a jaké stroje (např. `ACADEMY-EA-FILE`) generují provoz.|
|**Vypnutí LLMNR a NetBIOS (GPO/Skripty)**|**Hardening / Remediation (Zabezpečení infrastruktury)**|Defenzivní fáze. Odstraňuješ legacy protokoly a tím kompletně likviduješ celou tuto třídu útoků.|
|**Detekce pomocí falešných požadavků (Honey-targets)**|**Active Detection / Threat Hunting (Detekce incidentů)**|Obranný mechanismus. Záměrně generuješ do sítě dotazy na neexistující stroje a pokud někdo (Inveigh/Responder) odpoví, okamžitě víš, že máš v síti útočníka.|