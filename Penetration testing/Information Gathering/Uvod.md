## 🎯 Cíle webového průzkumu

Než začnete psát první příkazy, musíte vědět, co vlastně hledáte:

1. Identifikace aktiv: Najít vše, co firma vlastní (subdomény, zapomenuté testovací servery, IP adresy).
2. Odhalení skrytých informací: Zálohy webů (config.php.bak), interní dokumentace omylem vystavená ven nebo konfigurační soubory.
3. Analýza "Attack Surface" (plocha útoku): Zmapování technologií (běží tam zastaralá verze Apache? Používají WordPress?).
4. Zpravodajství (Intelligence): Jména zaměstnanců pro sociální inženýrství nebo struktura e-mailových adres.

Metodologie: Aktivní vs. Pasivní Recon

Průzkum dělíme podle toho, jak moc "klepeme na dveře" cílového systému.

### 1. Pasivní průzkum (Passive Recon)

Při pasivním průzkumu přímo nekomunikujete s cílovým serverem. Využíváte zdroje třetích stran. Cíl nemá šanci zjistit, že se o něj zajímáte.

- Google Dorking: Používání speciálních operátorů v Googlu (např. site:target.com filetype:log).
- WHOIS: Dotazování do veřejných databází o vlastníkovi domény, DNS serverech a datu registrace.
- Wayback Machine: Prohlížení historických verzí webu (můžete tam najít soubory, které už dnes na webu nejsou, ale zůstaly v archivu).
- OSINT (Open Source Intelligence): Prohledávání LinkedInu, GitHubu (hledání zapomenutých hesel v kódu) nebo sociálních sítí.

### 2. Aktivní průzkum (Active Recon)

Zde přímo interagujete s cílem. Posíláte pakety na jeho porty, zkoušíte jeho formuláře. Riziko odhalení je vysoké (firewally a IDS vás mohou zablokovat).

Port Scanning- Seznam otevřených služeb (HTTP, SSH, DB)

Banner Grabbing- Přesná verze softwaru (např. "Nginx 1.14.0").

Directory Brute Forcing- Nalezení skrytých složek jako /admin nebo /backup.

Web Spidering- Automatické prolezení všech odkazů na webu.

Při detailním pohledu jsou první kroky WHOIS a DNS
```
Whois victim.com
```