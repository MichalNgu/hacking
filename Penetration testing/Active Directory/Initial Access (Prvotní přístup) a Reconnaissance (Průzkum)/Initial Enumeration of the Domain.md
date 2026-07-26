# 🕵️‍♂️ 15. PRVOTNÍ VNITŘNÍ ENUMERACE DOMÉNY

## 👂 1. Pasivní naslouchání sítě (Passive Reconnaissance)

Předtím, než do sítě pošleš jediný paket, se vyplatí "přiložit ucho na drát". Využívají se k tomu síťové sniffery jako **Wireshark** nebo **tcpdump**. V přepínaném (switched) prostředí vidíš provoz v rámci své broadcast domény.

- **Co hledat:** ARP dotazy, MDNS (Multicast DNS), LLMNR nebo NetBIOS (NBT-NS) požadavky.
    
- **Příklad z praxe:** * Z ARP paketů okamžitě zjistíš živé IP adresy v okolí (`172.16.5.5`, `172.16.5.25`, atd.).
    
    - Z MDNS dotazů odhalíš konkrétní názvy strojů, např. `ACADEMY-EA-WEB01`.
        

### Pasivní analýza přes Responder

Nástroj **Responder** se spustí v analytickém módu (`-A`), což znamená, že síť neotravuje podvrženými odpověďmi, ale pouze pasivně loguje, kdo v síti křičí o pomoc (hledá neexistující sdílené složky nebo tiskárny).

Bash

```
sudo responder -I ens224 -A
```

_Tímto způsobem bezpečně sestavíš první seznam IP adres a NetBIOS jmen bez rizika, že tě odhalí obrana._

## 🎯 2. Aktivní ověřování cílů (Active Sweeping)

Jakmile máš hrubou představu, přejde se k rychlému a efektivnímu ICMP (ping) sweepu celé podsítě pomocí nástroje **fping**. Na rozdíl od klasického pingu posílá dotazy paralelně (round-robin), což dramaticky zrychluje proces.

Bash

```
# -a (pouze živé), -s (statistiky), -g (generuj rozsah z CIDR), -q (skryj chyby)
fping -asgq 172.16.5.0/23
```

- **Výsledek:** Získáš přesný seznam aktivních IP adres (např. 9 živých hostitelů), které následně předhodíš detailnějšímu skeneru.
    

## 🗺️ 3. Cílené skenování portů přes Nmap

Získaný seznam IP adres vložíš do souboru `hosts.txt` a spustíš **Nmap**. Protože tě zajímá Active Directory, sleduješ specifické porty:

- **53 (DNS)**, **88 (Kerberos)**, **389/636 (LDAP/LDAPS)**, **445 (SMB)**, **3389 (RDP)**.
    

Bash

```
sudo nmap -v -A -iL hosts.txt -oA internal_enum_results
```

### 🔍 Klíčové nálezy z Nmap výstupu v příkladu:

1. **Stroj 172.16.5.5 (Primární Domain Controller)**
    
    - Běží zde port 88 (Kerberos) a 389 (LDAP).
        
    - Nmap ze SSL certifikátu a RDP hlavičky vytáhne kompletní doménové jméno: **`ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL`**.
        
2. **Stroj 172.16.5.100 (Legacy Systém – Zlatý důl pro útočníka)**
    
    - Nmap hlásí přítomnost starého webového serveru Microsoft IIS 7.5 a operačního systému **Windows Server 2008 R2**.
        
    - **OPSEC Varování:** Staré systémy jsou náchylné k pádům (např. při zkoušení EternalBlue). Před jakýmkoliv pokusem o exploitaci na reálném pentestu je nutné mít písemný souhlas klienta, protože pád produkčního serveru může firmu stát velké peníze.
        

## 👥 4. Sběr uživatelů přes Kerberos (Kerbrute)

Když znáš IP adresu Domain Controlleru (`172.16.5.5`), můžeš začít zjišťovat validní uživatelská jména v doméně, aniž bys měl jakékoliv heslo. Využívá se nástroj **Kerbrute**.

- **Proč to funguje:** Kerbrute posílá požadavky na Kerberos před-autentizaci (TGT requests). Pokud uživatelské jméno v AD existuje, DC odpoví specifickou chybou (požaduje heslo). Pokud neexistuje, odpoví, že uživatel je neznámý.
    
- **Výhoda:** Selhání před-autentizace často **nezpůsobuje zápis do standardních logů** selhání přihlášení, takže je tento útok extrémně rychlý a tichý.
    

Bash

```
# Kompilace nástroje ze zdrojových kódů (Best Practice pro čistotu binárek)
sudo git clone https://github.com/ropnop/kerbrute.git && cd kerbrute
sudo make linux
sudo mv dist/kerbrute_linux_amd64 /usr/local/bin/kerbrute

# Spuštění uživatelské enumerace proti DC
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```

- **Výsledek:** Během 10 sekund získáš desítky platných doménových jmen (např. `jjones@INLANEFREIGHT.LOCAL`, `sbrown@INLANEFREIGHT.LOCAL`).
    

## 👑 Význam účtu NT AUTHORITY\SYSTEM

Pokud se ti během této neověřené fáze podaří ovládnout starý server (např. ten nalezený Windows Server 2008) a získáš práva **SYSTEM**, máš vyhráno.

Účet SYSTEM na stroji, který je v doméně, komunikuje s Active Directory pod identitou samotného počítače (např. `ACADEMY-EA-CTX1$`). Active Directory bere počítačové účty jako běžné doménové uživatele. S právy SYSTEM na jednom stroji tedy můžeš začít provádět kompletní vnitřní mapování domény (přes BloodHound nebo PowerView) a útočit na Kerberos (Kerberoasting) stejně, jako bys měl k dispozici standardní uživatelské jméno a heslo.