Tento modul se zaměřuje na **detekci**, **exploataci** a **prevenci** následujících vektorů útoků:

### 🛡️ Přehled probíraných webových útoků

| **Útok**                                       | **Podstata zranitelnosti**                                                                     | **Typický dopad / Riziko**                                                                             |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| **HTTP Verb Tampering**                        | Nedostatečná konfigurace autorizace na základě HTTP metod (`GET`, `POST`, `PUT`, `HEAD` atd.). | Obcházení autentizačních a autorizačních filtrů, bypass WAFu.                                          |
| **IDOR** _(Insecure Direct Object References)_ | Chybějící kontrola oprávnění na backendu při přímém přístupu k objektům přes jejich ID.        | Neoprávněný přístup k cizím datům, souborům a účtům jiných uživatelů.                                  |
| **XXE Injection** _(XML External Entity)_      | Zastaralé nebo špatně konfigurované XML parsery zpracovávající vnější entity.                  | Čtení citlivých lokálních souborů serveru, SSRF, únik přihlašovacích údajů, v některých případech RCE. |

### 🎯 Hlavní cíl modulu

Pochopit, jak útočníci zneužívají logické i technické nedostatky webových aplikací k získání neoprávněného přístupu k datům či samotnému serveru, a jak tyto chyby správně ošetřit v kódu i konfiguraci webového serveru.