### Co SIEM skutečně vidí

User-Agent je jen jeden z mnoha indikátorů. Moderní SOC obvykle sleduje také:

- Zdrojový proces (`powershell.exe`, `cmd.exe`, `python.exe`)
- Rodičovský proces (např. Word → PowerShell)
- Cílovou doménu/IP
- DNS dotazy
- TLS certifikáty a SNI
- Čas a frekvenci komunikace
- Vytvořené soubory
- Změny registrů
- PowerShell Script Block Logging
- Sysmon Event ID 1, 3, 11, 22 apod.

Proto samotná změna User-Agentu většinou nestačí k tomu, aby aktivita vypadala legitimně.

---

### CPTS pohled

Pro zkoušku je důležité umět:

#### Rozpoznat podezřelé User-Agenty

|User-Agent|Typický nástroj|
|---|---|
|`curl/x.x`|cURL|
|`Wget/x.x`|wget|
|`python-requests/x.x`|Python requests|
|`PowerShell/x.x`|PowerShell|
|`Microsoft BITS/x.x`|BITS|

#### Zkontrolovat HTTP hlavičky

Například:

```
curl -I https://example.com
```

nebo

```
wget --server-response https://example.com
```

---

### Co dnes obránci detekují nejčastěji

Na Windows:

- PowerShell stahující soubory z internetu
- Certutil komunikující přes HTTP/HTTPS
- BITS vytvářející netypické download úlohy
- MSHTA spouštějící vzdálený obsah
- Rundll32 spouštějící neobvyklé DLL

Na Linuxu:

- wget/curl stahující spustitelné soubory
- Python HTTP servery
- Netcat poslouchající na portech
- Neobvyklé SSH/SCP přenosy

---

### Doplnění do checklistu

Místo bodů zaměřených na obcházení detekce bych do studijních poznámek přidal:

```
[ ] Jaký proces vytvořil síťové spojení?
[ ] Jaký User-Agent byl použit?
[ ] Na jakou IP/doménu se komunikovalo?
[ ] Byl přenos šifrovaný?
[ ] Jsou v logu HTTP požadavky nebo DNS dotazy?
[ ] Existují Sysmon/Windows Event logy související s aktivitou?
```