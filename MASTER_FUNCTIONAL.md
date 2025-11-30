# MASTER_FUNCTIONAL

> **Versio:** 1.1  
> **Päivitetty:** 2025-11-25  
> **Tiedosto:** v1_1_MASTER_FUNCTIONAL.md  
> **Projekti:** Claude API -suunnittelutyökalu  
> **Tarkoitus:** Kokonaiskuva työkalun toiminnallisuuksista

---

## Projektin visio

**Oma suunnittelutyökalu** - Ratkaisu claude.ai:n 200K token konteksti-ikkunan rajoitukseen. Työkalu käyttää Claude API:a suoraan, mahdollistaen 1M token kontekstin ja älykkään muistinhallinnan.

```
┌─────────────────────────────────────────────────────────────────────┐
│                      ONGELMA JA RATKAISU                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ONGELMA                          RATKAISU                        │
│   ┌─────────────────┐              ┌─────────────────┐             │
│   │   claude.ai     │              │  Oma työkalu    │             │
│   │   200K tokenia  │              │  1M tokenia     │             │
│   │   Ei muistia    │              │  Ulkoinen muisti│             │
│   │   Infinite epä- │              │  Hallittu       │             │
│   │   luotettava    │              │  konteksti      │             │
│   └─────────────────┘              └─────────────────┘             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Toiminnallisuuksien yleiskatsaus

### Statussymbolit

| Symboli | Merkitys |
|---------|----------|
| ✅ | SPEC valmis |
| 🔶 | Työn alla |
| 🔲 | Ei aloitettu |
| 📋 | Backlogissa |

### Moduulikartta (päivitetty v1.1)

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                         YDINTOIMINNOT                               │
│                                                                     │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│   │ CLAUDE API  │    │   CHAT UI   │    │  ULKOINEN   │            │
│   │ INTEGRAATIO │    │   (Web)     │    │   MUISTI    │            │
│   │     ✅      │    │     🔲      │    │     ✅      │            │
│   │  SPEC_01    │    │   SPEC_05   │    │  SPEC_02    │            │
│   └─────────────┘    └─────────────┘    └─────────────┘            │
│                                                                     │
│                         KONTEKSTINHALLINTA                          │
│                                                                     │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│   │  CONTEXT    │    │   ÄLYKÄS    │    │  SESSION-   │            │
│   │  MANAGER    │    │  TIIVISTYS  │    │  HALLINTA   │            │
│   │     🔲      │    │     🔲      │    │     🔲      │            │
│   │  SPEC_03    │    │   (P2)      │    │  SPEC_04    │            │
│   └─────────────┘    └─────────────┘    └─────────────┘            │
│                                                                     │
│                         INTEGRAATIOT                                │
│                                                                     │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│   │   GITHUB    │    │   GOOGLE    │    │   TIEDOSTO- │            │
│   │             │    │   DRIVE     │    │   JÄRJEST.  │            │
│   │     📋      │    │     📋      │    │     🔲      │            │
│   └─────────────┘    └─────────────┘    └─────────────┘            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Moduulien dokumentaatiostatus

| Moduuli | RESEARCH | SPEC | TECH_SPEC | Koodi | Huom |
|---------|----------|------|-----------|-------|------|
| **ClaudeService** | ✅ v1.0 | ✅ v1.1 | 🔲 | 🔲 | Gemini reviewed |
| **MemoryService** | ✅ v1.0 | ✅ v1.1 | 🔲 | 🔲 | Gemini reviewed |
| ContextManager | 🔲 | 🔲 | 🔲 | 🔲 | Seuraava |
| SessionManager | 🔲 | 🔲 | 🔲 | 🔲 | |
| Chat UI | 🔲 | 🔲 | 🔲 | 🔲 | |

---

## 1. Ydintoiminnot

### 1.1 Claude API -integraatio ✅

```
┌────────────────────────────────────────────────────────────────────┐
│ CLAUDE API -INTEGRAATIO                                 SPEC_01   │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ Kuvaus:    Suora yhteys Claude API:in, ohittaen claude.ai:n       │
│            rajoitukset. Tuki 1M token kontekstille.               │
│                                                                    │
│ Ominaisuudet:                                                      │
│   • Mallin valinta: Sonnet 4 / Sonnet 4.5 (1M tuki)              │
│   • 1M konteksti (beta-header: context-1m-2025-08-07)            │
│   • Streaming-vastaukset + error event -käsittely                 │
│   • Circuit Breaker -resilienssi                                  │
│   • Prompt Caching (90% säästö)                                   │
│   • Token-laskenta (count_tokens API)                             │
│                                                                    │
│ Dokumentaatio:                                                     │
│   • RESEARCH_01_Claude_API_Patterns.md                            │
│   • SPEC_01_Claude_Service.md v1.1                                │
│                                                                    │
│ Status:       ✅ SPEC valmis (Gemini reviewed)                     │
│ Prioriteetti: P1 (MVP)                                            │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 1.2 Ulkoinen muisti (MemoryService) ✅

