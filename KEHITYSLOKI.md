# KEHITYSLOKI

> **Versio:** 2.0  
> **Päivitetty:** 2025-12-01  
> **Tiedosto:** v2_0_KEHITYSLOKI.md  
> **Projekti:** Claude API -suunnittelutyökalu

---

## Projektin vaihe

```
[██████░░░░] 60% - Suunnittelu (SPEC-kerros lähes valmis)
```

**Nykyinen fokus:** Desktop Commander -integraatio valmis → META Research jatkuu.

---

## Sessiohistoria

### Sessio #12 (2025-12-01) - Desktop Commander -integraatio

**Tavoite:** Päivittää dokumentaatio Desktop Commander -työnkulkuun

**Saavutukset:**
- ✅ System Prompt v3.0 - Desktop Commander työnkulku
- ✅ INDEX v2.0 → v2.1 (yhdistetty kahden rinnakkaissession versiot)
- ✅ PROCESS_SPEC_Writing v1.6 - Desktop Commander työnkulku
- ✅ Desktop Commander Guide (full + quick) - rinnakkaissessio
- ✅ KEHITYSLOKI v1.8 → v2.0

**Kriittiset muutokset:**

| Muutos | Vanha tapa | Uusi tapa |
|--------|------------|-----------|
| Tiedostojen luku | project_knowledge_search / GitHub | read_file suoraan levyltä |
| Tiedostojen kirjoitus | Copy-paste VS Codeen | write_file suoraan levylle |
| Git-operaatiot | Claude Code promptit | start_process PowerShellissä |
| Synkronointi | GitHub → Projekti | Ei tarvita (suora pääsy) |

**Roolien uudelleenmäärittely:**

| Työkalu | Uusi rooli |
|---------|------------|
| Claude.ai | Täysi kontrolli: luku, kirjoitus, Git |
| VS Code | Valinnainen: katselu, manuaalinen editointi |
| Claude Code | 100% koodaus (ei enää Git-dokumentaatiolle) |
| GitHub | Vain versionhallinta + backup |


---

### Sessio #11 (2025-12-01)

**Tavoite:** META Research - Testing Strategy (Area 2.2)

**Saavutukset:**
- ✅ RESEARCH_Testing_Strategy.md v1.0
- ✅ PROCESS_Testing.md v1.0
- ✅ RESEARCH_META v1.1 → v1.2 (Section 7: Open Questions lisätty)
- ✅ META_Research_Navigator.md (uusi navigointityökalu)

**Keskeiset löydökset:**

| Löydös | Vaikutus |
|--------|----------|
| Hypoteesi validoitu: SPEC=WHAT, CODE=HOW | Testausstrategia selkeä |
| TDD-sykli: RGRC (Red-Green-Refactor-Commit) | Koodausprosessi selvä |
| Testijakauma: 40/45/10/5 (unit/int/e2e/static) | Pytest-konfiguraatio selvä |

---

### Sessio #10 (2025-11-30)

**Tavoite:** META Research suunnittelu - AI Development Process

**Saavutukset:**
- ✅ RESEARCH_META_AI_Development_Process.md v1.0
- ✅ META_Development_Model.md v1.0 (geneerisyysperiaatteet)
- ✅ v2_1_AI-avusteinen_ohjelmistosuunnittelu.md (päivitys)
- ✅ 9 tutkimusaluetta määritelty (P1-P3)

---

### Sessiot #1-#9 (2025-11-25 – 2025-11-26)

| Sessio | Pääsaavutus |
|--------|-------------|
| #9 | Phase 0: PROCESS_Market_Research, System Prompt v2.4 |
| #8 | RESEARCH_03 + SPEC_03 ContextManager |
| #7 | TECH_RESEARCH_02 + TECH_SPEC_02 MemoryService |
| #6 | TECH_RESEARCH-prosessin suunnittelu |
| #5 | RESEARCH_02 + SPEC_02 MemoryService |
| #4 | RESEARCH_01 + SPEC_01 päivitys |
| #3 | Prosessidokumentit |
| #2 | API-dokumentaatio, SPEC_01 |
| #1 | Projektin perustaminen |


---

## Moduulien status

