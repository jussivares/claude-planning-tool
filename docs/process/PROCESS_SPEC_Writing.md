# PROCESS_SPEC_Writing

> **Versio:** 1.5  
> **Päivitetty:** 2025-11-26  
> **Tiedosto:** v1_5_PROCESS_SPEC_Writing.md  
> **Tarkoitus:** SPEC ja TECH_SPEC -dokumenttien kirjoitusprosessi alusta loppuun  
> **Liittyy:** PROCESS_Document_Updates.md (jatkaa tallennuksesta eteenpäin)  
> **Edellinen:** v1.4 (2025-11-25)

---

## Muutokset v1.4 → v1.5

| Muutos | Kuvaus |
|--------|--------|
| ✅ **Phase 0 lisätty** | Market Research and Idea Evaluation/Evolution (valinnainen) |
| ✅ Prosessikaavio päivitetty | 10-vaiheinen → 11-vaiheinen |
| ✅ Numerointi päivitetty | Phase 0 + 10 vaihetta |

---

## Prosessin yleiskuva (v1.5)

```
┌─────────────────────────────────────────────────────────────────┐
│              SPEC → TECH_SPEC KIRJOITUSPROSESSI                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ═══════════════ LIIKETOIMINTAYMMÄRRYS (VALINNAINEN) ══════════ │
│                                                                 │
│  0. MARKET RESEARCH  Kilpailijat, käyttäjätarpeet, best practices │
│         │            → MARKET_RESEARCH_[Project].md             │
│         │            → VISION_DOC_[Project].md                  │
│         │            → DEVELOPMENT_ROADMAP (optional)           │
│         │                                                       │
│         │            [Katso: PROCESS_Market_Research.md]        │
│         │                                                       │
│         ▼                                                       │
│  ═══════════════ TOIMINNALLINEN MÄÄRITTELY ═══════════════     │
│                                                                 │
│  1. KONTEKSTI        Lue arkkitehtuuri, master, edellinen SPEC  │
│         │                                                       │
│         ▼                                                       │
│  2. RESEARCH         Toiminnallinen tutkimus (patterns, teoria) │
│         │            → Tallenna RESEARCH_XX.md                  │
│         ▼                                                       │
│  3. SUUNNITTELU      Tunnista primitiivi, API, rakenne          │
│         │                                                       │
│         ▼                                                       │
│  4. SPEC-KIRJOITUS   Kirjoita SPEC-dokumentti                   │
│         │            → Käytä 🟢/🟡/🔵 vaiheistusta              │
│         ▼                                                       │
│  5. KÄYTTÄJÄ-REVIEW  Jussi arvioi, vastaa avoimiin kysymyksiin  │
│         │                                                       │
│         ▼                                                       │
│  6. GEMINI-REVIEW    Toiminnallinen laadunvarmistus             │
│         │                                                       │
│         ▼                                                       │
│  7. SPEC-VIIMEISTELY Korjaukset, tallennus GitHubiin            │
│         │                                                       │
│         │                                                       │
│  ═══════════════ TEKNINEN MÄÄRITTELY ═══════════════════       │
│         │                                                       │
│         ▼                                                       │
│  8. TECH_RESEARCH    Teknologiavalinnat (kirjastot, toteutus)   │
│         │            → Tallenna TECH_RESEARCH_XX.md             │
│         ▼                                                       │
│  9. TECH_SPEC        Tekninen määrittely ja toteutus            │
│         │            → Tallenna TECH_SPEC_XX.md                 │
│         ▼                                                       │
│  10. GEMINI TECH     Tekninen laadunvarmistus                   │
│         │                                                       │
│         ▼                                                       │
│  11. PÄIVITYKSET     Päivitä MASTER, LOKI, INDEX                │
│         │                                                       │
│         ▼                                                       │
│  → Implementation (koodaus alkaa)                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Phase 0: Market Research (valinnainen)

### Milloin käyttää?

Phase 0 on **valinnainen** mutta hyödyllinen kun:
- ✅ Aloitetaan **uusi projekti** (startup, SaaS)
- ✅ Halutaan ymmärtää **markkinaa ja kilpailijoita**
- ✅ Tarvitaan **Vision Doc** sidosryhmille
- ✅ Halutaan **priorisoida MVP** dataan perustuen

Phase 0 kannattaa **ohittaa** kun:
- ❌ Projekti on **hyvin määritelty replica**
- ❌ Kyseessä **proof of concept** tai lyhyt projekti (<2vk)
- ❌ **Tekninen kirjasto** jossa liiketoimintakonteksti ei relevantti

### Prosessi lyhyesti

1. **Claude kysyy:** "Aloitetaanko Market Research -vaiheella?"
2. **Käyttäjä päättää:** Kyllä / Ei
3. **Jos kyllä →** Katso täysi ohjeistus: **PROCESS_Market_Research.md**

### Tulokset

```
MARKET_RESEARCH_[Project].md  → Kilpailijat, käyttäjätarpeet
VISION_DOC_[Project].md        → Problem, Solution, MVP-scope
DEVELOPMENT_ROADMAP (optional) → Aikataulu, prioriteetit
```

**Token-budjetti:** ~20K tokenia, ~75 minuuttia

**Tallennuspaikka:** GitHub (aktivoi vain tarvittaessa)

---

## Phase 1: Konteksti

### Valmistelu

**Lue AINA ennen aloitusta:**

1. **ARCHITECTURE_OVERVIEW** - Ymmärrä projektin rakenne
2. **MASTER_FUNCTIONAL** - Katso moduulien statukset
3. **systems-architecture skill** - Muista black box -periaatteet
4. **SYNTEESI_Moderni_Suunnittelu** - TECH_SPEC -malli
5. **Edellinen SPEC** - Opi edellisestä moduulista

**Jos Phase 0 tehty:**
- **VISION_DOC** - Ymmärrä projektin visio ja MVP-priorisointi
- **MARKET_RESEARCH** - Kilpailijoiden best practices

---

## Phase 2: RESEARCH (toiminnallinen tutkimus)

### Tavoite

Vastata kysymykseen: **"Miten tämä moduuli pitäisi TOIMINNALLISESTI suunnitella?"**

### Tutkimuskysymykset

Claude laatii kysymykset jotka ohjaavat tutkimusta:

```
TOIMINNALLISET KYSYMYKSET:

