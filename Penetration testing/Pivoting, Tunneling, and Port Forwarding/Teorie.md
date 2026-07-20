# 🌐 Pivoting, Port Forwarding & Lateral Movement

Pivoting, tunelování a lateral movement jsou techniky používané během interních pentestů po získání prvního přístupu. Cílem je dostat se z kompromitovaného systému do dalších částí infrastruktury, které nejsou přímo dostupné z útočného stroje.

---

# 🏗️ 1. Pivoting

## Co je Pivoting?

Pivoting znamená použití kompromitovaného systému jako prostředníka pro přístup do jiné sítě nebo segmentu.

Kompromitovaný systém se označuje jako:

- **Pivot Host**
    
- **Beachhead Host**
    

Útočník nepřistupuje přímo na cílový systém, ale využívá již ovládnutý stroj.

---

## Typický scénář

```
Útočník
   |
   |
Internet
   |
   |
Web Server (DMZ)
   |
   |
Interní síť
   |
   |
DC / Database / Internal Servers
```

Příklad:

- získám shell na web serveru v DMZ,
    
- server má přístup do interní sítě,
    
- přes něj skenuji a útočím na interní systémy.
    

---

## Cíl Pivotingu

- obejít síťovou segmentaci,
    
- získat přístup k interním službám,
    
- objevit nové cíle.
    

---

# 🚇 2. Tunneling

## Co je tunneling?

Tunelování znamená zapouzdření jednoho typu provozu do jiného protokolu.

Používá se pro:

- obcházení firewallů,
    
- skrytí komunikace,
    
- přístup k interním službám,
    
- C2 komunikaci.
    

---

## Typy tunelů

---

## DNS Tunneling

Data jsou přenášena pomocí DNS dotazů.

Použití:

- pomalá exfiltrace dat,
    
- obcházení omezení firewallu.
    

---

## SSH Tunneling

Šifrovaný tunel přes SSH.

Použití:

- přístup k interním službám,
    
- bezpečné přesměrování portů.
    

---

## HTTP/S Tunneling

Provoz je maskovaný jako běžný webový provoz.

Použití:

- obcházení filtrů,
    
- C2 komunikace.
    

---

# 🔌 3. Port Forwarding

Port forwarding umožňuje přesměrovat port vzdáleného systému na lokální port útočníka.

Používá se hlavně během pivotingu.

---

# 1. Local Port Forwarding (-L)

Použití:

Chci se z mého stroje připojit na interní službu přes pivot.

Příklad:

```
Útočník → Pivot → Interní MySQL
```

Příkaz:

```bash
ssh -L 8080:internal-server:3306 user@pivot-host
```

Výsledek:

Lokálně:

```
localhost:8080
```

=

Interní:

```
internal-server:3306
```

---

# 2. Remote Port Forwarding (-R)

Použití:

Interní systém se potřebuje připojit zpět k útočníkovi.

Příklad:

```bash
ssh -R 4444:localhost:4444 user@pivot-host
```

Použití:

- reverse shell,
    
- callback komunikace.
    

---

# 3. Dynamic Port Forwarding (-D)

Vytvoří SOCKS proxy.

Příklad:

```bash
ssh -D 9050 user@pivot-host
```

Následně:

```
proxychains nmap
proxychains firefox
proxychains sqlmap
```

Provoz jde přes pivot.

---

# ⚔️ 4. Lateral Movement

## Co je Lateral Movement?

Lateral movement znamená pohyb v rámci sítě po získání prvního přístupu.

Cílem je:

- získat další účty,
    
- zvýšit oprávnění,
    
- dostat se k důležitým systémům.
    

---

## Příklady technik

### Pass-the-Hash

Použití NTLM hashe místo hesla.

Například:

- SMB,
    
- WinRM,
    
- PsExec.
    

---

### Pass-the-Ticket

Použití Kerberos ticketů.

Použití:

- přístup k doménovým službám,
    
- pohyb v Active Directory.
    

---

### Zneužití služeb

Typicky:

- SMB,
    
- RDP,
    
- WinRM,
    
- SSH.
    

---

# 🔄 Pivoting vs Lateral Movement

||Pivoting|Lateral Movement|
|---|---|---|
|Cíl|Přístup do jiné sítě|Šíření v dostupné síti|
|Směr|Mezi segmenty|Mezi systémy|
|Hlavní problém|Síťová izolace|Přístup a oprávnění|
|Techniky|Tunely, proxy, port forwarding|PtH, PtT, RDP, SMB|
|Výsledek|Nová dostupná síť|Nové kompromitované systémy|

---

# 🛠️ Nástroje

|Nástroj|Použití|
|---|---|
|SSH|Port forwarding, tunely|
|Chisel|HTTP tunely přes firewall|
|Proxychains|Směrování nástrojů přes proxy|
|Ligolo-ng|Moderní pivoting přes tun/tap|
|Meterpreter|Route, portfwd|
|Socat|TCP/UDP forwarding|

---

# 🧪 Praktický pentest workflow

## 1. Získání prvního přístupu

Například:

```
Web Server
```

---

## 2. Enumerace pivot hostu

Kontrola:

```bash
ip a
ip route
ifconfig
```

Hledám:

- další síťová rozhraní,
    
- interní subnety.
    

---

## 3. Přístup do interní sítě

Použiji:

- Chisel,
    
- Ligolo-ng,
    
- SSH tunnel.
    

---

## 4. Interní skenování

Například:

```bash
proxychains nmap -sT 10.10.10.0/24
```

---

## 5. Lateral movement

Použití:

- získané credentials,
    
- NTLM hashů,
    
- Kerberos ticketů,
    
- zranitelných služeb.