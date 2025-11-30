# PROCESS_Market_Research

> **Versio:** 1.0  
> **PÃ¤ivitetty:** 2025-11-26  
> **Tiedosto:** v1_0_PROCESS_Market_Research.md  
> **Tarkoitus:** Phase 0: Market Research and Idea Evaluation/Evolution  
> **Liittyy:** PROCESS_SPEC_Writing.md (jatkaa Phase 1:een)  

---

## Yleiskatsaus

**Phase 0** on valinnainen, mutta erittÃ¤in arvokas vaihe **ennen** teknistÃ¤ suunnittelua. Sen tavoite on:

1. **YmmÃ¤rtÃ¤Ã¤ markkinaa** - MitÃ¤ muut tekevÃ¤t oikein ja vÃ¤Ã¤rin?
2. **Validoida idea** - Onko tÃ¤lle oikeasti tarvetta?
3. **KehittÃ¤Ã¤ ideaa** - Miten idea voi evoluoitua tutkimuksen perusteella?
4. **Priorisoida MVP** - MitkÃ¤ ominaisuudet ovat kriittisimpiÃ¤?

---

## Milloin Phase 0 kannattaa kÃ¤yttÃ¤Ã¤?

### âœ… SUOSITELTU (Phase 0 hyÃ¶dyllinen)

| Projektin tyyppi | Miksi? |
|------------------|--------|
| **Startup / uusi tuote** | MarkkinaymmÃ¤rrys kriittinen |
| **SaaS-sovellus** | Kilpailu kovaa, differentiaatio tÃ¤rkeÃ¤Ã¤ |
| **B2C-sovellus** | KÃ¤yttÃ¤jÃ¤tarpeet monimutkaisia |
| **PitkÃ¤aikainen projekti** | Investointi kannattaa (6kk+) |
| **Tiimiprojekti** | Vision_Doc auttaa yhteisymmÃ¤rryksessÃ¤ |

### âš ï¸ EHKÃ„ (harkitse tapauskohtaisesti)

| Projektin tyyppi | Huomio |
|------------------|--------|
| **SisÃ¤inen tyÃ¶kalu** | Jos kÃ¤yttÃ¤jÃ¤ryhmÃ¤ pieni ja tuttu, kevyempi research riittÃ¤Ã¤ |
| **Enterprise B2B** | Jos asiakkaan vaatimukset selvÃ¤t, Phase 0 voi olla overkill |

### âŒ EI SUOSITELLA (Phase 0 liian raskas)

| Projektin tyyppi | Miksi? |
|------------------|--------|
| **Tekninen kirjasto** | Teknologia-driven ok, ei tarvita liiketoimintakontekstia |
| **Proof of concept** | Nopeutta tÃ¤rkeÃ¤mpi kuin perusteellisuus |
| **Hyvin mÃ¤Ã¤ritelty replica** | Jos kopioidaan tunnettua tuotetta 1:1 |
| **<2 viikon projekti** | Token-budjetti ja aika ei kannata |

---

## Prosessin vaiheet

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚              PHASE 0: MARKET RESEARCH & IDEA EVOLUTION          â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                 â”‚
â”‚  1. VALMISTELU       MÃ¤Ã¤rittele research-scope                  â”‚
â”‚         â”‚                                                       â”‚
â”‚         â–¼                                                       â”‚
â”‚  2. TUTKIMUS         Web search + analyysi (~30-45 min)         â”‚
â”‚         â”‚            â†’ MARKET_RESEARCH_[Project].md             â”‚
â”‚         â–¼                                                       â”‚
â”‚  3. SYNTEESI         Vision Doc kirjoitus                       â”‚
â”‚         â”‚            â†’ VISION_DOC_[Project].md                  â”‚
â”‚         â–¼                                                       â”‚
â”‚  4. USER REVIEW      KÃ¤yttÃ¤jÃ¤ hyvÃ¤ksyy vision                   â”‚
â”‚         â”‚                                                       â”‚
â”‚         â–¼                                                       â”‚
â”‚  5. PRIORISOINTI     MVP vs Phase 2/3 + Roadmap                 â”‚
â”‚         â”‚            â†’ PÃ¤ivitetty Vision Doc                    â”‚
â”‚         â–¼                                                       â”‚
â”‚  â†’ Phase 1 (RESEARCH_XX) alkaa                                  â”‚
â”‚                                                                 â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## Vaihe 1: Valmistelu