| Moduuli | RESEARCH | SPEC | TECH_RES | TECH_SPEC | CODE |
|---------|:--------:|:----:|:--------:|:---------:|:----:|
| ClaudeService | ✅ | ✅ | 🔲 | 🔲 | 🔲 |
| MemoryService | ✅ | ✅ | ✅ | ✅ | 🔲 |
| ContextManager | ✅ | ✅ | 🔲 | 🔲 | 🔲 |
| SessionManager | 🔲 | 🔲 | 🔲 | 🔲 | 🔲 |
| Chat UI | 🔲 | 🔲 | 🔲 | 🔲 | 🔲 |

**Symbolit:** ✅ Valmis | 🔶 Työn alla | 🔲 Ei aloitettu

---

## META Research Status

```
Thorough-polku:

✅ S10 Plan → ✅ S11 Testing → ✅ S12 DC → ▶ S13 CODE → ○ S14 Debug → ○ S15 Env → 🚀 KOODAUS
```

| Sessio | Alue | Status | Output |
|--------|------|--------|--------|
| S10 | Research Plan | ✅ | RESEARCH_META v1.0 |
| S11 | 2.2 Testing Strategy | ✅ | PROCESS_Testing.md |
| S12 | Desktop Commander | ✅ | System Prompt v3.0, INDEX v2.1 |
| S13 | 2.3 CODE Phase | ▶ Seuraava | PROCESS_Code.md |
| S14 | 2.5 Debugging | ○ | PROCESS_Debugging.md |
| S15 | 2.4 Coding Environment | ○ | GUIDE_Coding_Environment.md |
| S16 | 2.1 Process Model | ○ | PROCESS_Light_Mode.md |

---

## Seuraavat askeleet

### Session #13 (seuraava)

1. **Research Area 2.3: CODE Phase Process**
   - Miten koodausvaihe etenee step-by-step?
   - Commit-käytännöt (koko, viestit, frekvenssi)
   - Code review -prosessi
   - Definition of Done
   - Output: PROCESS_Code.md

### Myöhemmin

- S14: Debugging Protocol (2.5)
- S15: Coding Environment (2.4)
- S16: Process Model Comparison (2.1)
- S17: 🚀 MemoryService koodaus alkaa


---

## Avoimet kysymykset

### Ratkaistu ✅

| Kysymys | Ratkaisu | Sessio |
|---------|----------|--------|
| 1M konteksti Opukselle? | EI - vain Sonnet 4/4.5 | #2 |
| SPEC-prosessi? | 11-vaiheinen (Phase 0 mukana) | #7→#9 |
| MemoryService teknologiat? | aiosqlite, Raw SQL, oma chunker | #7 |
| Dokumenttien viittaukset? | EI viittauksia edellisiin versioihin | #7 |
| Frontend-teknologia? | React | #7→#8 |
| ContextManager token-laskenta? | tiktoken (cl100k_base) | #8 |
| Testausstrategia? | SPEC=WHAT, CODE=HOW, TDD/RGRC | #11 |
| Testijakauma? | 40% unit, 45% int, 10% e2e, 5% static | #11 |
| **Työnkulku dokumentaatiolle?** | Desktop Commander (read/write/git) | #12 |

### Avoin 🔲

| Kysymys | Status | Prioriteetti |
|---------|--------|--------------|
| Hosting-ratkaisu? | Local MVP | P3 |
| CI/CD pipeline? | Tutkittava | P2 |
| Coding environment? | CC vs Copilot vs... | P1 (S15) |

*Täysi Open Questions lista: RESEARCH_META Section 7*

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 2.0 | 2025-12-01 | **Desktop Commander -integraatio**: Session #12, System Prompt v3.0, INDEX v2.1, PROCESS v1.6 |
| 1.8 | 2025-12-01 | Session #10-#11, META Research, testausstrategia |
| 1.7 | 2025-11-26 | Session #8-#9, UTF-8 korjaus |
| 1.5 | 2025-11-25 | Session #6-#7, TECH_RESEARCH + TECH_SPEC_02 |
| 1.4 | 2025-11-25 | Session #5, RESEARCH_02 + SPEC_02 |
| 1.3 | 2025-11-25 | Session #4 |
| 1.0 | 2025-11-25 | Session #1 |

---

*Dokumentti on osa Claude API -suunnittelutyökalun dokumentaatiota.*
