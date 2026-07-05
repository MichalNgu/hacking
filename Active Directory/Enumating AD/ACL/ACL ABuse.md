### ACL Abuse Cheatsheet

|Právo v BloodHoundu|Co znamená|Co si okamžitě pomyslet|
|---|---|---|
|ForceChangePassword|Můžu změnit heslo uživatele bez znalosti starého|Převzetí účtu|
|GenericAll|Plná kontrola nad objektem|"Téměř cokoliv"|
|GenericWrite|Můžu měnit atributy objektu|SPN, logon script, atributy|
|WriteDACL|Můžu upravit ACL|Přidám si silnější práva|
|WriteOwner|Můžu změnit vlastníka|Pak si dám WriteDACL|
|AddMember|Můžu přidávat členy do skupiny|Přidám sebe/ovládaný účet|
|Self|Můžu přidat sám sebe do skupiny|Escalace přes skupiny|
|DCSync|Můžu replikovat AD|Prakticky Domain Admin|

---

### Jak přemýšlet při enumeraci

Když v BloodHoundu vidíš:

```
User -> User
```

přemýšlej:

```
ForceChangePassword?GenericAll?GenericWrite?
```

Když vidíš:

```
User -> Group
```

přemýšlej:

```
AddMember?GenericAll?Self?
```

Když vidíš:

```
User -> Domain
```

přemýšlej:

```
WriteDACL?DCSync?
```

To jsou nejčastější cesty k DA.

### Tvůj konkrétní příklad

Ty jsi udělal:

```
wley ↓ ForceChangePassworddamundsen ↓ AddMemberHelp Desk Level 1 ↓ MemberOfInformation Technology ↓ GenericAlladunn
```

Až budeš dělat další HTB laby, snaž se vždy najít:

```
Kdo?  ↓Jaké právo?  ↓Na jaký objekt?  ↓Co mi to umožní?
```

Tím se z BloodHound grafu stane logický příběh místo změti čar.

Pro CPTS bych doporučil mít otevřené:
https://bloodhound.specterops.io/resources/edges/overview
https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces
https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/index.html