### Claude kysyy kÃ¤yttÃ¤jÃ¤ltÃ¤:

```
"Aloitetaanko projekti Market Research -vaiheella (Phase 0)?"

TÃ¤mÃ¤ vie ~30-45 minuuttia ja ~20K tokenia, mutta antaa:
âœ… Kilpailijakartan ja best practices
âœ… KÃ¤yttÃ¤jÃ¤tarpeiden priorisoinnin
âœ… Vision dokumentin sidosryhmille
âœ… Realistisen MVP-scopen

Jatketaanko?
```

### Jos kÃ¤yttÃ¤jÃ¤ hyvÃ¤ksyy â†’ mÃ¤Ã¤rittele scope

**Claude kysyy:**

```
"MÃ¤Ã¤rittele tutkimuksen laajuus:

1. Nopea (15-20 min)
   - 3-5 kilpailijaa
   - Pinnallinen analyysi
   - Lyhyt Vision Doc

2. Keskitaso (30-45 min) [SUOSITELTU]
   - 5-8 kilpailijaa
   - KÃ¤yttÃ¤jÃ¤foorumit (Reddit, HN)
   - Kattava Vision Doc + Roadmap

3. SyvÃ¤llinen (60-90 min)
   - 10+ kilpailijaa
   - Research papers
   - Yksityiskohtainen markkina-analyysi

Suositus: Valitse 2 (keskitaso) ensimmÃ¤isellÃ¤ kerralla."
```

---

## Vaihe 2: Tutkimus

### Tutkimuskysymykset

Claude laatii tutkimuskysymykset jÃ¤rjestelmÃ¤llisesti:

```
YLEINEN MALLI:

1. KILPAILIJAT
   - KetkÃ¤ ovat suorat kilpailijat?
   - MitÃ¤ ominaisuuksia heillÃ¤ on?
   - MikÃ¤ on heidÃ¤n hintamalli?
   - MitÃ¤ he tekevÃ¤t hyvin? Huonosti?

2. KÃ„YTTÃ„JÃ„TARPEET
   - MitÃ¤ kÃ¤yttÃ¤jÃ¤t valittavat nykyisistÃ¤ ratkaisuista?
   - MitkÃ¤ ominaisuudet mainitaan useimmiten?
   - MitÃ¤ "would be nice to have" -asioita?

3. BEST PRACTICES
   - MitkÃ¤ design patternit ovat yleisiÃ¤?
   - MitÃ¤ teknologioita kÃ¤ytetÃ¤Ã¤n?
   - MitkÃ¤ arkkitehtuuripÃ¤Ã¤tÃ¶kset toistuvat?

4. TRENDIT
   - Mihin suuntaan ala kehittyy?
   - MitkÃ¤ teknologiat nousevat?
   - MitÃ¤ uusia kÃ¤yttÃ¶tapauksia ilmenee?
```

### Web Search -strategia

**ðŸŽ¯ TÃ„RKEÃ„:** Ã„lÃ¤ tee kevyesti tÃ¤tÃ¤ vaihetta! Systematemaattinen tutkimus maksaa itsensÃ¤ takaisin.

```python
# VAIHE 1: Kilpailijat (5-8 hakua)
- "AI planning tool long context"
- "Claude alternative memory management"
- "developer conversation tools"
- "AI-assisted software design"
- "MemGPT vs alternatives"

# VAIHE 2: KÃ¤yttÃ¤jÃ¤palaute (3-5 hakua)
- "site:reddit.com AI planning tool frustrations"
- "site:news.ycombinator.com Claude context window"
- "developer pain points AI tools"

# VAIHE 3: Best Practices (3-5 hakua)
- "long context RAG best practices"
- "session management AI systems"
- "memory architecture patterns"

# VAIHE 4: Research (2-3 hakua, jos syvÃ¤llinen)
- "long context language models research"
- "semantic memory decay patterns"
```

### LÃ¤hteiden analysointi

**Jokaisesta lÃ¤hteestÃ¤ poimi:**

