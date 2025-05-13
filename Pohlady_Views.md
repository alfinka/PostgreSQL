
# 📘 PostgreSQL: Pohľady a Materiálové pohľady

Materiály k online kurzom PostgreSQL so zameraním na VIEW a MATERIALIZED VIEW.

### B1 [👁️ Zoznam VIEW príkazov a odporúčaní](#zoznam-view-prikazov)
### B2 [🗃️ Porovnanie TABLE vs VIEW vs MATERIALIZED VIEW](#porovnanie-view-materialized)
### B3 [🔐 RLS a pohľady: bezpečnostné modely](#rls-a-pohlady)

---

<a name="zoznam-view-prikazov"></a>
## 👁️ Zoznam VIEW príkazov a odporúčaní

```sql
CREATE [ OR REPLACE ] [ TEMP | TEMPORARY ] [ RECURSIVE ] VIEW nazov_view [ ( stlpec1, stlpec2, ... ) ]
[ WITH ( security_barrier = true ) ]
AS SELECT ...
[ WITH [ CASCADED | LOCAL ] CHECK OPTION ];
```

**Vysvetlenie volieb:**

| Parameter                  | Popis                                                                 |
|---------------------------|------------------------------------------------------------------------|
| `OR REPLACE`              | Prepíše existujúci pohľad bez potreby DROP                          |
| `TEMP` / `TEMPORARY`      | Vytvorí dočasný pohľad pre reláciu                                |
| `RECURSIVE`               | Povolenie rekurzívnych pohľadov (od v14+)                                |
| `WITH (security_barrier)` | Zvýšená bezpečnosť proti SQL injection                          |
| `CHECK OPTION`            | Obmedzenie INSERT/UPDATE len na riadky, ktoré vyhovujú podmienke pohľadu   |

---

<a name="porovnanie-view-materialized"></a>
## 🗃️ Porovnanie: `TABLE` vs `VIEW` vs `MATERIALIZED VIEW`

| Kritérium                   | TABLE (`CREATE TABLE`)                         | VIEW (`CREATE VIEW`)                                      | MATERIALIZED VIEW (`CREATE MATERIALIZED VIEW`)               |
|-----------------------------|------------------------------------------------|------------------------------------------------------------|---------------------------------------------------------------|
| **Ukladanie dát**           | ✅ Áno – fyzicky uložené                        | ❌ Nie – dynamický dotaz                                   | ✅ Áno – výsledky dotazu sa ukladajú                          |
| **Aktualizácia dát**        | ✅ INSERT/UPDATE/DELETE                         | ⚠️ Len pre jednoduché pohľady                              | ❌ Nie – len `REFRESH MATERIALIZED VIEW`                      |
| **Aktuálnosť dát**          | ✅ Reálne časové                               | ✅ Pri každom dotaze                                       | ❌ Závislé od času posledného refreshu                        |
| **Výkon (performance)**     | ✅ Vysoký, optimalizované                       | ⚠️ Závisí od zložitosti dotazu                             | ✅ Vysoký pre SELECT, pomalý pre REFRESH                      |
| **Indexy**                  | ✅ Áno                                          | ❌ Nie                                                     | ✅ Áno – len na uložené dáta                                  |
| **Primárne/kľúče**          | ✅ Áno                                          | ❌ Nie                                                     | ❌ Nie, len cez indexy                                        |
| **CRUD operácie**           | ✅ Plne podporované                            | ⚠️ Často len SELECT                                        | ❌ Len SELECT                                                 |
| **Podpora RLS**             | ✅ Áno                                          | ⚠️ Nie priamo – cez podkladové tabuľky                     | ❌ Nie                                                        |
| **Vhodné použitie**         | Trvalé údaje, riadenie referenčnej integrity   | Zjednodušenie dotazov, pohľad na podmnožinu dát            | Predpočítané reporty, analýzy, dashboardy                    |
| **Automatická aktualizácia**| ✅ Áno                                          | ✅ Áno – vždy pri SELECT                                   | ❌ Nie – treba `REFRESH` ručne alebo cez cron/job             |

---


# 🧩 Kedy použiť TABLE, VIEW a MATERIALIZED VIEW

## 🧩 Kedy použiť TABUĽKU
- Potrebovať trvalé uloženie dát
- Vyžadovať indexy, kľúče a výkonné dopyty
- Vykonávať zmeny v údajoch (`INSERT`, `UPDATE`, `DELETE`)

## 🧩 Kedy použiť POHĽAD
- Zjednodušiť zložitý dopyt
- Obmedziť viditeľnosť stĺpcov alebo riadkov
- Vytvoriť vrstvu nad databázou pre reporting alebo bezpečnosť
- Použiť Row-Level Security (RLS) alebo skryť citlivé údaje