1. Mikä on moduulin primitiivi?
   → Perusyksikkö jota moduuli käsittelee

2. Mitä vastaavia moduuleja muissa järjestelmissä?
   → Kilpailijoiden toteutukset, best practices

3. Mitkä ovat kriittiset edge caset?
   → Mitä voi mennä pieleen?

4. Miten tämä integroituu muihin moduuleihin?
   → Black box -rajapinnat

5. Mitä standardeja tai protokollia noudatetaan?
   → Esim. OAuth, JSON:API, jne.
```

**Huom:** Jos Phase 0 tehty, MARKET_RESEARCH sisältää jo osan vastauksista!

### Tutkimuksen laajuus

| Taso | Kesto | Lähteitä | Syvyys |
|------|-------|----------|--------|
| **Nopea** | 15-20 min | 3-5 | Pinnallinen |
| **Keskitaso** | 30-45 min | 5-10 | Perusteellinen |
| **Syvällinen** | 60-90 min | 10+ | Akateeminen |

**Suositus:** Keskitaso ensimmäisellä kerralla.

### Web Search -strategia

```
VAIHE 1: Yleiskatsaus (2-3 hakua)
- "[moduuli] architecture patterns"
- "[moduuli] best practices"

VAIHE 2: Kilpailijat (3-5 hakua)
- "[kilpailija A] [moduuli] implementation"
- "[kilpailija B] how to [moduuli]"

VAIHE 3: Ongelmat ja ratkaisut (2-3 hakua)
- "[moduuli] common pitfalls"
- "[moduuli] edge cases"

VAIHE 4: Standardit (1-2 hakua)
- "[moduuli] standards protocols"
```

### Dokumentointi

Tallenna löydökset → **RESEARCH_XX_[Nimi].md**

```markdown
# RESEARCH_02_Memory_Architecture

> **Versio:** 1.0
> **Tutkimuspäivä:** 2025-11-26

## Executive Summary
[2-3 kappaletta keskeiset löydökset]

## Tutkimuskysymykset ja vastaukset

### 1. Mikä on muistin primitiivi?
[Vastaus + lähteet]

### 2. Mitä muistiarkkitehtuureja kilpailijoilla?
[Analyysi]

## Best Practices
[Lista opituista asioista]

## Suositukset SPECille
[Mitä pitää huomioida SPEC:ssä?]

---
*Lähteet: [Lista]*
```

**Tallennuspaikka:** GitHub `docs/research/`

---

## Phase 3: Suunnittelu

### Systems Architecture Checklist

**Lue AINA:** `/mnt/skills/user/systems-architecture/SKILL.md`

```
Analysis Checklist:

[ ] Primitiivi tunnistettu?
    → Mikä on järjestelmän perusyksikkö?

