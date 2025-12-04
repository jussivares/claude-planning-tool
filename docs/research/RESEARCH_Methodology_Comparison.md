# RESEARCH: Methodology Comparison - External AI Development Frameworks

> **Versio:** 1.0  
> **Päivitetty:** 2025-12-04  
> **Tyyppi:** META Research  
> **Status:** ✅ Valmis

---

## Tiivistelmä

Vertailu ulkoisten AI-ohjelmistosuunnittelun metodologioiden ja oman lähestymistapamme välillä. Tavoite: tunnistaa adoptoitavat parhaat käytännöt ja validoida oman metodologiamme vahvuudet.

---

## Tutkitut metodologiat

### 1. Specification Document Generator (adrianpuiu)

**Lähde:** https://github.com/adrianpuiu/specification-document-generator

**Kuvaus:** 6-vaiheinen arkkitehtuurikehys joka tuottaa 5 dokumenttia: research.md, blueprint.md, requirements.md, design.md, tasks.md, validation.md.

**Ydinfilosofia:** "Research slop" -esto ja juridinen turvaaminen pakollisella lähdeviittauksella.

---

## Vertailutaulukko

| Aspekti | Specification Document Generator | Meidän metodologia |
|---------|----------------------------------|-------------------|
| **Vaiheet** | 6 (Phase 0-5) | 11 (0-10) |
| **Fokus** | Hallusinaatioiden esto, citation | Kontekstin hallinta, 1M token |
| **Granulariteetti** | Koko projekti kerralla | Moduuli kerrallaan |
| **Dokumentit** | 5 koko projektille | N × 4 (RESEARCH, SPEC, TECH_RES, TECH_SPEC) per moduuli |
| **Iteraatio** | Lineaarinen (0→5) | Spiraali (SPEC ↔ DB ↔ ARCH) |
| **Validointi** | Python-skripti (100% coverage) | Gemini + Claude Code review |
| **Tekninen kerros** | Ei erillistä | TECH_RESEARCH + TECH_SPEC |
| **Arkkitehtuuriperiaatteet** | Single responsibility | Primitiivit, black box, wrap deps |

---

## Heidän vahvuudet (adoptoitavissa)

### V1: "Research Slop" -tietoisuus

**Havainto:** He tunnistavat eksplisiittisesti AI-hallusinaatioiden riskin:

> "AI generates content that is polished, articulate, and inspires 'faith-like confidence,' yet is often unverified and factually incorrect."

**Konkreettiset riskit:**
- Lakitoimistot: keksityt viittaukset → sanktiot
- Konsulttifirmat: virheelliset tilastot → sakot
- Ammattilaiset: uskottavan kuuloinen vale → maineen menetys

**Adoptoitavuus:** ✅ Korkea - lisätään tietoisuus PROCESS-dokumentteihin.

---

### V2: Strict Citation Protocol

**Havainto:** Pakollinen `[cite:INDEX]` -formaatti jokaiselle väitteelle.

```
❌ KIELLETTY (Research Slop):
"Node.js is great for real-time apps"

✅ VAADITTU (Evidence-Based):
"Node.js excels at real-time applications due to its event-driven, 
non-blocking I/O model [cite:1]"
```

**Adoptoitavuus:** 🔶 Osittainen
- RESEARCH-dokumenteissa: ✅ Kyllä, tiukka lähdeviittaus
- SPEC-dokumenteissa: 🔲 Ei tarpeen (ei väitteitä, vaan päätöksiä)

---

### V3: "Search THEN Browse" -sääntö

**Havainto:** 

> "You MUST read full source content, not just search snippets"

Hakutuloksen snippet voi olla:
- Kontekstista irrotettu
- Vanhentunut
- Harhaanjohtava

**Käytäntö:**
1. `web_search` → löydä lähteet
2. `web_fetch` → lue KOKO artikkeli
3. Vasta sitten tee johtopäätös

**Adoptoitavuus:** ✅ Korkea - lisätään PROCESS_SPEC_Writing:iin.

---

### V4: Automaattinen Coverage Validation

**Havainto:** Python-skripti tarkistaa:

```
Validation Report:
- Total Acceptance Criteria: 28
- Criteria Covered by Tasks: 28  
- Coverage Percentage: 100%
- Invalid References: 0
✅ Plan validated and ready for execution
```

**Adoptoitavuus:** ✅ Korkea (koodausvaiheessa)
- SPEC → Test coverage tarkistus
- Acceptance criteria → Implementation mapping

---

## Meidän vahvuudet (validoitu)

### M1: Moduulikohtainen dokumentaatio

He tekevät yhden dokumenttisetin koko projektille. Me teemme täyden syklin (RESEARCH → SPEC → TECH_RESEARCH → TECH_SPEC) **jokaiselle moduulille erikseen**.

**Miksi parempi:** Skaalautuu monimutkaisiin projekteihin, mahdollistaa rinnakkaisen työskentelyn.

---

### M2: Iteratiivinen spiraali

He käyttävät lineaarista Phase 0→5. Me ymmärrämme että:

```
SPEC ◄────────► DATABASE
  │      │           │
  ▼      ▼           ▼
ARKKITEHTUURI ◄──────┘
```

Dokumentit vaikuttavat toisiinsa ja kehittyvät samanaikaisesti.

---

### M3: TECH-kerros (WHAT vs HOW)

Heillä ei ole erillistä teknistä kerrosta. Meidän malli:

| Kerros | Sisältö | Esimerkki |
|--------|---------|-----------|
| SPEC | MITÄ tehdään | "Tallentaa keskusteluhistorian" |
| TECH_SPEC | MITEN tehdään | "SQLite-vec, parent-child chunking" |

