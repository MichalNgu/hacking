# 📚 Metasploit Teorie & Interní Fungování

---

# 🎯 Co je Metasploit?

Metasploit Framework je open-source platforma pro:

- vývoj exploitů
    
- testování zranitelností
    
- generování payloadů
    
- post-exploitation
    
- automatizaci bezpečnostního testování
    

Metasploit není jeden program, ale rozsáhlý framework složený z tisíců modulů.

Zjednodušeně:

```text
Zranitelnost
      │
      ▼
Exploit
      │
      ▼
Payload
      │
      ▼
Session
      │
      ▼
Post-Exploitation
```

---

# 🏗️ A. Architektura Modulů

Metasploit rozděluje funkčnost do několika kategorií.

---

## 1. Exploits

Moduly určené k využití zranitelnosti.

Úkol exploitu:

- zneužít chybu
    
- získat spuštění kódu
    
- doručit payload
    

Příklady:

```text
exploit/windows/smb/ms17_010_psexec
exploit/multi/http/apache_struts_rce
exploit/linux/http/example_rce
```

---

## 2. Payloads

Kód spuštěný po úspěšném exploitu.

Příklady:

```text
windows/x64/meterpreter/reverse_tcp
linux/x64/meterpreter/reverse_tcp
cmd/unix/reverse_bash
```

Typické úkoly:

- otevření shellu
    
- Meterpreter session
    
- spuštění příkazů
    

---

## 3. Auxiliary

Pomocné moduly.

Nevyužívají zranitelnost přímo.

Použití:

- skenování
    
- brute-force
    
- fingerprinting
    
- enumerace
    

Příklady:

```text
auxiliary/scanner/smb/smb_version
auxiliary/scanner/http/title
auxiliary/scanner/ssh/ssh_login
```

---

## 4. Post

Moduly používané po získání přístupu.

Použití:

- sběr dat
    
- enumerace systému
    
- eskalace oprávnění
    
- pivoting
    

Příklady:

```text
post/windows/gather/hashdump
post/multi/recon/local_exploit_suggester
```

---

## 5. Encoders

Historicky sloužily k úpravě payloadů.

Účel:

- obfuskace
    
- změna podpisů payloadů
    

Příklad:

```text
x86/shikata_ga_nai
```

Poznámka:

Moderní EDR a AV řešení encodery většinou obejdou, takže jejich význam je dnes výrazně menší než dříve.

---

# ⚙️ B. Ruby Architektura a Mixins

Metasploit je napsaný v jazyce Ruby.

Místo psaní všeho od začátku využívá tzv. Mixins.

Mixiny jsou předpřipravené moduly funkcionality.

---

## HTTP Client Mixin

```ruby
include Msf::Exploit::Remote::HttpClient
```

Přidává funkce:

```ruby
send_request_cgi
send_request_raw
normalize_uri
```

Vhodné pro:

- webové aplikace
    
- REST API
    
- RCE exploity
    

---

## TCP Mixin

```ruby
include Msf::Exploit::Remote::Tcp
```

Přidává:

```ruby
connect
disconnect
sock.put
sock.get
```

Použití:

- vlastní TCP protokoly
    
- síťové služby
    

---

## SMB Mixin

```ruby
include Msf::Exploit::Remote::SMB::Client
```

Použití:

- SMB
    
- Active Directory
    
- Windows exploity
    

---

# 🧱 Základní Struktura Modulu

```ruby
class MetasploitModule < Msf::Exploit::Remote

  Rank = ExcellentRanking

  def initialize(info = {})
    super(update_info(info,
      'Name'        => 'Example Exploit',
      'Description' => 'Demo module',
      'Author'      => ['Author'],
      'License'     => MSF_LICENSE
    ))
  end

  def exploit
    connect
    handler
  end

end
```

---

# 📊 Exploit Ranking

Metasploit hodnotí spolehlivost exploitů.

|Rank|Význam|
|---|---|
|Manual|Vyžaduje zásah operátora|
|Low|Nízká spolehlivost|
|Average|Průměrná stabilita|
|Normal|Dobrá použitelnost|
|Good|Vysoká spolehlivost|
|Great|Velmi stabilní|
|Excellent|Maximální stabilita|

---

# 💣 C. Payload Architektura

Payload není exploit.

Exploit pouze vytvoří podmínky pro spuštění payloadu.

---

## Staged Payload

Příklad:

```text
windows/meterpreter/reverse_tcp
```

Průběh:

```text
Exploit
    ↓
Malý Stager
    ↓
Stažení Meterpreteru
    ↓
Session
```

Výhody:

- menší počáteční přenos
    
- menší exploit payload
    

Nevýhody:

- potřebuje druhé spojení
    
- vyšší riziko selhání
    

---

## Stageless Payload

Příklad:

```text
windows/meterpreter_reverse_tcp
```

Průběh:

```text
Exploit
    ↓
Celý Payload
    ↓
Session
```

Výhody:

- stabilnější
    
- pouze jedno spojení
    

Nevýhody:

- větší velikost
    

---

# 🏭 D. MSFVenom Filozofie

MSFVenom generuje payloady do různých formátů.

---

## Obecná Syntaxe

```bash
msfvenom -p PAYLOAD LHOST=<IP> LPORT=<PORT> -f FORMAT
```

---

## Výběr Platformy

Windows:

```text
windows/x64/meterpreter/reverse_tcp
```

Linux:

```text
linux/x64/meterpreter/reverse_tcp
```

PHP:

```text
php/meterpreter_reverse_tcp
```

Java:

```text
java/meterpreter/reverse_tcp
```

---

## Výběr Formátu

|Formát|Použití|
|---|---|
|exe|Windows|
|elf|Linux|
|apk|Android|
|aspx|IIS|
|war|Java Tomcat|
|raw|Shellcode|
|ps1|PowerShell|

---

# 🔄 E. Workflow Portování Exploitu

Při převodu veřejného PoC do Metasploit modulu.

---

## Krok 1: Analýza PoC

Zjisti:

- jazyk
    
- argumenty
    
- URL endpointy
    
- požadavky
    
- autentizaci
    

Příklady:

```text
Python
C
Go
Ruby
```

---

## Krok 2: Najdi Podobný Modul

Vyhledej existující modul.

Například:

```bash
grep -R "HttpClient" modules/
```

nebo

```bash
search type:exploit http
```

---

## Krok 3: Definuj Parametry

Nejčastější:

```ruby
register_options(
[
  OptString.new(
    'TARGETURI',
    [true, 'Target Path', '/']
  )
])
```

Další běžné:

```text
RHOSTS
RPORT
SSL
TARGETURI
USERNAME
PASSWORD
```

---

## Krok 4: Implementace Logiky

Typicky:

```text
1. Odeslat request
2. Ověřit odpověď
3. Trigger exploit
4. Spustit payload
5. Handler
```

---

## Krok 5: Testování

Po úpravě modulu:

```bash
reload_all
```

Vyhledání:

```bash
search example
```

Načtení:

```bash
use exploit/example/module
```

Otestování:

```bash
check
run
```

---

# 🧠 Jak Metasploit Přemýšlí

Interně lze framework zjednodušit na:

```text
Target
   ↓
Module
   ↓
Exploit
   ↓
Payload
   ↓
Session
   ↓
Post Modules
   ↓
Persistence / Pivoting
```

Pokud tomuto řetězci rozumíš, rozumíš většině práce v Metasploitu.