[ ] Black box -rajat selkeät?
    → Onko API dokumentoitu? Sisäiset yksityiskohdat piilossa?

[ ] Ulkoiset riippuvuudet wrapattu?
    → Claude API → ClaudeService wrapper

[ ] Yksi omistaja per moduuli?
    → Voiko yksi henkilö ymmärtää koko moduulin?

[ ] Voidaanko kirjoittaa uudelleen?
    → Voisiko joku kirjoittaa moduulin uudelleen pelkän API:n perusteella?

[ ] Toimiiko 10x vaatimuksilla?
    → Skaalautuuko arkkitehtuuri?
```

---

## Phase 4: SPEC-kirjoitus

### Rakenne (päivitetty v1.4)

```markdown
# SPEC_XX_Moduulin_Nimi

> **Versio:** 1.0
> **Päivitetty:** 2025-XX-XX

## 1. Yleiskatsaus

[Kuvaus, tavoite, scope]

## 2. Toiminnalliset vaatimukset

### 2.1 Ominaisuus A 🟢

[Kuvaus]

**Prioriteetti:** MVP (🟢)
**Riippuvuudet:** ModuuliB, ModuuliC

### 2.2 Ominaisuus B 🟡

[Kuvaus]

**Prioriteetti:** Phase 2 (🟡)

### 2.3 Ominaisuus C 🔵

[Kuvaus]

**Prioriteetti:** Phase 3 (🔵)

## 3. API-määrittely (Black Box)

[Julkinen rajapinta - muut moduulit näkevät vain tämän]

## 4. Edge Cases

[Mitä voi mennä pieleen?]

## 5. Turvallisuus

[Uhkamallit, suojaukset]

## 6. Suorituskyky

[SLA:t, bottleneckit]

## 7. Vaiheistus yhteenveto

| Prioriteetti | Ominaisuudet | Perustelu |
|--------------|--------------|-----------|
| 🟢 MVP | 2.1, 2.4 | Ydinominaisuudet |
| 🟡 Phase 2 | 2.2, 2.5 | Parantavat UX |
| 🔵 Phase 3 | 2.3, 2.6 | Nice-to-have |

## 8. Avoimet kysymykset

[Lista kysymyksiä käyttäjälle - vaihtoehdot + ehdotus]
```

### Vaiheistussymbolit (v1.4)

```
🟢 MVP (Phase 1)        - Kriittinen, ilman ei toimi
🟡 Phase 2              - Tärkeä, mutta ei blokkeri
🔵 Phase 3              - Nice-to-have, voidaan viivästyttää
```

**Tallennuspaikka:** GitHub `docs/specs/`

---

## Phase 5: Käyttäjä-review

### Claude kysyy:

```
"SPEC_XX v1.0 valmis lukemiseen.

Tarkista erityisesti:
1. Onko primitiivi oikea?
2. Puuttuuko kriittinen ominaisuus MVP:stä?
3. Onko jokin ominaisuus väärin priorisoitu?
4. API-määrittely selkeä ja riittävä?

AVOIMET KYSYMYKSET (vastaa näihin):
[Lista kysymyksiä + vaihtoehdot + Clauden ehdotus]
"
```

### Käyttäjä vastaa:

```
Vaihtoehto A: Hyväksyy sellaisenaan
   → Siirry Phase 6 (Gemini review)

Vaihtoehto B: Pyytää muutoksia
   → Claude päivittää → uusi käyttäjä-review

Vaihtoehto C: Vastaa avoimiin kysymyksiin
   → Claude päivittää SPEC → uusi käyttäjä-review
```

---

## Phase 6: Gemini-review (toiminnallinen)

### Tarkistuslista

**Gemini tarkistaa:**

```
TOIMINNALLINEN LAATU:

[ ] Onko primitiivi järkevä ja johdonmukainen?
[ ] Ovatko toiminnalliset vaatimukset täsmälliset?
[ ] API-määrittely täydellinen (input/output/errors)?
[ ] Edge caset kattavat?
[ ] Turvallisuusriskit tunnistettu?
[ ] Vaiheistus looginen (🟢/🟡/🔵)?
[ ] Riippuvuudet muihin moduuleihin selvät?

PUUTTEET JA RISTIRIIDAT:

[ ] Onko ristiriitoja muiden moduulien kanssa?
[ ] Puuttuuko joku kriittinen ominaisuus?
[ ] Onko jokin määrittely liian epämääräinen?

PARANNUSEHDOTUKSET:

[ ] Voiko API:a yksinkertaistaa?
[ ] Pitäisikö joku ominaisuus jakaa useampaan moduuliin?
[ ] Onko jokin edge case huomiotta?
```

### Gemini-promptin rakenne

```
Olet software architect. Arvioi tämä SPEC:

