# ☣️ 16. LLMNR/NBT-NS POISONING Z LINUXU (RESPONDER)

### 💡 Jak funguje princip selhání DNS?

Když počítač se systémem Windows hledá v síti jiný stroj (např. tiskárnu nebo sdílenou složku) a **DNS server ji nezná** (například kvůli překlepu uživatele: `\\printer01` místo `\\print01`), Windows se nevzdá.

Začne do celé lokální sítě křičet (broadcastovat) pomocí protokolů **LLMNR** (port 5355 UDP) a **NetBIOS** (port 137 UDP): _„Kdo v této síti ví, kde je `\\printer01`?“_

V tomto okamžiku zasáhne útočník s nástrojem **Responder**:

1. Odpoví oběti: _„Ano, to jsem já, já jsem `\\printer01`!“_
    
2. Oběť této odpovědi uvěří a pokusí se k útočníkovi připojit (např. přes protokol SMB).
    
3. Aby se oběť autentizovala, automaticky pošle útočníkovi své uživatelské jméno a **NetNTLMv2 hash** hesla.
    

## 🚀 Spuštění Responderu v ostrém režimu

Na svém útočném Linuxu (Kali/Parrot) spustíš Responder na síťovém rozhraní, které směřuje do sítě klienta.

Bash

```
# Spuštění Responderu na rozhraní ens224 v aktivním jedovitém režimu
sudo responder -I ens224
```

> ⚠️ **Důležité OPSEC pravidlo:** Před spuštěním se ujisti, že na tvém útočném stroji neběží služby jako Apache, Samba nebo DNS na portech (80, 445, 53), protože Responder si tyto porty potřebuje otevřít sám, aby mohl emulovat falešné servery a odchytávat přihlášení.

### Klíčové parametry Responderu:

- `-A` (Analyze mode): Pouze pasivně naslouchá (fly on the wall), neotravuje síť podvrženými odpověďmi.
    
- `-w` (WPAD proxy): Spustí falešný WPAD server. Extrémně účinné ve velkých firmách – zachytí HTTP provoz uživatelů, kteří otevřou internetový prohlížeč se zapnutou autodetekcí sítě.
    
- `-f` (Fingerprint): Pokusí se identifikovat verzi operačního systému stroje, který poslal dotaz.
    

## 🎯 Výstupy a struktura logů

Jakmile Responder zachytí hash, okamžitě ho vypíše do konzole a zároveň ho uloží do adresáře `/usr/share/responder/logs/` (případně do lokálního adresáře podle instalace).

Plaintext

```
# Ukázka vygenerovaných logů po úspěšném útoku
Analyzer-Session.log         -> Log pasivní analýzy
Poisoners-Session.log        -> Log odeslaných falešných odpovědí
SMB-NTLMv2-SSP-172.16.5.25.txt -> Samotný odchycený hash uživatele ze stroje .25
```

Obsah souboru s hashem vypadá jako specifický řetězec, který obsahuje uživatelské jméno, název domény, náhodný text (challenge) a samotný kryptografický otisk hesla: `FOREND::INLANEFREIGHT:4af70a79938ddf8a:0f85ad1e8...`

## 🔨 Prolomení NetNTLMv2 hashe v Hashcatu

Na rozdíl od lokálních LM/NT hashů, které lze zneužít přímo (útok Pass-the-Hash), síťové **NetNTLMv2 hashe nelze přímo vložit do přihlašovacích oken**. Musíš je buď rovnou zařídit do jiného útoku (SMB Relay), nebo je **prolomit offline** pomocí slovníku.

K prolomení použijeme GPU skener **Hashcat** s módem **`5600`**, který je určen výhradně pro NetNTLMv2.

Bash

```
# Spuštění Hashcatu proti uloženému hashi se slovníkem rockyou.txt
hashcat -m 5600 forend_ntlmv2.txt /usr/share/wordlists/rockyou.txt
```

### 📜 Výsledek úspěšného prolomení:

Plaintext

```
FOREND::INLANEFREIGHT:4af70a79938ddf8a:...:Klmcargo2

Session..........: hashcat
Status...........: Cracked
Hash.Name........: NetNTLMv2
Recovered........: 1/1 (100.00%) Digests
```

Uživatel **FOREND** používá slabé, 8 znaků dlouhé heslo **`Klmcargo2`**. Tímto jsi úspěšně získal své první platné čistovtextové údaje v doméně `INLANEFREIGHT.LOCAL`. Tento účet nyní představuje tvůj stabilní **Foothold**, ze kterého můžeš pokračovat hlouběji do struktury Active Directory.