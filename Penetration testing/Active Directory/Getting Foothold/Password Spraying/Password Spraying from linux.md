# 🐧 19. INTERNAL PASSWORD SPRAYING Z LINUXU

K provedení útoku z Linuxu (Kali/Parrot) existují tři hlavní a vysoce efektivní cesty. Každá využívá jiný protokol a má svá specifika.

### A) Protokol SMB: CrackMapExec (CME) / NetExec

CME je švýcarský nůž pro vnitřní síť. Dokáže jako vstup přijmout celý soubor uživatelů a otestovat je proti jednomu heslu. Aby ses neutopil v záplavě řádků o neúspěšných pokusech, filtruj výstup pomocí `grep +`.

Bash

```
# Spuštění password spraye proti DC s heslem Password123
sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +
```

- **Výsledek:** Pokud CME vypíše zelené `[+]`, přihlášení bylo úspěšné (např. `INLANEFREIGHT.LOCAL\avazquez:Password123`).
    

### B) Protokol Kerberos: Kerbrute

Kerbrute posílá požadavky na lístky TGT (port 88 UDP). Je **extrémně rychlý** (stovky pokusů za sekundu) a generuje v logách Windows Event ID 4768 namísto Event ID 4625 (selhání přihlášení).

Bash

```
# Rychlý password spray pomocí protokolu Kerberos
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt Welcome1
```

### C) Protokol RPC: rpcclient (Bash One-Liner)

Pokud na stroji nemáš pokročilé nástroje, vystačíš si s vestavěným nástrojem `rpcclient` a jednoduchou vestavěnou smyčkou v bashi. Úspěšné přihlášení poznáš podle přítomnosti řetězce `Authority Name`.

Bash

```
# Smyčka pro postupné testování uživatelů přes rpcclient s heslem Welcome1
for u in $(cat valid_users.txt); do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

## 🎯 Útok na lokální administrátory (Local Administrator Password Reuse)

Password spraying se nemusí omezovat pouze na doménové (Active Directory) účty. Obrovským nešvarem firem je **opakované používání stejného hesla pro lokálního administrátora** (`Administrator`) na všech stanicích a serverech (často kvůli nasazování operačních systémů z jednoho univerzálního obrazu/image).

Pokud se ti podaří z lokální databáze SAM jednoho počítače vytáhnout čisté heslo nebo **NTLM hash** lokálního administrátora, můžeš spustit plošný spray přes celou podsíť.

Bash

```
# Spraying lokálního NT hashe přes celou podsíť /23 s příznakem --local-auth
sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +
```

### 🔥 Klíčové flagy a pojmy pro tento útok:

- **`--local-auth` (Kriticky důležité):** Říká nástroju, aby se ověřoval lokálně vůči dané stanici, nikoliv vůči doméně. Bez tohoto flagu by se CME pokusil ověřit doménového administrátora, což by vedlo k **okamžitému uzamčení účtu Domain Admin napříč celou firmou!**
    
- **`(Pwn3d!)`:** Pokud CME uvidí tento řetězec, znamená to, že dané přihlašovací údaje mají na cílovém stroji nejvyšší administrátorská práva. Můžeš na daný stroj rovnou nahrát payload nebo z něj vytáhnout paměť LSASS (Lateral Movement).
    

## 🛡️ Doporučená Remediance pro klienta

Pokud během pentestu uspěješ s útokem na lokální administrátory, tvé doporučení pro klienta je jednoznačné: **Nasazení Microsoft LAPS (Local Administrator Password Solution)**.

LAPS zajistí, že Active Directory bude automaticky generovat silné, unikátní a pravidelně rotované heslo pro lokálního administrátora každého jednotlivého počítače v síti. Útok typu Local Admin Spraying se tím stává kompletně nefunkčním.