```
┌────────────────────────────────────────────────────────────────────┐
│ ULKOINEN MUISTI (MemoryService)                         SPEC_02   │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ Kuvaus:    Pysyvä muisti joka säilyy sessioiden välillä.          │
│            Hybridi-arkkitehtuuri: Markdown + SQLite-vec.          │
│                                                                    │
│ Rakenne:                                                           │
│   /memory/                                                         │
│   ├── index.db           (SQLite-vec: embeddings, metadata)       │
│   ├── decisions/         (arkkitehtuuripäätökset)                 │
│   ├── summaries/         (sessioyhteenvedot)                      │
│   └── context/           (aktiivisen session konteksti)           │
│                                                                    │
│ Ominaisuudet:                                                      │
│   • Markdown-tiedostot (ihmisen luettavissa, Git-yhteensopiva)    │
│   • Semantic search (Ollama nomic-embed-text, 768 dim)            │
│   • Parent-Child chunking (pitkät dokumentit)                     │
│   • Kategoriaperusteinen salience decay                           │
│   • Graceful degradation (fallback keyword-hakuun)                │
│                                                                    │
│ Dokumentaatio:                                                     │
│   • RESEARCH_02_Memory_Architecture.md                            │
│   • SPEC_02_Memory_Service.md v1.1                                │
│                                                                    │
│ Status:       ✅ SPEC valmis (Gemini reviewed)                     │
│ Prioriteetti: P1 (MVP)                                            │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 1.3 Chat UI (Web) 🔲

```
┌────────────────────────────────────────────────────────────────────┐
│ CHAT UI                                                 SPEC_05   │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ Kuvaus:    Web-pohjainen käyttöliittymä keskusteluun Clauden      │
│            kanssa. Markdown-tuki, tiedostojen hallinta.           │
│                                                                    │
│ Ominaisuudet:                                                      │
│   • Chat-ikkuna (Markdown-renderöinti)                            │
│   • Projektin tiedostot (sivupalkki)                              │
│   • Muistin sisältö (näkyvissä, muokattavissa)                    │
│   • Token-laskuri (reaaliaikainen)                                │
│   • Mallin valinta + effort-säätö                                 │
│   • Keskusteluhistoria                                            │
│                                                                    │
│ Tekniset vaatimukset:                                              │
│   • React / Vue / Svelte (valittava)                              │
│   • WebSocket streaming                                           │
│   • Responsiivinen design                                         │
│                                                                    │
│ Riippuvuudet: Claude API -integraatio                             │
│ Status:       🔲 Ei aloitettu                                      │
│ Prioriteetti: P1 (MVP)                                            │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 2. Kontekstinhallinta

### 2.1 Context Manager 🔲

```
┌────────────────────────────────────────────────────────────────────┐
│ CONTEXT MANAGER                                         SPEC_03   │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ Kuvaus:    Kontekstin rakentaminen ja token-budjetin hallinta.    │
│            Yhdistää MemoryServicen ja ClaudeServicen.             │
│                                                                    │
│ Ominaisuudet:                                                      │
│   • Token-laskenta ja budjetointi                                 │
│   • Relevantin kontekstin valinta (MemoryService.search())        │
│   • Kontekstin priorisointi ja karsinta                           │
│   • Prompt injection -suojaus (<memory> tagit)                    │
│                                                                    │
│ Riippuvuudet: MemoryService, ClaudeService                        │
│ Status:       🔲 Ei aloitettu (SEURAAVA)                          │
│ Prioriteetti: P1 (MVP)                                            │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 2.2 Älykäs tiivistys 🔲

```
┌────────────────────────────────────────────────────────────────────┐
│ ÄLYKÄS TIIVISTYS                                        SPEC_04   │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ Kuvaus:    Automaattinen kontekstin tiivistäminen kun token-      │
│            budjetti täyttyy.                                       │
│                                                                    │
│ Ominaisuudet:                                                      │
│   • Token threshold trigger (50K)                                 │
│   • Session summarization (ClaudeService kautta)                  │
│   • Manuaalinen override                                          │
│                                                                    │
│ Riippuvuudet: MemoryService, ClaudeService                        │
│ Status:       🔲 Ei aloitettu                                      │
│ Prioriteetti: P2                                                  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 2.3 Session-hallinta 🔲