**Miksi parempi:** Mahdollistaa teknologiavaihdon ilman SPEC-muutoksia.

---

### M4: Systems Architecture -periaatteet

He mainitsevat vain "single responsibility". Meidän periaatteet:

1. **Tunnista primitiivi** - jokaisen moduulin ydin
2. **Black box -rajapinnat** - sisäinen toteutus vaihdettavissa
3. **Wrap external dependencies** - ei suoria kutsuja
4. **Yksi omistaja per moduuli**
5. **10x-skaalautuvuus** - toimiiko 10× kuormalla?

---

### M5: 1M Token kontekstin hyödyntäminen

Heidän työkalu ei mainitse kontekstirajoja lainkaan. Meidän koko projekti rakentuu tämän ympärille:

- Pitkäkestoiset suunnittelusessiot
- Ulkoinen muisti (MemoryService)
- Hallittu kontekstin priorisointi (ContextManager)

---

### M6: Kaksoisrooli

He olettavat käyttäjän ohjaavan prosessia. Meidän Claude toimii:

```
┌─────────────────────────────────────────────────────────────┐
│  ROOLI 1: SUUNNITTELIJA     ROOLI 2: TUTKIJA               │
│  ─────────────────────      ─────────────────               │
│  Kysyy oikeat kysymykset    Hakee vastaukset (web search)   │
│  Tunnistaa vaihtoehdot      Vertailee ratkaisuja            │
│  Tekee suositukset          Dokumentoi lähteet              │
└─────────────────────────────────────────────────────────────┘
```

**"Tutki ensin, kysy sitten"** - ei kuormita käyttäjää turhilla kysymyksillä.

---

## Kehitysehdotukset

### Prioriteetti 1 (Välitön adoptointi)

| ID | Ehdotus | Kohde | Status |
|----|---------|-------|--------|
| **E1** | Lisää "Read Full Source" -sääntö | PROCESS_SPEC_Writing | 🔲 |
| **E2** | Lisää "Research Slop" -varoitus | PROCESS_SPEC_Writing | 🔲 |
| **E3** | Tiukenna citation protocol RESEARCH-dokumenteissa | RESEARCH_* template | 🔲 |

### Prioriteetti 2 (Koodausvaiheessa)

| ID | Ehdotus | Kohde | Status |
|----|---------|-------|--------|
| **E4** | Luo SPEC→Test coverage validation script | PROCESS_Code.md | 🔲 |
| **E5** | Acceptance criteria → Task mapping tarkistus | Validation tooling | 🔲 |

### Prioriteetti 3 (Myöhemmin)

| ID | Ehdotus | Kohde | Status |
|----|---------|-------|--------|
| **E6** | Formaalimpi [cite:X] -formaatti kaikissa RESEARCH-doceissa | Template update | 🔲 |

---

## Konkreettiset tekstilisäykset

### PROCESS_SPEC_Writing -lisäys (E1, E2)

```markdown
### Tiedonhaun kultaiset säännöt

#### 1. "Read Full Source" -sääntö

> **PAKOLLINEN:** Älä koskaan luota hakutuloksen snippettiin. 
> Käytä AINA `web_fetch` ja lue koko artikkeli ennen johtopäätöstä.

Hakutuloksen snippet voi olla:
- Kontekstista irrotettu
- Vanhentunut  
- Harhaanjohtava

**Oikea prosessi:**
1. `web_search` → löydä relevantit lähteet
2. `web_fetch` → lue KOKO artikkeli
3. Vasta sitten tee johtopäätös ja dokumentoi

#### 2. "Research Slop" -varoitus

> **VAROITUS:** AI-generoitu teksti voi olla "polished, articulate, 
> and inspires faith-like confidence" - mutta silti VÄÄRÄ.

Älä koskaan:
- Esitä väitteitä ilman lähdettä
- Luota muistiin teknisistä yksityiskohdista
- Oleta että "kuulostaa oikealta" = on oikein

AINA kun teet väitteen teknologiasta, varmista lähde.
```

### RESEARCH-dokumenttien citation-parannus (E3)

```markdown
### Lähdeviittaukset

Jokainen tekninen väite PITÄÄ merkitä lähteellä:

❌ KIELLETTY:
"SQLite-vec tukee HNSW-indeksointia"

✅ VAADITTU:
"SQLite-vec tukee HNSW-indeksointia (Lähde: sqlite-vec GitHub README, 2024)"

TAI formaalimmin:
"SQLite-vec tukee HNSW-indeksointia [1]"
...
[1] https://github.com/asg017/sqlite-vec - README.md, viitattu 2025-12-04
```

---

## Johtopäätös

**Specification Document Generator** on hyvä mutta kapeampi metodologia:

| Arvio | Selitys |
|-------|---------|
| **Vahvuus** | Hallusinaatioiden esto, citation discipline, validation script |
| **Heikkous** | Ei skaalaudu, ei iteroi, ei moduuliajattelua, ei teknistä kerrosta |

**Metafora:** 
- He: "Dokumenttigeneraattori" (yksi ajo → 5 dokumenttia)
- Me: "Suunnittelutyökalu" (pitkäkestoinen sessio → hallittu konteksti)

Adoptoimme parhaat osat (E1-E3), säilytämme omat vahvuutemme.

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.0 | 2025-12-04 | Ensimmäinen versio, Specification Document Generator -vertailu |

---

*Dokumentti on osa Claude API -suunnittelutyökalun META Research -vaihetta.*
