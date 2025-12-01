# INDEX - Projektin dokumenttikartta

> **Versio:** 1.12  
> **Päivitetty:** 2025-11-26  
> **Tiedosto:** v1_12_INDEX.md  
> **Projekti:** Claude API -suunnittelutyökalu

---

## Muutokset v1.11 → v1.12

| Muutos | Kuvaus |
|--------|--------|
| ✅ **Phase 0 dokumentit lisätty** | MARKET_RESEARCH ja VISION_DOC tyypit dokumentoitu |
| ✅ Dokumenttihierarkia päivitetty | Phase 0 oma tasonsa |
| ✅ Prosessiohjeet päivitetty | PROCESS_Market_Research lisätty listaan |

---

## Projektin status

```
[██████░░░░] 60% - Suunnittelu (SPEC-kerros lähes valmis)
```

**MemoryService dokumentoitu täysin** - valmis toteutettavaksi!  
**ContextManager SPEC valmis** - TECH_SPEC seuraavaksi tai koodaus.

---

## Dokumenttihierarkia

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  TASO 0: LIIKETOIMINTA (GitHub - vain projektin alussa) [UUSI] │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  MARKET_RESEARCH_[Project].md  Kilpailijat, käyttäjätarpeet││
│  │  VISION_DOC_[Project].md       Problem, Solution, MVP-scope ││
│  │  DEVELOPMENT_ROADMAP (optional) Aikataulu, prioriteetit    ││
│  │                                                             ││
│  │  Huom: Phase 0 valinnainen, vain uusille projekteille      ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  TASO 1: PROJEKTIN YDIN (Projekti - aina päällä)               │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  v1_12_INDEX.md            📍 Olet tässä                    ││
│  │  v1_7_KEHITYSLOKI.md       Päätökset, edistyminen           ││
│  │  v1_1_MASTER_FUNCTIONAL.md Moduulikartta, statukset         ││
│  │  v1_0_ARCHITECTURE_OVERVIEW.md  Tekninen arkkitehtuuri      ││
│  │  v1_0_API_REFERENCE.md     Claude API dokumentaatio         ││
│  │  v2_2_VS_CODE_WORKFLOW.md  Git-työnkulku                    ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  TASO 2: SYSTEM PROMPT (Projekti - aina päällä)                │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  v2_4_System_Prompt.md     Clauden ohjeet tälle projektille ││
│  │  (Custom instructions -kentässä)                            ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  TASO 3: TUTKIMUS- JA MÄÄRITTELYDOKUMENTIT (GitHub - ON/OFF)   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  RESEARCH_XX           Toiminnallinen tutkimus (mitä?)      ││
│  │  SPEC_XX               Toiminnallinen määrittely (mitä?)    ││
│  │  TECH_RESEARCH_XX      Teknologiatutkimus (millä?)          ││
│  │  TECH_SPEC_XX          Tekninen määrittely (miten?)         ││
│  │  GUIDE_XX              Käyttöohjeet ja best practices       ││
│  │                                                             ││
│  │  Huom: Ei version# nimessä - Git history hoitaa             ││
│  │  Numerointi: XX viittaa moduuliin (01=Claude, 02=Memory...) ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  TASO 4: PROSESSIOHJEET (Projekti - aina päällä)               │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  v1_0_PROCESS_Market_Research.md Phase 0: Market Research   ││
│  │  v1_5_PROCESS_SPEC_Writing.md    11-vaiheinen prosessi      ││
│  │  v1_7_PROCESS_Document_Updates.md Dokumentointikäytännöt    ││
│  │  v1_0_PROCESS_Database_Management.md Tietokannan hallinta   ││
│  │  v2_0_AI-avusteinen_ohjelmistosuunnittelu.md  Filosofia     ││
│  │  v1_2_SYNTEESI_Moderni_Suunnittelu.md  TECH_SPEC ohjeistus  ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Dokumenttien suhde (prosessi)

```
═══════════════ PHASE 0 (VALINNAINEN) ═══════════════

MARKET_RESEARCH → VISION_DOC → DEVELOPMENT_ROADMAP
(Kilpailijat,     (Problem,     (Aikataulu)
 Käyttäjätarpeet) Solution,
                  MVP-scope)

═══════════════ PHASE 1-11 (TOIMINNALLINEN + TEKNINEN) ═══════════════

RESEARCH_XX     → SPEC_XX    → TECH_RESEARCH_XX → TECH_SPEC_XX → CODE
(Toiminnallinen  (Mitä         (Millä              (Miten         (Toteutus)
 tutkimus)        tehdään)      tehdään)            tehdään)

Esimerkki MemoryService:
RESEARCH_02     → SPEC_02    → TECH_RESEARCH_02 → TECH_SPEC_02 → src/memory/
"Muistiarkkitehtuuri" "MemoryService"  "aiosqlite,         "Python-luokat, *.py
                                        Raw SQL, Pydantic"  SQL-skeemat"
```

---

## Moduulien dokumentaatiostatus