```
KILPAILIJA-ANALYYSI:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Nimi:           MemGPT                                  â”‚
â”‚ URL:            https://memgpt.ai                       â”‚
â”‚ Ydinominaisuus: Hierarchical memory system              â”‚
â”‚ Hinnoittelu:    Free tier + $20/mo                      â”‚
â”‚                                                         â”‚
â”‚ âœ… Hyvin:       - PitkÃ¤ muisti                          â”‚
â”‚                 - Avoin lÃ¤hdekoodi                      â”‚
â”‚                                                         â”‚
â”‚ âŒ Huonosti:    - Monimutkainen setup                   â”‚
â”‚                 - UI ei intuitiivinen                   â”‚
â”‚                                                         â”‚
â”‚ ðŸ’¡ Opimme:      Hierarchical memory = hyvÃ¤ idea,       â”‚
â”‚                 mutta UX pitÃ¤Ã¤ olla yksinkertainen      â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Dokumentointi

Tallenna kaikki lÃ¶ydÃ¶kset â†’ **MARKET_RESEARCH_[Project].md**

**Rakenne:**

```markdown
# MARKET_RESEARCH_Claude_Planning_Tool

> **Versio:** 1.0
> **TutkimuspÃ¤ivÃ¤:** 2025-11-26
> **Tutkija:** Claude (ohjattu kÃ¤yttÃ¤jÃ¤n toimesta)

## Executive Summary

[2-3 kappaletta: Keskeiset lÃ¶ydÃ¶kset]

## Kilpailija-analyysi

### 1. MemGPT
[Yksityiskohtainen analyysi]

### 2. LangChain
[Yksityiskohtainen analyysi]

...

## KÃ¤yttÃ¤jÃ¤tarpeet

### Priorisoitu lista (esiintymisfrekvenssin mukaan)

| Tarve | Frekvenssi | LÃ¤hde |
|-------|-----------|-------|
| Muisti sessionien vÃ¤lillÃ¤ | 87% | Reddit, HN |
| PitkÃ¤ konteksti (>200K) | 76% | HN, Discord |
| Dokumentaation automaatio | 54% | Reddit |

## Best Practices

### Muistiarkkitehtuuri
- Semantic search > keyword search
- Markdown > proprietary formats
- Gradual decay (vanhat haalistuvat)

### UI/UX
- Session branching (Notion AI)
- Auto-save (Cursor)
- Keyboard shortcuts (suosittua devien keskuudessa)

## Teknologiatrendit

- Vector databases (Pinecone, Weaviate)
- Embedding models (OpenAI, Cohere)
- Long context models (Claude, GPT-4 Turbo)

## Suositukset

[Claude tiivistÃ¤Ã¤: MitÃ¤ meidÃ¤n pitÃ¤isi tehdÃ¤?]

---

*LÃ¤hteet: [Lista kaikista kÃ¤ytetyistÃ¤ lÃ¤hteistÃ¤]*
```

---

## Vaihe 3: Synteesi (Vision Doc)

### Vision Doc -rakenne

```markdown
# VISION_DOC_Claude_Planning_Tool

> **Versio:** 1.0
> **PÃ¤ivitetty:** 2025-11-26
> **Status:** Draft / Reviewed / Approved

---

## 1. Ongelma (Problem Statement)

[MikÃ¤ on ongelma jota ratkomme?]

**Esimerkki:**
```
Claude.ai on loistava tyÃ¶kalu, mutta sillÃ¤ on kolme kriittistÃ¤ rajoitetta:
1. 200K token konteksti-ikkuna (liian pieni monimutkaisille projekteille)
2. Ei muistia sessionien vÃ¤lillÃ¤ (jokainen keskustelu alkaa tyhjÃ¤stÃ¤)
3. Dokumentaatio hajallaan (ei yhtÃ¤ totuuden lÃ¤hdettÃ¤)

TÃ¤mÃ¤ johtaa:
- Jatkuvaan kontekstin uudelleenlatailuun (aikaa hukkaan)
- PÃ¤Ã¤tÃ¶sten katoamiseen (sama asia pÃ¤Ã¤tetÃ¤Ã¤n moneen kertaan)
- Frustraatioon (keskustelun keskeytyminen keskellÃ¤ tyÃ¶tÃ¤)
```

## 2. Ratkaisu (Solution)

[Miten ratkaisemme tÃ¤mÃ¤n?]

**Esimerkki:**
```
Claude Planning Tool on erikoistunut AI-assistentti ohjelmistosuunnitteluun, joka:

1. HyÃ¶dyntÃ¤Ã¤ Clauden tÃ¤yden 1M token kontekstin (5x enemmÃ¤n kuin claude.ai)
2. Tallentaa kaiken automaattisesti ulkoiseen muistiin (Markdown + SQLite)
3. Muistaa projektin historian ja pÃ¤Ã¤tÃ¶kset ikuisesti
4. Generoi dokumentaation suunnittelun sivutuotteena

â†’ Tulos: YhtÃ¤jaksoinen, tehokas suunnittelu ilman keskeytyksiÃ¤.
```

