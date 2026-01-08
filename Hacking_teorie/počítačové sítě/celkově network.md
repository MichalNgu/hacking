
# 🛡️ Základní kámen Pentestera: Síťový a Bezpečnostní Přehled

Tento dokument mapuje teoretické znalosti z HTB modulu na praktické útočné techniky.

---

## 🏗️ 1. Modely OSI vs. TCP/IP a jejich zranitelnosti
# 🛡️Networking & Security Cheat Sheet (OSI vs. TCP/IP)

| OSI Vrstva         | TCP/IP Vrstva   | Jednotka | Protokoly             | Funkce                                     | Hacking / Zneužití                                         |
| :----------------- | :-------------- | :------- | :-------------------- | :----------------------------------------- | :--------------------------------------------------------- |
| **7. Aplikační**   | **Aplikační**   | Data     | HTTP, FTP, SMB, DNS   | Rozhraní pro uživatele a síťové služby.    | **Web útoky:** SQLi, XSS, API exploity.                    |
| **6. Prezenční**   | **Aplikační**   | Data     | SSL, TLS, JPEG, ASCII | Šifrování, komprese a formátování dat.     | **Bypass šifrování**, SSL Stripping, útoky na certifikáty. |
| **5. Relační**     | **Aplikační**   | Data     | NetBIOS, RPC, SOCKS   | Správa relací a spojení mezi aplikacemi.   | **Session Hijacking**, krádež cookies, Session Fixation.   |
| **4. Transportní** | **Transportní** | Segment  | TCP, UDP              | Přenos mezi procesy (porty), spolehlivost. | **Port scanning**, SYN Flood, hádání ISN čísel.            |
| **3. Síťová**      | **Internetová** | Paket    | IP, ICMP, IPsec       | Směrování (Routing) a logická adresace.    | **IP Spoofing**, Recon přes TTL, ICMP tunneling.           |
| **2. Linková**     | **Přístupová**  | Rámec    | Ethernet, ARP, VLAN   | Fyzická adresace (MAC) a switchování.      | **ARP Poisoning (MITM)**, VLAN Hopping.                    |
| **1. Fyzická**     | **Přístupová**  | Bit      | Hub, Kabely, Wi-Fi    | Přenos nul a jedniček fyzickým médiem.     | **Odposlech (Sniffing)**, Wi-Fi Jamming.                   |
|                    |                 |          |                       |                                            |                                                            |
|                    |                 |          |                       |                                            |                                                            |
## 🕵️ 2. Síťová forenzika a rozpoznávání (Reconnaissance)

### Rozpoznání OS podle TTL (Time To Live)
Při pingu nebo skenování lze odhadnout operační systém cíle podle výchozí hodnoty TTL v IP hlavičce:
* **Linux/Unix:** TTL ≈ 64
* **Windows:** TTL ≈ 128
* **Cisco (IOS):** TTL ≈ 255

### Identifikace hostitele (IP ID)
* **Princip:** Pokud má stroj více IP adres (multi-homed), jeho **IP ID** v IP hlavičce obvykle číselně navazuje napříč všemi rozhraními.
* **Využití:** Identifikace, zda dvě různé IP adresy patří stejnému fyzickému hardwaru.

### Cisco Discovery Protocol (CDP)
* Pokud je zapnutý, odesílá pakety s názvem zařízení, verzí IOS a IP adresou. Pro útočníka je to "mapa k pokladu".

---

## 🔐 3. Kryptografie a Autentizace

### Symetrické vs. Asymetrické šifrování
* **Symetrické (AES, DES):** Rychlé, stejný klíč pro šifrování i dešifrování. 
    * *Pozor na DES:* Klíč 56 bitů je dnes prolomitelný hrubou silou během pár hodin!
* **Asymetrické (RSA, ECC):** Veřejný klíč (šifruje) a soukromý klíč (dešifruje). 
    * *ECC výhoda:* Menší klíče při stejné bezpečnosti jako RSA.

### Šifrovací módy (Cipher Modes)
* **ECB (Electronic Code Book):** Nebezpečný! Nezakrývá vzory v datech.
* **CBC / GCM:** Bezpečnější volby. GCM navíc zajišťuje integritu dat.

### VPN a IKE (Internet Key Exchange)
* **Aggressive Mode:** Zranitelný. Odesílá hash předsdíleného klíče (PSK) v první fázi. Útočník ho může zachytit a cracknout offline pomocí slovníku.
* **Main Mode:** Bezpečnější, chrání identitu účastníků.

---

## 🛠️ 4. Praktické útoky na síťovou vrstvu

### VLAN Hopping (Double Tagging)
1.  Útočník pošle rámec se dvěma tagy 802.1Q.
2.  První switch odstraní vnější tag (Native VLAN).
3.  Druhý switch uvidí vnitřní tag a doručí paket do cílové (vzdálené) VLAN.
* **Podmínka:** Útočník musí být v Native VLAN daného trunku.

### VPN Tunelování (HTB Základ)
* VPN přiděluje útočníkovi **vnitřní IP adresu** (např. 10.10.x.x).
* Celá komunikace je zapouzdřena v šifrovaném tunelu (ESP v IPsec nebo OpenVPN), což umožňuje bezpečný přístup k labům.

---

## 💻 5. Tahák pro Linux Terminál (VLAN & Recon)

```bash
# Načtení modulu pro práci s VLAN
sudo modprobe 8021q

# Vytvoření VLAN rozhraní (např. VLAN 20 na kartě eth0)
sudo ip link add link eth0 name eth0.20 type vlan id 20
sudo ip addr add 192.168.20.5/24 dev eth0.20
sudo ip link set up eth0.20

# Síťový průzkum cesty k cíli
ping -c 1 -R <target_ip>   # Record-Route (pokud není blokováno)
traceroute <target_ip>      # Mapování routerů na cestě