| Moduuli | RESEARCH | SPEC | TECH_RESEARCH | TECH_SPEC | CODE |
|---------|----------|------|---------------|-----------|------|
| **ClaudeService** | ✅ v1.0 | ✅ v1.1 | 🔲 | 🔲 | 🔲 |
| **MemoryService** | ✅ v1.0 | ✅ v1.1 | ✅ v1.0 | ✅ v1.0 | 🔲 |
| **ContextManager** | ✅ v1.0 | ✅ v1.0 | 🔲 | 🔲 | 🔲 |
| SessionManager | 🔲 | 🔲 | 🔲 | 🔲 | 🔲 |
| Chat UI | 🔲 | 🔲 | 🔲 | 🔲 | 🔲 |

**Symbolit:** ✅ Valmis | 🔶 Työn alla | 🔲 Ei aloitettu

---

## GitHub-dokumenttien sijainnit

### PHASE 0: Liiketoiminta (docs/ tai docs/research/)

| Dokumentti | Polku | Milloin? |
|------------|-------|----------|
| MARKET_RESEARCH | `docs/research/MARKET_RESEARCH_[Project].md` | Uusi projekti |
| VISION_DOC | `docs/VISION_DOC_[Project].md` | Uusi projekti |
| DEVELOPMENT_ROADMAP | `docs/DEVELOPMENT_ROADMAP_[Project].md` | Valinnainen |

### RESEARCH-dokumentit (docs/research/)

| Dokumentti | Polku | Status |
|------------|-------|--------|
| RESEARCH_01 | `docs/research/RESEARCH_01_Claude_API_Patterns.md` | ✅ v1.0 |
| RESEARCH_02 | `docs/research/RESEARCH_02_Memory_Architecture.md` | ✅ v1.0 |
| RESEARCH_03 | `docs/research/RESEARCH_03_Context_Manager.md` | ✅ v1.0 |
| TECH_RESEARCH_02 | `docs/research/TECH_RESEARCH_02_Memory_Service.md` | ✅ v1.0 |

### SPEC-dokumentit (docs/specs/)

| Dokumentti | Polku | Status |
|------------|-------|--------|
| SPEC_01 | `docs/specs/SPEC_01_Claude_Service.md` | ✅ v1.1 |
| SPEC_02 | `docs/specs/SPEC_02_Memory_Service.md` | ✅ v1.1 |
| SPEC_03 | `docs/specs/SPEC_03_Context_Manager.md` | ✅ v1.0 |

### TECH_SPEC-dokumentit (docs/tech-specs/)

| Dokumentti | Polku | Status |
|------------|-------|--------|
| TECH_SPEC_02 | `docs/tech-specs/TECH_SPEC_02_Memory_Service.md` | ✅ v1.0 |

### GUIDE-dokumentit (docs/guides/)

| Dokumentti | Polku | Status |
|------------|-------|--------|
| GUIDE_Extended_Thinking | `docs/guides/GUIDE_Extended_Thinking_Usage.md` | ✅ v1.0 |

---

## Claude-projektin tiedostot

### Aina kontekstissa (~12K tokenia)

**Ydin (~12K):**

| Dokumentti | Tiedostonimi | Tarkoitus |
|------------|--------------|-----------|
| INDEX | `v1_12_INDEX.md` | 📍 Navigointikartta |
| KEHITYSLOKI | `v1_7_KEHITYSLOKI.md` | Päätökset, edistyminen |
| MASTER_FUNCTIONAL | `v1_1_MASTER_FUNCTIONAL.md` | Moduulit, statukset |
| ARCHITECTURE_OVERVIEW | `v1_0_ARCHITECTURE_OVERVIEW.md` | Tekninen rakenne |
| API_REFERENCE | `v1_0_API_REFERENCE.md` | Claude API referenssi |
| VS_CODE_WORKFLOW | `v2_2_VS_CODE_WORKFLOW.md` | Git-työnkulku |

**Prosessiohjeet (~8K tokenia):**

| Dokumentti | Tiedostonimi | Kuvaus |
|------------|--------------|--------|
| PROCESS_Market_Research | `v1_0_PROCESS_Market_Research.md` | Phase 0 prosessi |
| PROCESS_SPEC_Writing | `v1_5_PROCESS_SPEC_Writing.md` | 11-vaiheinen prosessi |
| PROCESS_Document_Updates | `v1_7_PROCESS_Document_Updates.md` | Dokumentointi + UTF-8 |
| PROCESS_Database_Management | `v1_0_PROCESS_Database_Management.md` | Tietokanta |
| AI-avusteinen suunnittelu | `v2_0_AI-avusteinen_ohjelmistosuunnittelu.md` | Filosofia |
| SYNTEESI | `v1_2_SYNTEESI_Moderni_Suunnittelu.md` | TECH_SPEC malli |

---

## GitHub ON/OFF -ohje

### Milloin aktivoida

Claude pyytää dokumenttia näin:
```
"Tarvitsen SPEC_01:n. Voitko aktivoida sen GitHubista?"
```

### Aktivointi

1. Projektin oikeassa sivupalkissa → GitHub-osio
2. Klikkaa tiedoston toggle-nappia (ON)
3. Sano Claudelle: "Valmis!"

