# OWASP Juice Shop – SQL Injection (Obcházení přihlášení)

Tento repozitář dokumentuje bezpečnostní zranitelnost typu **SQL Injection (SQLi)**
v úmyslně zranitelné webové aplikaci **OWASP Juice Shop**.

> ⚠️ Projekt slouží **výhradně pro studijní a výukové účely**.  
> Testování probíhalo pouze na aplikaci, která je k tomuto účelu určena.

---

## 📌 Testovaná aplikace
- **Název:** OWASP Juice Shop
- **Typ:** Záměrně zranitelná webová aplikace
- **Postižená část:** Přihlašovací formulář
- **Výzva:** Password Strength / Authentication Bypass

---

## ❓ Co je SQL Injection
**SQL Injection** je bezpečnostní zranitelnost, při které aplikace **nekontroluje
nebo špatně zpracovává vstup od uživatele**, který je následně použit
v databázovém SQL dotazu.

Pokud je uživatelský vstup přímo vložen do SQL dotazu bez správného ošetření,
může útočník:
- změnit logiku dotazu
- obejít autentizaci
- získat neoprávněný přístup k datům
- manipulovat nebo mazat data v databázi

---

## 🧠 Jak SQL Injection funguje (obecně)
Webové aplikace často používají databáze pro:
- přihlašování uživatelů
- ukládání hesel
- práci s objednávkami a účty

Pokud aplikace:
- skládá SQL dotazy dynamicky
- a zároveň důvěřuje uživatelskému vstupu  

může dojít k situaci, kdy vstup uživatele **ovlivní chování databázového dotazu**.
To může vést například k **obejití přihlášení bez znalosti hesla**.

---

## 🛡️ Přehled zranitelnosti
- **Typ:** SQL Injection
- **OWASP Top 10:** A03 – Injection
- **Postižená komponenta:** Autentizace (login)
- **Dopad:** Neoprávněný přístup k administrátorskému účtu

Aplikace nedostatečně validuje vstup při přihlašování, což umožňuje změnit
vyhodnocení autentizační logiky na straně databáze.

---

## 📸 Důkaz úspěšnosti (Proof of Concept)
Po úspěšném zneužití zranitelnosti byl získán přístup k účtu administrátora.

Dokumentace obsahuje:
- potvrzení úspěšného splnění výzvy
- přístup k administrátorskému účtu

Viz složka `/screenshots`.

---

## 🚨 Bezpečnostní dopad
V reálné aplikaci by tato zranitelnost mohla vést k:
- eskalaci oprávnění (běžný uživatel → administrátor)
- kompromitaci uživatelských účtů
- úniku citlivých dat
- úplnému převzetí aplikace

---

## 🛠️ Doporučená bezpečnostní opatření
- používat **parametrizované dotazy (prepared statements)**
- nepoužívat dynamicky sestavované SQL dotazy
- důsledně validovat vstup na straně serveru
- používat databázové účty s minimálními oprávněními
- implementovat bezpečné autentizační mechanismy

---

## 📚 Upozornění
Tento repozitář **neobsahuje konkrétní exploitační payloady ani postupy útoku**.
Slouží výhradně k pochopení principu SQL Injection
v rámci výukové aplikace OWASP Juice Shop.