## 3. KohderyhmÃ¤ (Target Audience)

[Kenelle tÃ¤mÃ¤ on?]

**Esimerkki:**
```
PRIMARY: Ohjelmistosuunnittelijat ja arkkitehdit
- TyÃ¶skentelevÃ¤t monimutkaisten jÃ¤rjestelmien parissa
- Tarvitsevat pitkÃ¤Ã¤ kontekstia (API-design, arkkitehtuuri)
- Arvostavat dokumentaatiota

SECONDARY: Tech leads ja CTO:t
- Haluavat pÃ¤Ã¤tÃ¶sten jÃ¤ljitettÃ¤vyyden
- Tarvitsevat vision-dokumentteja sidosryhmille

NOT FOR: Casual AI-kÃ¤yttÃ¤jÃ¤t
- Jos tarvitaan vain nopeat vastaukset
- Jos projekti kestÃ¤Ã¤ alle viikon
```

## 4. Kilpailuetu (Competitive Advantage)

[Miksi valita meidÃ¤n ratkaisu kilpailijoiden sijaan?]

**Malli:**
```
| Ominaisuus | Claude.ai | MemGPT | LangChain | MEIDÃ„N |
|------------|-----------|--------|-----------|--------|
| Konteksti | 200K | 1M | N/A | **1M** |
| Muisti sessionien vÃ¤lillÃ¤ | âŒ | âœ… | âš ï¸ | âœ… |
| HelppokÃ¤yttÃ¶isyys | âœ…âœ… | âŒ | âŒ | âœ… |
| Dokumentaation autom. | âŒ | âŒ | âŒ | âœ… |
| Git-integraatio | âŒ | âŒ | âš ï¸ | âœ… |

Kilpailuetumme:
1. YhdistetÃ¤Ã¤n pitkÃ¤ konteksti + muisti + helppous
2. Suunniteltu NIMENOMAAN ohjelmistosuunnitteluun (ei yleiskÃ¤yttÃ¶)
3. Dokumentaatio syntyy automaattisesti (ei erillinen tyÃ¶)
```

## 5. MVP-scope

[MitÃ¤ rakennetaan ENSIKSI?]

**Kriteerit MVP:lle:**
- Ratkaistava ydinongelmaan (pitkÃ¤ konteksti + muisti)
- Toteutettavissa 2-3 kuukaudessa
- RiittÃ¤vÃ¤n hyÃ¶dyllinen kÃ¤ytettÃ¤vÃ¤ksi heti

**MVP-ominaisuudet:**
```
âœ… MUST HAVE (MVP Phase 1):
- Claude API -integraatio (1M context)
- Ulkoinen muisti (Markdown + SQLite)
- Session-hallinta (tallennus, lataus)
- Yksinkertainen web UI (chat-ikkuna)
- Tiedostojen upload (projektidata)

â³ SHOULD HAVE (Phase 2):
- Semantic search (vektorihaku)
- Tiimi-ominaisuudet (shared projects)
- Export (PDF, DOCX)

ðŸ’Ž NICE TO HAVE (Phase 3):
- Multi-project hallinta
- Advanced analytics
- Plugin system
```

## 6. Success Metrics

[Miten mittaamme onnistumisen?]

**Esimerkki:**
```
MVP Success (3kk):
- 50 aktiivista kÃ¤yttÃ¤jÃ¤Ã¤
- KeskimÃ¤Ã¤rÃ¤inen session pituus >1h (vs claude.ai ~20min)
- NPS >40
- 0 kriittistÃ¤ bugia tuotannossa

Product-Market Fit (6kk):
- 200 aktiivista kÃ¤yttÃ¤jÃ¤Ã¤
- 30% kÃ¤yttÃ¤jistÃ¤ maksavia ($20/kk)
- Retention >60% (30 pÃ¤ivÃ¤n)
```

## 7. Development Roadmap

[Aikataulu ja priorisointi]

**Q1 2025 - MVP (Phase 1):**
```
Tammi:  ClaudeService + MemoryService
Helmi:  ContextManager + SessionManager
Maalis: Basic UI + Testing + Launch
```