[SPEC-dokumentti]

Käytä seuraavaa tarkistuslistaa:
[Tarkistuslista yllä]

Anna palaute muodossa:
1. ✅ Hyvät puolet
2. ⚠️ Huolenaiheet
3. 🔧 Konkreettiset korjausehdotukset (jos tarpeen)

Ole kriittinen mutta rakentava.
```

---

## Phase 7: SPEC-viimeistely

### Claude päivittää SPECin

Gemini-reviewin perusteella:
```
SPEC_XX v1.0 → v1.1
- Korjaus 1: [Geminin havainto]
- Korjaus 2: [...]
- Lisäys: [...]
```

### Tallennus GitHubiin

```
Tiedosto: docs/specs/SPEC_XX_Moduuli_Nimi.md
Commit message: "Add SPEC_XX v1.1: [Moduuli] specification (post-Gemini review)"
```

---

## Phase 8: TECH_RESEARCH (tekninen tutkimus)

### Tavoite

Vastata kysymykseen: **"MILLÄ teknologioilla ja kirjastoilla tämä toteutetaan?"**

### Tutkimuskysymykset

```
TEKNISET KYSYMYKSET:

1. Mikä ohjelmointikieli? (Python / TypeScript / jne.)
2. Mitkä kirjastot ja frameworkit?
   → Anthropic SDK, FastAPI, React, jne.
3. Miten data tallennetaan?
   → SQLite, PostgreSQL, Markdown, jne.
4. Mitkä ulkoiset palvelut?
   → Anthropic API, Ollama, jne.
5. Miten testataan?
   → pytest, jest, Playwright, jne.
```

### Dokumentointi

Tallenna → **TECH_RESEARCH_XX_[Moduuli].md**

```markdown
# TECH_RESEARCH_02_Memory_Service

## Teknologiavalinnat

| Komponentti | Vaihtoehto A | Vaihtoehto B | VALINTA |
|-------------|--------------|--------------|---------|
| Tietokanta | SQLite | PostgreSQL | **SQLite** |
| ORM | SQLAlchemy | Raw SQL | **Raw SQL** |
| Embedding | OpenAI | Ollama | **Ollama** |

## Perustelut

[Miksi valittiin näin?]

## POC-koodit

[Pienet testikoodit vahvistavat valinnat]
```

**Katso:** PROCESS_Market_Research.md sisältää osan teknologiatrendeistä!

**Tallennuspaikka:** GitHub `docs/research/`

---

## Phase 9: TECH_SPEC-kirjoitus

### Rakenne

```markdown
# TECH_SPEC_XX_Moduuli_Nimi

> **Versio:** 1.0

## 1. Tekninen yleiskatsaus

[Arkkitehtuuri, teknologiat]

## 2. API-toteutus (yksityiskohtainen)

### 2.1 Endpoint: create_memory()

```python
def create_memory(
    content: str,
    category: MemoryCategory,
    metadata: dict[str, Any]
) -> Memory:
    """
    Luo uusi muistimuistiinpano.
    
    Args:
        content: Muistin sisältö
        category: DECISION | SUMMARY | CONTEXT
        metadata: Lisätiedot
        
    Returns:
        Memory-objekti
        
    Raises:
        ValueError: Jos content tyhjä
    """
```

[Toteutuslogiikka, SQL-kyselyt]

## 3. Tietomallit

```python
class Memory(BaseModel):
    id: UUID
    content: str
    category: MemoryCategory
    created_at: datetime
    salience: float
```

## 4. Tietokantaskeema

```sql
CREATE TABLE memories (
    id TEXT PRIMARY KEY,
    content TEXT NOT NULL,
    category TEXT NOT NULL,
    created_at TEXT NOT NULL,
    salience REAL NOT NULL
);
```

## 5. Testausstrategia

[Unit tests, integration tests]

## 6. Error Handling

[Virhetilanteet, retry-logiikka]

## 7. Deployment

[Konfiguraatio, ympäristömuuttujat]
```

**Katso malli:** SYNTEESI_Moderni_Suunnittelu.md

**Tallennuspaikka:** GitHub `docs/tech-specs/`

---

## Phase 10: Gemini TECH-review

### Tarkistuslista

```
TEKNINEN LAATU:

[ ] Koodi noudattaa best practiceja?
[ ] Error handling kattava?
[ ] Testauskattavuus riittävä?
[ ] Dokumentaatio selkeä?
[ ] Type hints käytössä (Python/TypeScript)?
[ ] Security considerations huomioitu?

