## 📊 Relační databáze (Relational Databases / RDBMS)

Relační databáze jsou nejrozšířenějším typem databází. Pro organizaci dat používají pevné **schéma (schema)**, které definuje strukturu tabulek, sloupců a řádků.

### 🔑 Klíčové koncepty RDBMS:

- **Tabulky a relace:** Data jsou uložena v tabulkách (entitách), které jsou mezi sebou navzájem propojeny.
    
- **Klíče (Keys):**
    
    - **Primární klíč (Primary Key):** Unikátní identifikátor řádku (např. `id` uživatele v tabulce `users`).
        
    - **Cizí klíč (Foreign Key):** Odkaz na primární klíč jiné tabulky (např. `user_id` v tabulce `posts` odkazuje na konkrétního uživatele).
        
- **Výhody:** Vysoká rychlost, efektivita a spolehlivost při dotazování na komplexní strukturovaná data. Umožňuje získat kompletní související data z více tabulek pomocí jediného dotazu.
    
- **Příklady RDBMS:** MySQL, PostgreSQL, Oracle, Microsoft SQL Server, SQLite.
    

> ℹ️ K dotazování a manipulaci s relačními databázemi se používá výhradně jazyk **SQL**, který je cílem **SQL Injection (SQLi)**.

## 🌐 Nerelační databáze (NoSQL Databases)

NoSQL databáze nepoužívají tradiční strukturu tabulek, řádků, sloupců ani relačních klíčů. Jsou navrženy pro práci se nestrukturovanými nebo proměnlivými datovými sadami a vynikají vysokou flexibilitou a škálovatelností.

### 🗂️ 4 základní modely ukládání v NoSQL:

1. **Klíč-Hodnota (Key-Value):** Data jsou uložena jako dvojice klíče a hodnoty (často ve formátu JSON nebo XML, podobně jako slovníky v Pythonu/PHP).
    
2. **Dokumentové (Document-Based):** Ukládání celých dokumentů (např. JSON/BSON).
    
3. **Sloupcové (Wide-Column):** Ukládání dat po sloupcích s proměnlivou strukturou.
    
4. **Grafové (Graph):** Zaměřují se na uzel a vztahy mezi nimi (vhodné pro sociální sítě).
    

#### Příklad Key-Value v JSON:

JSON

```
{
  "100001": {
    "date": "01-01-2021",
    "content": "Welcome to this web application."
  },
  "100002": {
    "date": "02-01-2021",
    "content": "This is the first post on this web app."
  }
}
```

- **Příklady NoSQL:** MongoDB, Redis, Cassandra, CouchDB.
    

> ⚠️ **Bezpečnostní poznámka:** Útoky na NoSQL databáze se označují jako **NoSQL Injection**. Princip fungování i syntaxe se od tradiční SQL Injection zcela liší.