
|**Typ Fuzzingu**|**Příkaz ffuf**|
|---|---|
|**Adresáře (Directories)**|`ffuf -w wordlist.txt:FUZZ -u http://TARGET/FUZZ`|
|**Přípony (Extensions)**|`ffuf -w extensions.txt:FUZZ -u http://TARGET/indexFUZZ`|
|**Soubory (Files)**|`ffuf -w wordlist.txt:FUZZ -u http://TARGET/FUZZ.php`|
|**Rekurzivní (Recursive)**|`ffuf -w wordlist.txt:FUZZ -u http://TARGET/FUZZ -recursion -recursion-depth 1 -e .php -v`|
|**Poddomény (DNS)**|`ffuf -w subdomains.txt:FUZZ -u http://FUZZ.TARGET/`|
|**VHosty (Virtual Hosts)**|`ffuf -w subdomains.txt:FUZZ -u http://TARGET/ -H 'Host: FUZZ.TARGET' -fs <size>`|
|**GET Parametry**|`ffuf -w params.txt:FUZZ -u http://TARGET/page.php?FUZZ=key -fs <size>`|
|**POST Parametry**|`ffuf -w params.txt:FUZZ -u http://TARGET/page.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs <size>`|
|**Hodnoty parametrů**|`ffuf -w values.txt:FUZZ -u http://TARGET/page.php -X POST -d 'param=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs <size>`|