TOTEUTETTAVUUS:

[ ] Onko toteutus realistinen?
[ ] Skaalautuuko ratkaisu?
[ ] Performance bottleneckit tunnistettu?

YLLÄPIDETTÄVYYS:

[ ] Onko koodi luettavaa?
[ ] Logging riittävä?
[ ] Configuration management järkevä?
```

---

## Phase 11: Päivitykset

### Dokumentit päivitettävä:

```
1. MASTER_FUNCTIONAL
   → Päivitä moduulin status (SPEC valmis / TECH_SPEC valmis)

2. KEHITYSLOKI
   → Kirjaa session tulokset, päätökset, seuraavat askeleet

3. INDEX
   → Jos uusia dokumenttityyppejä (GUIDE, MARKET_RESEARCH, jne.)

4. ARCHITECTURE_OVERVIEW (jos tarpeen)
   → Jos moduuli muutti arkkitehtuuria
```

---

## Pikamuistilista (11-vaiheinen)

```
═══════════════ VALINNAINEN ═══════════════

[ ] Phase 0: MARKET RESEARCH (jos uusi projekti)
    - Kilpailijat, käyttäjätarpeet
    - VISION_DOC + ROADMAP

═══════════════ TOIMINNALLINEN ═══════════════

[ ] Phase 1: Konteksti (lue dokumentit)
[ ] Phase 2: RESEARCH (toiminnallinen tutkimus)
[ ] Phase 3: Suunnittelu (primitiivi, black box)
[ ] Phase 4: SPEC-kirjoitus (🟢/🟡/🔵 vaiheistus)
[ ] Phase 5: Käyttäjä-review
[ ] Phase 6: Gemini-review
[ ] Phase 7: SPEC-viimeistely

═══════════════ TEKNINEN ═══════════════

[ ] Phase 8: TECH_RESEARCH (teknologiat)
[ ] Phase 9: TECH_SPEC-kirjoitus
[ ] Phase 10: Gemini TECH-review

═══════════════ PÄIVITYKSET ═══════════════

[ ] Phase 11: Päivitä MASTER, LOKI, INDEX
```

---

## Muistisäännöt

> **"Phase 0 = strategia, Phase 1-11 = taktiikka"**  
> Market Research kertoo MIKSI ja MITÄ, SPEC kertoo MITÄ JA MITEN.

> **"Tutki ensin, kysy sitten"** - Älä kysy käyttäjältä ennen kuin olet tehnyt tiedonhaun.

> **"Vaihtoehdot + Ehdotus AINA"** - Kun kysyt käyttäjältä, anna valinnat ja oma suositus perusteluineen.

> **"RESEARCH tallentaa toiminnallisen oppimisen"** - "Miten muisti pitäisi toimia?"

> **"TECH_RESEARCH tallentaa teknisen oppimisen"** - "Millä kirjastoilla toteutetaan?"

> **"Käyttäjä ennen Geminiä"** - Jussi hyväksyy ennen teknistä reviewia.

> **"Numerointi yhtenäinen"** - RESEARCH_02, SPEC_02, TECH_RESEARCH_02, TECH_SPEC_02 kuuluvat yhteen.

> **"Kysy syventäviä kysymyksiä"** - "Mitä voi mennä pieleen?" on tärkeämpi kuin "Miten tämä toimii?"

> **"GitHub → project_knowledge_search"** - Tiedostot eivät näy /mnt/project -hakemistossa.

> **"Käytä 🟢/🟡/🔵 vaiheistusta"** - Jokainen ominaisuus merkitään, yhteenveto dokumentin lopussa.

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.5 | 2025-11-26 | **Phase 0 lisätty**: Market Research and Idea Evaluation/Evolution, 10→11-vaiheinen prosessi |
| 1.4 | 2025-11-25 | Vaiheistusohje (🟢/🟡/🔵), päivitetty SPEC-rakenne, checklist päivitetty, Gemini TECH_SPEC tarkistuslista lisätty |
| 1.3 | 2025-11-25 | TECH_RESEARCH -vaihe lisätty, prosessi 10-vaiheiseksi, dokumenttinimeäminen päivitetty |
| 1.2 | 2025-11-25 | Syventävät tutkimuskysymykset, GitHub-selvennys |
| 1.1 | 2025-11-25 | Välitallennus-ohje |
| 1.0 | 2025-11-25 | Ensimmäinen versio |

---

*Tämä dokumentti jatkuu PROCESS_Document_Updates.md:ssä (tallennus, versionhallinta, encoding).*
