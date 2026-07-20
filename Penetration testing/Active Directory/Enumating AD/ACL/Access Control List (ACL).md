## 🧠 Co jsou ACL / ACE v AD

- **ACL (Access Control List)** = seznam oprávnění k objektu v AD
- **ACE (Access Control Entry)** = jedna konkrétní položka v ACL
- Každý objekt (user, group, computer) má ACL
- ACL říká:
    - kdo má přístup
    - jaký typ přístupu má

---

## 🔐 Typy ACL

- **DACL** → řeší _kdo má/ nemá přístup_
- **SACL** → řeší _logování (audit přístupů)_

---

## ⚡ Jak se ACL vyhodnocuje

- čte se **shora dolů**
- rozhoduje:
    - explicitní DENY
    - nebo ALLOW

---

## 🎯 Nejdůležitější ACE (pro útoky v AD)

### 🔥 Nejvíc exploitable práva:

- **GenericAll**
    - úplná kontrola nad objektem
    - reset hesla / změna členství / LAPS čtení
- **GenericWrite**
    - zápis do atributů
    - → umožní např. **Kerberoasting (SPN abuse)**
- **WriteDACL**
    - můžeš si přidat vlastní práva (full control)
- **WriteOwner**
    - změníš vlastníka objektu → pak si přidáš práva

---

## 🧨 Speciální „high impact“ práva

- **ForceChangePassword**
    - reset hesla bez znalosti starého
- **AddMembers / AddSelf**
    - přidání do privilegovaných skupin
- **AllExtendedRights**
    - často zahrnuje reset hesla + group management
- **ReadGMSAPassword**
    - čtení hesel service accounts (gMSA)

---

## 💥 Nejčastější útoky z ACL

- 🔺 Privilege escalation (získání admin práv)
- ↔️ Lateral movement (pohyb v doméně)
- 🔁 Persistence (udržení přístupu)

---

## 🧪 Klíčové praktické insighty

- ACL chyby jsou **časté a dlouho neviditelné**
- nástroje:
    - BloodHound (mapování)
    - PowerView (exploiting)
- často vedou k:
    - resetu hesel
    - přidání do Domain Admin cest
    - Kerberoasting
    - LAPS dumpu

---

## 🧷 Jedna věta na zapamatování

> **Nejvíc problémů v AD vzniká z GenericAll / GenericWrite / WriteDACL – tyhle tři jsou klíč k většině eskalací.**