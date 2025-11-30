# KEHITYSLOKI

> **Versio:** 1.7  
> **PÃ¤ivitetty:** 2025-11-26  
> **Tiedosto:** v1_7_KEHITYSLOKI.md  
> **Projekti:** Claude API -suunnittelutyÃ¶kalu

---

## Projektin vaihe

```
[â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘â–‘] 60% - Suunnittelu (SPEC-kerros lÃ¤hes valmis)
```

**Nykyinen fokus:** Viimeiset SPECit (SessionManager, Chat UI) + koodauksen aloitus.

---

## Sessiohistoria

### Sessio #9 kÃ¤ynnissÃ¤ (2025-11-26)

**Tavoite:** Dokumenttien siivous + koodauksen aloitus

**AamupÃ¤ivÃ¤ â€” Siivous:**
1. âœ… INDEX v1.10 â†’ v1.11 (Session #8 tulokset)
2. âœ… KEHITYSLOKI v1.6 â†’ v1.7 (UTF-8 korjaus + Session #8)

**IltapÃ¤ivÃ¤ â€” Suunnitelma:**
- Vaihtoehto A: SPEC_04 SessionManager
- Vaihtoehto B: MemoryService koodaus

---

### Sessio #8 (2025-11-25)

**Tavoite:** ContextManager SPEC + Gemini review

**Saavutukset:**
- âœ… RESEARCH_03_Context_Manager.md v1.0
- âœ… SPEC_03_Context_Manager.md v1.0 (Gemini reviewed & hyvÃ¤ksytty)
- âœ… PROCESS_SPEC_Writing v1.3 â†’ v1.4 (vaiheistus + Gemini-tarkistuslista)
- âœ… GUIDE_Extended_Thinking_Usage.md v1.0 (uusi dokumenttityyppi)

**ContextManager keskeiset pÃ¤Ã¤tÃ¶kset:**

| PÃ¤Ã¤tÃ¶s | Ratkaisu | Perustelu |
|--------|----------|-----------|
| Token-laskenta | tiktoken (cl100k_base) | Claude-yhteensopiva |
| Kontekstin priorisointi | Kategoriapohjaiset painot | SYSTEM > MEMORY > USER > HISTORY |
| Tiivistys | Threshold-pohjainen (50K) | Automaattinen + manuaalinen |

**Gemini Review (SPEC_03):**
- HyvÃ¤ksytty sellaisenaan
- PieniÃ¤ tarkennuksia prioriteettijÃ¤rjestykseen

---

### Sessio #7 (2025-11-25)

**Tavoite:** TECH_RESEARCH-prosessin luominen + TECH_SPEC_02 viimeistely

**Saavutukset:**
- âœ… TECH_RESEARCH-vaihe lisÃ¤tty prosessiin (uusi dokumenttityyppi)
- âœ… PROCESS_SPEC_Writing v1.2 â†’ v1.3 (10-vaiheinen prosessi)
- âœ… System Prompt v2.2 â†’ v2.3 (dokumenttien itsenÃ¤isyys -sÃ¤Ã¤ntÃ¶)
- âœ… TECH_RESEARCH_02 v1.0 (Geminin teknologiavalinnat + Claude review)
- âœ… TECH_SPEC_02 v1.0 (tÃ¤ysi toteutusmÃ¤Ã¤rittely, 1300 riviÃ¤)
- âœ… Gemini-review ja hyvÃ¤ksyntÃ¤ TECH_SPEC_02:lle

**Kriittinen prosessioppi:**

| Oppi | SÃ¤Ã¤ntÃ¶ |
|------|--------|
| **Dokumenttien itsenÃ¤isyys** | Jokainen versio PITÃ„Ã„ olla tÃ¤ydellinen - ei viittauksia edellisiin |
| **Miksi?** | Vanhat versiot poistetaan projektista â†’ viittaukset rikkoutuvat |

**TECH_RESEARCH_02 pÃ¤Ã¤tÃ¶kset (Gemini + Claude):**

| Kysymys | PÃ¤Ã¤tÃ¶s | Perustelu |
|---------|--------|-----------|
| Async SQLite | aiosqlite | Moderni async-tuki |
| SQL-abstraktio | Raw SQL | sqlite-vec yhteensopivuus |
| Chunking | Oma toteutus | Ei LangChain-riippuvuutta |
| File I/O | aiofiles | Non-blocking |
| Datamallit | Pydantic | FastAPI-integraatio |

**TECH_SPEC_02 keskeiset komponentit:**

| Komponentti | Tiedosto | Vastuu |
|-------------|----------|--------|
| MemoryService | service.py | Facade (public API) |
| SQLiteRepository | repository.py | Raw SQL + sqlite-vec |
| FileStore | file_store.py | Markdown I/O |
| EmbeddingClient | embedding_client.py | Ollama-kutsut |
| TextSplitter | text_splitter.py | Chunking |

---

### Sessio #6 (2025-11-25)

**Tavoite:** TECH_RESEARCH-prosessin suunnittelu

**Saavutukset:**
- âœ… Tunnistettu tarve TECH_RESEARCH-vaiheelle
- âœ… Prosessin suunnittelu (SPEC â†’ TECH_RESEARCH â†’ TECH_SPEC)
- âœ… Geminin 3 teknologiakysymystÃ¤ dokumentoitu

**Tunnistettu tarve:**

```
RESEARCH (toiminnallinen): "Miten muisti pitÃ¤isi toimia?"
TECH_RESEARCH (tekninen):  "MillÃ¤ kirjastoilla toteutetaan?" â† PUUTTUI
TECH_SPEC (toteutus):      "Miten koodataan?"
```

---

### Sessio #5 (2025-11-25)

**Tavoite:** RESEARCH_02 + SPEC_02 MemoryServicelle

**Saavutukset:**
- âœ… RESEARCH_02_Memory_Architecture.md v1.0 (syvÃ¤llinen tutkimus ~2h)
- âœ… SPEC_02_Memory_Service.md v1.0 â†’ v1.1 (Gemini review)
- âœ… Gemini-review toteutettu ja korjaukset tehty

**Kriittiset lÃ¶ydÃ¶kset (RESEARCH_02):**

| LÃ¶ydÃ¶s | Vaikutus |
|--------|----------|
| SQLite-vec tukee vektorihakua | Hybridi: Markdown + SQLite-vec |
| Ollama embedding ilmainen | MVP: nomic-embed-text (768 dim) |
| mem0 salience decay -pattern | Kategoriaperusteinen haalistuminen |
| Parent-Child chunking | PitkÃ¤t dokumentit â†’ useita vektoreita |

**Gemini Review (SPEC_02 v1.0 â†’ v1.1):**

| Havainto | Korjaus |
|----------|---------|
| Chunking + MemoryItem ristiriita | Parent-Child skeema |
| Salience decay sokea kategorioille | DECISIONS=1.0, SUMMARIES=0.99, CONTEXT=0.90 |
| Circular dependency | summarize_session â†’ SessionManager |

---

### Sessio #4 (2025-11-25)

**Tavoite:** Testaa SPEC-prosessi pÃ¤ivittÃ¤mÃ¤llÃ¤ SPEC_01

**Saavutukset:**
- âœ… RESEARCH_01_Claude_API_Patterns.md v1.0
- âœ… SPEC_01_Claude_Service.md v1.0 â†’ v1.1
- âœ… PROCESS_SPEC_Writing.md v1.1 â†’ v1.2
- âœ… GitHub/Projects -integraation toiminta selvitetty

**Kriittiset lÃ¶ydÃ¶kset (RESEARCH_01):**

| LÃ¶ydÃ¶s | Vaikutus SPECiin |
|--------|------------------|
| Streaming-virhe voi tulla 200:n jÃ¤lkeen | + error event -kÃ¤sittely |
| count_tokens() on ILMAINEN API | + selvennetty dokumentaatioon |
| Circuit breaker estÃ¤Ã¤ kaskadivirheet | + uusi resilienssipattern |

---

### Sessio #3 (2025-11-25)

**Tavoite:** Prosessien kehittÃ¤minen ja dokumentointi

**Saavutukset:**
- âœ… PROCESS_SPEC_Writing.md v1.0
- âœ… AI-avusteinen_ohjelmistosuunnittelu.md v2.0
- âœ… INDEX.md v1.6
- âœ… System Prompt v2.1

---

### Sessio #2 (2025-11-25)

**Tavoite:** API-dokumentaatio ja SPEC-aloitus

**Saavutukset:**
- âœ… SPEC_01_Claude_Service.md v1.0
- âœ… API_REFERENCE.md v1.0
- âœ… INDEX.md

---

### Sessio #1 (2025-11-25)

**Tavoite:** Projektin perustaminen

**Saavutukset:**
- âœ… Claude-projekti perustettu
- âœ… Master prompt kirjoitettu
- âœ… MASTER_FUNCTIONAL.md
- âœ… ARCHITECTURE_OVERVIEW.md
- âœ… Arkkitehtuurin primitiivit tunnistettu

---

## Moduulien status

| Moduuli | RESEARCH | SPEC | TECH_RES | TECH_SPEC | CODE |
|---------|:--------:|:----:|:--------:|:---------:|:----:|
| ClaudeService | âœ… | âœ… | ðŸ”² | ðŸ”² | ðŸ”² |
| MemoryService | âœ… | âœ… | âœ… | âœ… | ðŸ”² |
| ContextManager | âœ… | âœ… | ðŸ”² | ðŸ”² | ðŸ”² |
| SessionManager | ðŸ”² | ðŸ”² | ðŸ”² | ðŸ”² | ðŸ”² |
| Chat UI | ðŸ”² | ðŸ”² | ðŸ”² | ðŸ”² | ðŸ”² |

**Symbolit:** âœ… Valmis | ðŸ”¶ TyÃ¶n alla | ðŸ”² Ei aloitettu

---

## Seuraavat askeleet

### Session #9 (2025-11-26)

1. **SPEC_04_Session_Manager** â€” Sessioiden tallennus ja lataus
2. **SPEC_05_Chat_UI** â€” Kevyt luonnos React-pohjaisesta UI:sta
3. **MemoryService koodaus** â€” TECH_SPEC_02 pohjalta

### MyÃ¶hemmin

- TECH_RESEARCH_01 + TECH_SPEC_01 (ClaudeService)
- TECH_RESEARCH_03 + TECH_SPEC_03 (ContextManager)
- TECH_RESEARCH_04 + TECH_SPEC_04 (SessionManager)
- Frontend-toteutus

---

## Avoimet kysymykset

### Ratkaistu âœ…

| Kysymys | Ratkaisu | Sessio |
|---------|----------|--------|
| 1M konteksti Opukselle? | EI - vain Sonnet 4/4.5 | #2 |
| SPEC-prosessi? | 10-vaiheinen (TECH_RESEARCH mukana) | #7 |
| MemoryService teknologiat? | aiosqlite, Raw SQL, oma chunker | #7 |
| Dokumenttien viittaukset? | EI viittauksia edellisiin versioihin | #7 |
| Frontend-teknologia? | React | #7â†’#8 |
| Koodauksen aloitus? | KyllÃ¤, Session #8/#9 | #7â†’#8 |
| ContextManager token-laskenta? | tiktoken (cl100k_base) | #8 |

### Avoin ðŸ”²

| Kysymys | Status | Prioriteetti |
|---------|--------|--------------|
| Hosting-ratkaisu? | Local MVP, tarkemmin TECH_RESEARCH:ssa | P3 |

---

## Muutoshistoria

| Versio | PÃ¤ivÃ¤mÃ¤Ã¤rÃ¤ | Muutokset |
|--------|------------|-----------|
| 1.7 | 2025-11-26 | Session #8 tulokset (RESEARCH_03 + SPEC_03), UTF-8 encoding korjattu, moduulistatus pÃ¤ivitetty |
| 1.6 | 2025-11-25 | Session #8 suunnitelma, React-pÃ¤Ã¤tÃ¶s, koodauksen aloitus pÃ¤Ã¤tetty |
| 1.5 | 2025-11-25 | Session #6-#7, TECH_RESEARCH + TECH_SPEC_02 valmis |
| 1.4 | 2025-11-25 | Session #5, RESEARCH_02 + SPEC_02 v1.1 |
| 1.3 | 2025-11-25 | Session #4, RESEARCH_01 + SPEC_01 v1.1 |
| 1.2 | 2025-11-25 | Session #3, prosessidokumentit |
| 1.1 | 2025-11-25 | Session #2, API-dokumentaatio |
| 1.0 | 2025-11-25 | Session #1, projektin perustaminen |

---

*Dokumentti on osa Claude API -suunnittelutyÃ¶kalun dokumentaatiota.*