**Q2 2025 - Phase 2:**
```
Huhti:  Semantic search
Touko:  Team features
KesÃ¤:   Export + Integrations
```

**Q3 2025 - Phase 3:**
```
Multi-project, Analytics, Plugins
```

---

## 8. Avoimet kysymykset

[MitÃ¤ emme vielÃ¤ tiedÃ¤?]

**Esimerkki:**
```
1. Hinnoittelu: Free tier + Pro? Vai pelkkÃ¤ Pro?
2. Hosting: Cloud vai self-hosted?
3. API access: Annetaanko kÃ¤yttÃ¤jille suora API-access?
```

---

## 9. Appendix: Research Summary

[Linkki tÃ¤yteen MARKET_RESEARCH dokumenttiin]
```

---

## Vaihe 4: User Review

### Claude esittÃ¤Ã¤ kÃ¤yttÃ¤jÃ¤lle:

```
"Vision Doc v1.0 valmis. Tarkista:

1. Onko Problem Statement oikea?
2. Puuttuuko kohderyhmÃ¤stÃ¤ joku tÃ¤rkeÃ¤ segmentti?
3. Onko MVP-scope realistinen?
4. Haluatko lisÃ¤tÃ¤/poistaa jotain Phase 2/3:sta?

TÃ„RKEÃ„: Vision Doc on sinun dokumenttisi. Voit hylÃ¤tÃ¤ tai muuttaa
mitÃ¤ tahansa, vaikka tutkimus ehdottaisi muuta."
```

### KÃ¤yttÃ¤jÃ¤ pÃ¤Ã¤ttÃ¤Ã¤:

```
Vaihtoehto A: HyvÃ¤ksyy sellaisenaan
   â†’ Phase 0 valmis, siirrytÃ¤Ã¤n Phase 1:een

Vaihtoehto B: PyytÃ¤Ã¤ muutoksia
   â†’ Claude pÃ¤ivittÃ¤Ã¤ Vision Doc â†’ uusi review

Vaihtoehto C: HylkÃ¤Ã¤ kokonaan
   â†’ Phase 0 keskeytetÃ¤Ã¤n, siirrytÃ¤Ã¤n suoraan Phase 1:een
```

---

## Vaihe 5: Priorisointi & Roadmap

### Development Roadmap (tarkempi)

Jos Vision Doc hyvÃ¤ksytty â†’ laaditaan tarkempi roadmap:

```markdown
# DEVELOPMENT_ROADMAP_Claude_Planning_Tool

## MVP (Phase 1) - Q1 2025

### Moduulit prioriteettijÃ¤rjestyksessÃ¤:

1. **ClaudeService** (2 viikkoa)
   - API-wrapper, streaming, error handling
   - SPEC_01 + TECH_SPEC_01

2. **MemoryService** (3 viikkoa)
   - Markdown + SQLite + semantic search
   - SPEC_02 + TECH_SPEC_02

3. **ContextManager** (2 viikkoa)
   - Token-laskenta, priorisointi
   - SPEC_03 + TECH_SPEC_03

4. **SessionManager** (2 viikkoa)
   - Tallennus, lataus, historia
   - SPEC_04 + TECH_SPEC_04

5. **Chat UI** (3 viikkoa)
   - React frontend, websocket
   - SPEC_05 + TECH_SPEC_05

**YhteensÃ¤: ~12 viikkoa (3 kuukautta)**

## Phase 2 - Q2 2025

### Uudet ominaisuudet:

1. **Semantic Search** (2 viikkoa)
   - Vektorihaku, relevance scoring
   - SPEC_06 + TECH_SPEC_06

2. **Team Features** (3 viikkoa)
   - Shared projects, permissions
   - SPEC_07 + TECH_SPEC_07

3. **Export System** (2 viikkoa)
   - PDF, DOCX, Markdown export
   - SPEC_08 + TECH_SPEC_08

**YhteensÃ¤: ~7 viikkoa**

## Phase 3 - Q3 2025