### Deaktivointi

Claude ilmoittaa kun ei enää tarvitse:
```
"Voit deaktivoida SPEC_01:n. Olen saanut tarvitsemani tiedot."
```

Toggle OFF → Token-budjetti vapautuu.

---

## Session aloitus -pikaohje

### 1. Tarkista missä ollaan

```
Lue: INDEX.md → Projektin status, dokumenttisijainnit
Lue: KEHITYSLOKI.md → Viimeisin edistyminen, seuraavat askeleet
```

### 2. Määritä session tavoite

Kysy käyttäjältä tai tarkista KEHITYSLOKI:n "Seuraavat askeleet" -osio.

### 3. Jos uusi projekti → Ehdota Phase 0

```
"Aloitetaanko Market Research -vaiheella (Phase 0)?
Katso: PROCESS_Market_Research.md"
```

### 4. Käytä oikeita työkaluja

| Tehtävä | Käytä dokumenttia |
|---------|-------------------|
| Moduulin status | MASTER_FUNCTIONAL |
| Arkkitehtuuripäätös | ARCHITECTURE_OVERVIEW + systems-architecture skill |
| API-toteutus | API_REFERENCE |
| Git commit/push | VS_CODE_WORKFLOW |
| Päätöksen kirjaus | KEHITYSLOKI |
| Phase 0 prosessi | PROCESS_Market_Research |
| SPEC/TECH_SPEC -kirjoitus | PROCESS_SPEC_Writing |
| Dokumenttipäivitys | PROCESS_Document_Updates |

### 5. Session lopetus

1. Päivitä KEHITYSLOKI (mitä tehtiin, seuraavat askeleet)
2. Päivitä MASTER_FUNCTIONAL (jos moduulin status muuttui)
3. Päivitä INDEX (jos uusia dokumentteja)
4. Tallenna uudet dokumentit GitHubiin (VS_CODE_WORKFLOW)
5. Muistuta käyttäjää päivittämään projektin dokumentit

---

## Token-budjetti

| Osio | Tokenit |
|------|---------|
| Pakolliset projektidokumentit | ~12K |
| Prosessiohjeet | ~8K |
| Claude system prompt | ~5K |
| **Yhteensä perustaso** | **~25K** |
| **Käytettävissä keskusteluun** | **~165K** |

**GitHub-dokumentit:** ~2K tokenia/dokumentti (aktivoi max 1-2 kerrallaan)

**Phase 0 kustannus:** ~20K tokenia (MARKET_RESEARCH + VISION_DOC)

---

## Kriittiset API-tiedot (pikakatsaus)

| Parametri | Arvo |
|-----------|------|
| **1M konteksti -header** | `anthropic-beta: context-1m-2025-08-07` |
| **Tuetut mallit (1M)** | Sonnet 4, Sonnet 4.5 |
| **Vaatimus** | Usage tier 4 |
| **Opus 4.5** | EI tue 1M kontekstia |

---

## Seuraavat askeleet

### Session #9 (2025-11-26) - Käynnissä

1. ✅ **Dokumenttipäivitykset Phase 0** - Valmis!
   - PROCESS_Market_Research v1.0
   - PROCESS_SPEC_Writing v1.5
   - System Prompt v2.4
   - INDEX v1.12

2. **TAUKO** - Käyttäjä lataa dokumentit

3. **Market Research tälle projektille** (harjoitus + strategia):
   - MARKET_RESEARCH_Claude_Planning_Tool.md
   - VISION_DOC_Claude_Planning_Tool.md
   - Phase 2/3 kehityspolku

### Myöhemmin

- SPEC_04_Session_Manager
- SPEC_05_Chat_UI
- MemoryService koodaus
- TECH_RESEARCH_01 + TECH_SPEC_01 (ClaudeService)
- TECH_RESEARCH_03 + TECH_SPEC_03 (ContextManager)
- Frontend-toteutus

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.12 | 2025-11-26 | **Phase 0 lisätty**: MARKET_RESEARCH ja VISION_DOC dokumenttityypit, PROCESS_Market_Research prosessiohjeisiin, dokumenttihierarkia päivitetty |
| 1.11 | 2025-11-26 | Session #8 tulokset: RESEARCH_03 + SPEC_03 ContextManager, GUIDE-dokumenttityyppi, prosessiohjeet päivitetty |
| 1.10 | 2025-11-25 | Session #7 vahvistus, versionumerointi tarkistettu |
| 1.9 | 2025-11-25 | TECH_RESEARCH_02 + TECH_SPEC_02 statukset, prosessiohjeet v1.3 |
| 1.8 | 2025-11-25 | RESEARCH_02 + SPEC_02 statukset |
| 1.7 | 2025-11-25 | GitHub ON/OFF -ohjeet |
| 1.6 | 2025-11-25 | Prosessidokumentit |
| 1.0 | 2025-11-25 | Ensimmäinen versio |

---

*Dokumentti on osa Claude API -suunnittelutyökalun dokumentaatiota.*
