# PROCESS_SPEC_Writing

> **Versio:** 1.2  
> **Päivitetty:** 2025-11-25  
> **Tiedosto:** v1_2_PROCESS_SPEC_Writing.md  
> **Edellinen:** v1.1 (2025-11-25)

---

## Muutokset v1.1 → v1.2

| Muutos | Kuvaus |
|--------|--------|
| ✅ Syventävät tutkimuskysymykset | Ohjeistus syvällisempien kysymysten muodostamiseen |
| ✅ GitHub/Projects -selvennys | Miten GitHub-tiedostot näkyvät projektissa |

---

## GitHub-tiedostojen lukeminen (v1.2)

**⚠️ TÄRKEÄ:** GitHub-tiedostot EIVÄT näy `/mnt/project` -hakemistossa!

GitHub-integraatio toimii näin:
1. Käyttäjä lisää GitHub-repon Project Knowledge -osioon
2. Käyttäjä klikkaa "Sync now" päivittääkseen
3. Claude käyttää `project_knowledge_search` työkalua hakemaan sisältöä

```python
# Oikea tapa lukea GitHub-tiedostoja:
project_knowledge_search(query="SPEC_01 ClaudeService")

# EI toimi:
view("/mnt/project/docs/specs/SPEC_01.md")  # ❌ Tiedosto ei ole täällä
```

---

## Tutkimuskysymysten muodostaminen (v1.2)

### Kolme kysymystasoa

| Taso | Tyyppi | Esimerkki |
|------|--------|-----------|
| 1. **Perus** | Mitä? Miten? | "Miten streaming toteutetaan?" |
| 2. **Vertaileva** | Mikä parhaiten? | "Redis vs SQLite - kumpi parempi?" |
| 3. **Syventävä** | Miksi? Entä jos? | "Mitä tapahtuu edge caseissa?" |

### Syventävien kysymysten muodostaminen

```
PERUSKYSYMYS:
"Miten virheenkäsittely toteutetaan?"

SYVENTÄVÄT JATKOKYSYMYKSET:
├── "Mitä tapahtuu kun API palauttaa 200, mutta virhe tulee streamin keskellä?"
├── "Miten circuit breaker eroaa pelkästä retrystä?"
├── "Miksi exponential backoff eikä lineaarinen odotus?"
├── "Mitä muut projektit tekevät eri tavalla?"
└── "Mikä voisi mennä pieleen tuotannossa?"
```

### Syventävän tutkimuksen checklist

```
Jokaisen peruskysymyksen jälkeen, kysy:

- [ ] "Mitä edge caseja en ole miettinyt?"
- [ ] "Mikä voisi epäonnistua tuotannossa?"
- [ ] "Miten muut ovat ratkaisseet saman ongelman?"
- [ ] "Onko tutkimusta/dataa joka tukee/kumoaa oletukseni?"
- [ ] "Mitä en tiedä mitä en tiedä?" (unknown unknowns)
```

### Milloin käyttää syventävää tasoa?

- Moduuli on kriittinen (esim. API-wrapper)
- Päätöksellä on pitkäaikaiset vaikutukset
- Aihe on uusi tiimille
- MVP:n jälkeen muuttaminen olisi kallista

---

## Muu prosessi

Muu prosessi pysyy samana kuin v1.1:ssä:

1. **Konteksti** - Lue arkkitehtuuri, master, edellinen SPEC
2. **Tutkimus** - Hae best practices, tallenna RESEARCH_XX.md (+ syventävät kysymykset v1.2)
3. **Suunnittelu** - Tunnista primitiivi, API, rakenne
4. **Kirjoitus** - Kirjoita SPEC-dokumentti
5. **Käyttäjä-review** - Jussi arvioi
6. **Gemini-review** - Tekninen laadunvarmistus
7. **Viimeistely** - Korjaukset, tallennus
8. **Päivitykset** - MASTER, LOKI, INDEX

---

## Muistisäännöt (v1.2 lisäykset)

> **Kysy syventäviä kysymyksiä** - "Mitä voi mennä pieleen?" on tärkeämpi kuin "Miten tämä toimii?"

> **GitHub → project_knowledge_search** - Tiedostot eivät näy /mnt/project -hakemistossa.

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.2 | 2025-11-25 | Syventävät tutkimuskysymykset, GitHub-selvennys |
| 1.1 | 2025-11-25 | Välitallennus-ohje |
| 1.0 | 2025-11-25 | Ensimmäinen versio |