[Placeholder - mÃ¤Ã¤ritellÃ¤Ã¤n Phase 2:n jÃ¤lkeen]
```

---

## Token-budjetti

### Phase 0 kustannukset:

| Vaihe | Tokenit | Aika |
|-------|---------|------|
| Valmistelu | ~1K | 5 min |
| Tutkimus (web_search) | ~10-15K | 30-45 min |
| MARKET_RESEARCH kirjoitus | ~5K | 10 min |
| VISION_DOC kirjoitus | ~3K | 10 min |
| Review & iteraatio | ~2K | 10 min |
| **YHTEENSÃ„** | **~20K** | **~75 min** |

**Ratkaisu:** Tallenna GitHub â†’ Deaktivoi heti kun ei tarvita â†’ SÃ¤Ã¤stÃ¤ tokeneita

---

## Onnistumisen kriteerit

### âœ… HyvÃ¤ Phase 0 -tutkimus sisÃ¤ltÃ¤Ã¤:

```
1. KILPAILIJA-ANALYYSI:
   - VÃ¤hintÃ¤Ã¤n 5 relevanttia kilpailijaa
   - Yksityiskohtaiset +/- listat
   - SelkeÃ¤t oppimislÃ¶ydÃ¶kset

2. KÃ„YTTÃ„JÃ„TARPEET:
   - Priorisoitu lista (frekvenssi)
   - Konkreettiset lÃ¤hteet
   - YllÃ¤ttÃ¤viÃ¤ lÃ¶ydÃ¶ksiÃ¤ (ei vain itsestÃ¤Ã¤nselvyyksiÃ¤)

3. VISION DOC:
   - SelkeÃ¤ ongelma + ratkaisu
   - Realistinen MVP-scope
   - Mitattavat success metrics
   - KÃ¤yttÃ¤jÃ¤n hyvÃ¤ksymÃ¤

4. ROADMAP:
   - Moduulit priorisoitu
   - Aikataulu-estimaatit
   - Riippuvuudet tunnistettu
```

### âŒ Huono Phase 0 -tutkimus:

```
- Pinnallinen (vain 2-3 kilpailijaa, ei analyysiÃ¤)
- Geneerinen (ei konkreettisia lÃ¶ydÃ¶ksiÃ¤)
- Vision Doc liian abstrakti (ei MVP-scopea)
- Roadmap puuttuu tai epÃ¤realistinen
```

---

## Dokumenttien sijainnit

```
MARKET_RESEARCH_[Project].md  â†’ docs/research/
VISION_DOC_[Project].md        â†’ docs/
DEVELOPMENT_ROADMAP_[Project].md â†’ docs/ (optional)
```

**GitHub-synkaus:**
- Aktivoi VAIN tarvittaessa (esim. kun revisoidaan visiota)
- Muuten pidÃ¤ OFF:ssa (token-sÃ¤Ã¤stÃ¶)

---

## MuistisÃ¤Ã¤nnÃ¶t

> **"Systematiikka > Nopeus"**  
> Ã„lÃ¤ tee kevyesti Phase 0:aa. 30 minuuttia tutkimusta sÃ¤Ã¤stÃ¤Ã¤ viikkoja vÃ¤Ã¤rÃ¤Ã¤n suuntaan rakentamista.

> **"KÃ¤yttÃ¤jÃ¤ pÃ¤Ã¤ttÃ¤Ã¤"**  
> Research ehdottaa, Vision Doc dokumentoi, mutta kÃ¤yttÃ¤jÃ¤ tekee lopulliset pÃ¤Ã¤tÃ¶kset.

> **"MVP on minimalistinen"**  
> Jos ominaisuus ei ratkaise ydinongelmaa, se on Phase 2/3:ssa.

> **"Roadmap on elÃ¤vÃ¤"**  
> Vision Doc lukitaan, mutta Roadmap voi muuttua oppimisen myÃ¶tÃ¤.

---

## LiittyvÃ¤t dokumentit

- **PROCESS_SPEC_Writing.md** - Phase 1 alkaa tÃ¤Ã¤ltÃ¤ (RESEARCH_XX)
- **System_Prompt.md** - Ohjeet milloin kÃ¤yttÃ¤Ã¤ Phase 0:aa
- **INDEX.md** - Dokumenttien sijainnit

---

## Muutoshistoria

| Versio | PÃ¤ivÃ¤mÃ¤Ã¤rÃ¤ | Muutokset |
|--------|------------|-----------|
| 1.0 | 2025-11-26 | EnsimmÃ¤inen versio - Phase 0 prosessiohjeet |

---

*TÃ¤mÃ¤ dokumentti on osa Claude API -suunnittelutyÃ¶kalun prosessidokumentaatiota.*