```
┌────────────────────────────────────────────────────────────────────┐
│ SESSION-HALLINTA (SessionManager)                       SPEC_06   │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ Kuvaus:    Sessioiden tallennus, lataus ja jatkaminen.            │
│            Orkestroi MemoryService + ClaudeService.               │
│                                                                    │
│ Ominaisuudet:                                                      │
│   • Tallenna sessio (koko keskustelu + konteksti)                 │
│   • Lataa vanha sessio                                            │
│   • Jatka keskeytettyä sessiota                                   │
│   • Sessioiden listaus ja haku                                    │
│   • Automaattinen tallennus (5 min välein)                        │
│   • Session summarization orchestration                           │
│                                                                    │
│ Riippuvuudet: MemoryService, ClaudeService                        │
│ Status:       🔲 Ei aloitettu                                      │
│ Prioriteetti: P1 (MVP)                                            │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 3. Integraatiot

### 3.1 Tiedostojärjestelmä 🔲

```
┌────────────────────────────────────────────────────────────────────┐
│ TIEDOSTOJÄRJESTELMÄ                                     SPEC_07   │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ Kuvaus:    Paikallisten tiedostojen hallinta ja lataus            │
│            kontekstiin.                                            │
│                                                                    │
│ Ominaisuudet:                                                      │
│   • Tiedostojen upload (drag & drop)                              │
│   • Projektin tiedostorakenne                                     │
│   • Tiedostojen esikatselu                                        │
│   • Markdown, koodi, PDF -tuki                                    │
│                                                                    │
│ Status:       🔲 Ei aloitettu                                      │
│ Prioriteetti: P1 (MVP)                                            │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 3.2 GitHub-integraatio 📋

```
┌────────────────────────────────────────────────────────────────────┐
│ GITHUB-INTEGRAATIO                                      SPEC_08   │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ Kuvaus:    Yhteys GitHub-repositorioihin.                         │
│                                                                    │
│ Status:       📋 Backlogissa                                       │
│ Prioriteetti: P3                                                  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### 3.3 Google Drive -integraatio 📋

```
┌────────────────────────────────────────────────────────────────────┐
│ GOOGLE DRIVE -INTEGRAATIO                               SPEC_09   │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│ Kuvaus:    Yhteys Google Driveen dokumenttien lukemiseen.         │
│                                                                    │
│ Status:       📋 Backlogissa                                       │
│ Prioriteetti: P4                                                  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## Yhteenveto: Prioriteetit

### P1 - MVP

| Moduuli | Status | SPEC |
|---------|--------|------|
| Claude API -integraatio | ✅ SPEC valmis | SPEC_01 v1.1 |
| Ulkoinen muisti (MemoryService) | ✅ SPEC valmis | SPEC_02 v1.1 |
| Context Manager | 🔲 Seuraava | SPEC_03 |
| Session-hallinta | 🔲 | SPEC_06 |
| Chat UI | 🔲 | SPEC_05 |
| Tiedostojärjestelmä | 🔲 | SPEC_07 |

### P2 - Parannettu kontekstinhallinta

| Moduuli | Status |
|---------|--------|
| Älykäs tiivistys | 🔲 |

### P3-P4 - Myöhemmin

| Moduuli | Status |
|---------|--------|
| GitHub-integraatio | 📋 |
| Google Drive | 📋 |

---

## Teknologiavalinnat

### Vahvistetut

| Komponentti | Teknologia | Perustelu |
|-------------|------------|-----------|
| **AI API** | Claude API | Projektin ydin |
| **Backend-kieli** | Python | FastAPI, hyvä AI-ekosysteemi |
| **Muistin tallennus** | Markdown + SQLite-vec | Hybridi: luettava + haettava |
| **Embedding** | Ollama nomic-embed-text | Ilmainen, 768 dim |

### Arvioitavana

| Komponentti | Vaihtoehdot | Päätös |
|-------------|-------------|--------|
| **Backend-framework** | FastAPI | 🔲 Vahvistettava |
| **Frontend** | React, Vue, Svelte | 🔲 |
| **Hosting** | Local, VPS, Cloud | 🔲 |

---

## Dokumenttiviittaukset

### GitHub (SPEC/RESEARCH dokumentit)

| Dokumentti | Polku |
|------------|-------|
| RESEARCH_01 | docs/research/RESEARCH_01_Claude_API_Patterns.md |
| RESEARCH_02 | docs/research/RESEARCH_02_Memory_Architecture.md |
| SPEC_01 | docs/specs/SPEC_01_Claude_Service.md |
| SPEC_02 | docs/specs/SPEC_02_Memory_Service.md |

### Projekti (aina kontekstissa)

| Dokumentti | Sisältö |
|------------|---------|
| KEHITYSLOKI | Päätökset ja edistyminen |
| ARCHITECTURE_OVERVIEW | Tekninen arkkitehtuuri |
| API_REFERENCE | Claude API dokumentaatio |
| VS_CODE_WORKFLOW | Git-työnkulku |

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.1 | 2025-11-25 | MemoryService SPEC valmis, moduulikartta päivitetty |
| 1.0 | 2025-11-25 | Ensimmäinen versio |

---

*Dokumentti on osa Claude API -suunnittelutyökalun dokumentaatiota.*