## 🧩 Kedy použiť MATERIALIZED VIEW
### ✅ Odporúča sa, ak:
- Potrebovať zrýchliť opakujúce sa zložité dopyty
- Mať potrebu statického snapshotu dát (napr. denne aktualizovaného)
- Použiť na dashboardy, reporty, analytické prehľady

### ⚠️ Pozor:
- Dáta sa **neaktualizujú automaticky** – treba ručne alebo pomocou `cron`/`job`:

```sql
REFRESH MATERIALIZED VIEW nazov_view;
```

---

# 🧾 Vysvetlenie nastavení pohľadu v pgAdmin (sekcia Definition)

## 1. **Security barrier?**
- Prepínač zap/vyp
- Ak je **zapnutý**, PostgreSQL **najprv aplikuje WHERE** v pohľade, a **až potom** spája s dopytom používateľa
- Chráni pred SQL injection (napr. pri RLS)

```sql
CREATE VIEW moja_view WITH (security_barrier = true) AS ...;
```

## 2. **Security invoker?**
- Ak je **zapnutý**, pohľad sa vykonáva ako **používateľ**, ktorý ho volá
- Ak je **vypnutý** (default), vykonáva sa ako **vlastník** (definer)

```sql
CREATE VIEW moja_view SECURITY INVOKER AS ...;
```

## 3. **Check options**
- Rozbaľovacie menu (CASCADED / LOCAL)
- Určuje, či INSERT/UPDATE musia **spĺňať podmienky pohľadu**

```sql
CREATE VIEW aktivni_zamestnanci AS
SELECT * FROM zamestnanci WHERE aktivny = TRUE
WITH CHECK OPTION;
```

## 4. **Chyba 'Name' cannot be empty**
- Znamená, že v sekcii **General** nie je vyplnený **názov pohľadu**
- Bez toho nie je možné uložiť objekt

## 5. **Ovládacie tlačidlá**
- 🛈 Informácia: pomocník
- ❓ Help: dokumentácia
- ❌ Close: zavrie okno
- 🔄 Reset: vymaže zmeny
- 💾 Save: uloží definíciu (len ak je vyplnený "Name")

---
![view-pgadmin-1](https://github.com/user-attachments/assets/e2275b55-c342-4ede-82f5-5ab57609db03)

# 🔐 Vysvetlenie nastavení v pgAdmin (sekcia Security)

## **Privileges (Oprávnenia)**
- Spravuje prístup k pohľadu

| Stĺpec   | Popis                                      |
|-----------|---------------------------------------------|
| Grantee   | Kto získa oprávnenie (napr. `reporting_users`) |
| Privilege | SELECT, INSERT, UPDATE, DELETE              |
| Grantor   | Vlastník pohľadu                         |

```sql
GRANT SELECT ON VIEW bezpecne_udaje TO reporting_users;
```

## **Security labels**
- Použitie MAC/SELinux bezpečnostných značiek (napr. `sepgsql`)

```sql
SECURITY LABEL FOR sepgsql ON VIEW moja_view IS 'system_u:object_r:sepgsql_view_t:s0';
```

➡️ Bez MAC rozšírenia sa táto sekcia nevyužije

## **Chyba 'Name' cannot be empty**
- Potrebné vyplniť názov pohľadu v sekcii **General**

![view-pgadmin-2](https://github.com/user-attachments/assets/20be8e43-e7d1-420e-bfe2-e884286c4926)

<a name="rls-a-pohlady"></a>
## 🔐 RLS a pohľady: bezpečnostné modely

**1. RLS (Row-Level Security)** funguje len nad TABUĽKOU, ale je možné ho nepriamo použiť cez VIEW:

- Vytvorenie pohľadu s podmienkou:
```sql
CREATE VIEW moje_dokumenty AS
SELECT * FROM dokumenty WHERE vlastnik = current_user;
```

- Priradenie práv len k pohľadu:
```sql
GRANT SELECT ON moje_dokumenty TO analyzator;
```

- Vlastník pohľadu by mal použiť `WITH (security_barrier = true)` pre zamedzenie injekcie.

---
**2. Materializovaný pohľad + obmedzenie prístupu:**
- Vhodný pre ťažké analytické dopyty
- Neobsahuje dynamické RLS
- Vhodné kombinovať s `REFRESH MATERIALIZED VIEW CONCURRENTLY` a prístupovými rolami

---
**Praktické odporúčanie:**

- Použiť **VIEW** na zjednodušenie SELECT dopytov, maskovanie alebo filtráciu
- Použiť **MATERIALIZED VIEW**, ak je SELECT drahý a dáta sa menia zriedka
- Na **RLS použite radšej priamo TABUĽKY**, pohľady len ako prezentačné vrstvy
