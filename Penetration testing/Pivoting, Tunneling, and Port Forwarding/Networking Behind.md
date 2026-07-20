# 🌐 Networking Basics for Pivoting

Síťové základy jsou klíčové pro správné provedení pivotingu. Bez pochopení IP adres, routingu a síťové komunikace není možné efektivně nastavovat tunely ani pohybovat se mezi segmenty infrastruktury.

---

# 🔌 1. IP adresy a síťová rozhraní (NIC)

Každé zařízení komunikuje pomocí IP adresy přiřazené ke konkrétnímu síťovému rozhraní.

Během pentestu je důležité hledat stroje s více síťovými rozhraními, protože mohou sloužit jako pivot point.

---

## Typy rozhraní

### Public IP

Veřejná IP adresa dostupná z internetu.

Příklad:

```text
Internet
   |
DMZ Server
eth0: Public IP
```

Použití:

- web servery,
    
- VPN gateway,
    
- veřejné služby.
    

---

### Private IP

Interní adresy používané uvnitř organizace.

Typické rozsahy:

```text
10.0.0.0/8

172.16.0.0/12

192.168.0.0/16
```

Tyto sítě nejsou přímo dostupné z internetu.

---

### Loopback

Lokální komunikace na stejném stroji.

Adresa:

```text
127.0.0.1
```

Použití:

- lokální služby,
    
- testování aplikací.
    

---

### Tunnel Interface

Virtuální síťové rozhraní vytvořené tunelem nebo VPN.

Příklady:

```text
tun0
tap0
```

Použití:

- HTB VPN,
    
- Ligolo-ng,
    
- OpenVPN.
    

---

# 🔎 Enumerace síťových rozhraní

Po získání přístupu vždy kontroluji síťovou konfiguraci.

---

## Linux

```bash
ifconfig
```

nebo:

```bash
ip a
```

---

## Windows

```cmd
ipconfig
```

---

## Pivot Host indikátor

Pokud vidím více privátních sítí:

Příklad:

```text
eth0:
10.10.10.5

eth1:
192.168.1.5
```

znamená to:

- systém má přístup do dvou sítí,
    
- může sloužit jako pivot.
    

---

# 🛣️ 2. Routing

Routing určuje, kam operační systém pošle síťový provoz.

---

## Routing tabulka

Ukazuje:

- dostupné sítě,
    
- gateway,
    
- rozhraní.
    

---

## Linux

```bash
route -n
```

nebo:

```bash
ip route
```

---

## Windows

```cmd
route print
```

---

# Default Gateway

Výchozí cesta pro provoz, který nemá specifické pravidlo.

Příklad:

```text
Internal Network
       |
    Gateway
       |
   Internet
```

---

# Static Routes

Pravidla určující cestu do konkrétní sítě.

Příklad:

```text
10.10.20.0/24
       |
       |
Pivot Host
```

Útočný stroj musí vědět:

"Pro síť 10.10.20.0/24 použij pivot."

---

# 🔄 AutoRoute

Nástroje jako Metasploit umí automaticky přidat routy přes existující session.

Výsledek:

```text
Útočník
   |
Tunnel
   |
Pivot Host
   |
Interní síť
```

---

# 🚢 3. Protokoly a porty

IP adresa určuje zařízení.

Port určuje konkrétní službu.

---

## Příklady

|Port|Služba|
|---|---|
|22|SSH|
|80|HTTP|
|443|HTTPS|
|445|SMB|
|3389|RDP|
|1433|MSSQL|
|3306|MySQL|

---

# Firewall a tunelování

Firewally často:

- blokují neznámé porty,
    
- povolují běžné služby.
    

Typicky:

Povoleno:

```text
80 HTTP
443 HTTPS
```

Blokováno:

```text
4444
5555
9001
```

Proto se používá:

- HTTP tunneling,
    
- HTTPS tunneling,
    
- SSH tunneling.
    

Cílem je ukrýt komunikaci do povoleného provozu.

---

# 🎨 4. Network Mapping

Během pivotingu je důležité vytvářet síťovou mapu.

Příklad:

```text
             Internet

                |
                |

          Web Server
          10.10.10.5
                |
                |
        Internal Network

        192.168.1.0/24

          DC01
          SQL01
          FILE01
```

---

# Proč kreslit diagram?

## 1. Přehled infrastruktury

Vidím:

- počet segmentů,
    
- pivot body,
    
- cestu k cíli.
    

---

## 2. Cleanup

Po ukončení testu vím:

- jaké tunely vypnout,
    
- jaké změny odstranit,
    
- kde jsem zasahoval.
    

---

## 3. Reporting

Síťový diagram pomáhá klientovi pochopit:

- špatnou segmentaci,
    
- rizika lateral movementu,
    
- cestu útočníka.
    
