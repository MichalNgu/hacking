
### 🛠️ Nástroj `icacls`: Tvé oči v terminálu

Když máš jen příkazovou řádku, `icacls` je tvůj nejlepší přítel pro audit systému.

#### Jak číst výstup `icacls`:

Když napíšeš `icacls c:\windows`, uvidíš u každého uživatele zkratky v závorkách:

- **(OI) Object Inherit:** Právo se přenese na soubory v této složce.
    
- **(CI) Container Inherit:** Právo se přenese na podsložky.
    
- **(I) Inherited:** Právo bylo zděděno od nadřazené složky.
    

**Příklad z praxe:** `BUILTIN\Users:(RX)` znamená, že běžní uživatelé mohou systémové soubory číst a spouštět, ale nemohou je měnit.


### 🛠️ Sysinternals: Švýcarský nůž pro Windows

Tuto sadu nástrojů vytvořil Mark Russinovich a je to absolutní nutnost pro každého profesionála.

- **Process Explorer:** „Správce úloh na steroidech“. Ukazuje stromovou strukturu (který proces spustil který) a které DLL knihovny proces používá.
    
- **Process Monitor (ProcMon):** Sleduje v reálném čase každý přístup k souborům, registrům a síti. Skvělé pro hledání cest k _Privilege Escalation_.
    
- **psexec:** Umožňuje spustit proces na vzdáleném stroji (často používané pro laterální pohyb v síti).
    

> **Tip z praxe:** Můžeš je spustit přímo z internetu bez stahování zadáním `\\live.sysinternals.com\tools\` do průzkumníka souborů.