# PROCESS_Database_Management

> **Versio:** 1.0  
> **Päivitetty:** 2025-05-22  
> **Tiedosto:** v1.0_PROCESS_Database_Management.md

---

## Periaate

Tietokantamallia päivitetään **erissä**, ei jokaisen löydöksen jälkeen. Löydökset kirjataan välittömästi muutoslokiin.

## Tiedostot

| Tiedosto | Tarkoitus |
|----------|-----------|
| `DATABASE_MODEL_v*.md` | Virallinen tietokantamalli (versionoitu) |
| `DATABASE_CHANGELOG.md` | Löydökset ja päivityshistoria |

## Prosessi

### 1. Löydös dokumentista/API:sta
```
→ Kirjaa heti DATABASE_CHANGELOG.md:hen kohtaan "Odottavat muutokset"
→ Merkitse lähde (dokumentti, API, keskustelu)
→ Älä päivitä tietokantamallia vielä
```

### 2. Päivitys (erässä)
```
Milloin:
- Yhden dokumentin/API:n analysointi valmis
- 5-10 muutosta kertynyt
- Ennen isoa virstanpylvästä (UI, koodaus alkaa)

Toimenpiteet:
1. Käy läpi odottavat muutokset
2. Päivitä DATABASE_MODEL → kasvata versionumero
3. Siirrä muutokset lokissa kohtaan "Toteutetut"
4. Kirjoita lyhyt yhteenveto muutoksista
```

### 3. Versionumerointi

- **v1.0, v2.0...** = Merkittävä rakennemuutos (uusi moduuli)
- **v1.1, v1.2...** = Täydennykset ja korjaukset

## Muutoslokin rakenne

```markdown
## Odottavat muutokset
- [ ] Muutos X (lähde: dokumentti Y)

## Versio X.X (pvm)
### Lisätty
- Kenttä Z tauluun W
### Muutettu
- ...
### Poistettu
- ...
```

## Muistisääntö

> **Löydös → Loki heti. Malli → Päivitä erässä.**

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.0 | 2025-05-22 | Ensimmäinen versio, lisätty versionumerointi |

---

*Dokumentti on osa AI-taloushallintojärjestelmän prosessiohjeistusta.*
