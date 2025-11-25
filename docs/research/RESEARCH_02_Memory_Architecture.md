# RESEARCH_02: Memory Architecture for LLM Applications

> **Versio:** 1.0  
> **Päivitetty:** 2025-11-25  
> **Liittyy:** SPEC_02_Memory_Service  
> **Tiedonhaun taso:** Syvällinen (~2h)  
> **Status:** ✅ Valmis

---

## Tutkimuskysymykset

1. **Mikä tallennusmuoto on paras pitkäaikaiselle muistille?** (MD vs JSON vs SQLite vs Vector DB)
2. **Miten semantic search toteutetaan kustannustehokkaasti?** (Embeddings, RAG patterns)
3. **Miten olemassa olevat ratkaisut toimivat?** (mem0, LangChain Memory, Obsidian)
4. **Mikä on optimaalinen muisti-arkkitehtuuri LLM-sovelluksille?** (Akateeminen tutkimus)
5. **Mitä edge caseja ja sudenkuoppia on?** (Tuotantokokemukset)

---

## Löydökset

### Kysymys 1: Tallennusmuotojen vertailu

#### Markdown + Tiedostojärjestelmä

**Edut:**
- Ihmisen luettavissa suoraan
- Git-yhteensopiva (versionhallinta "ilmaiseksi")
- Ei riippuvuuksia - toimii millä tahansa editorilla
- Obsidian/VS Code -ekosysteemi

**Haitat:**
- Ei natiivisti tue semantic search
- Haku vaatii embeddings-lisäyksen
- Metadatan hallinta manuaalista (YAML frontmatter)

**Käyttötapaukset:** Pitkäaikainen arkistointi, ihmisen tarkasteltavat dokumentit

---

#### SQLite + sqlite-vec

**Löydös:** SQLite-vec on erittäin lupaava ratkaisu MVP:lle!

**Edut:**
- Yksi tiedosto - helppo siirtää, varmuuskopioida
- sqlite-vec -laajennus tuo vector search:in
- Sub-millisekunnin hakuajat
- Ei palvelinta tarvita
- Python-tuki erinomainen

**Haitat:**
- Vaatii sqlite-vec laajennuksen asentamisen
- Ei ole "ihmisen luettavissa" kuten markdown
- Concurrent write -rajoitukset (WAL-mode auttaa)

**Tekninen toteutus (sqlite-vec):**
```sql
-- Vektorihaku SQLitessä
CREATE VIRTUAL TABLE vec_memories USING vec0(
  embedding float[768]
);

-- KNN-haku
SELECT rowid, distance 
FROM vec_memories 
WHERE embedding MATCH '[0.1, 0.2, ...]'
ORDER BY distance 
LIMIT 5;
```

**Lähde:** https://github.com/asg017/sqlite-vec

---

#### Dedicated Vector DB (Qdrant, Milvus, FAISS)

**Edut:**
- Optimoitu miljardien vektorien käsittelyyn
- HNSW, IVF ja muut tehokkaat indeksit
- Skaalautuu horisontaalisesti

**Haitat:**
- Ylimääräinen infrastruktuuri
- Overkill pienille dataseteille (<100K dokumenttia)
- Lisää monimutkaisuutta

**Soveltuvuus MVP:lle:** ❌ Ei suositeltu - liian raskas

---

#### Hybridi: Markdown tallennukseen + SQLite/Vector hakuun

**Paras molemmista maailmoista:**
```
┌─────────────────────────────────────────────────────┐
│                  HYBRIDI-ARKKITEHTUURI              │
├─────────────────────────────────────────────────────┤
│                                                     │
│   /memory/                     SQLite Index         │
│   ├── decisions/               ┌──────────────┐    │
│   │   └── *.md        ──────►  │ embeddings   │    │
│   ├── summaries/               │ metadata     │    │
│   │   └── *.md        ──────►  │ file_paths   │    │
│   └── context/                 └──────────────┘    │
│       └── *.md                        │            │
│                                       │            │
│   Ihmisen luettavissa          Nopea haku          │
│   Git-versionhallinta          Semantic search     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Suositus MVP:lle:** ✅ Hybridi (Markdown + SQLite-vec)

---

### Kysymys 2: Semantic Search ja Embeddings

#### Embedding-mallien vertailu (2024-2025)

| Malli | Dimensiot | Hinta/1M tokenia | Suorituskyky | Huom |
|-------|-----------|------------------|--------------|------|
| **OpenAI text-embedding-3-small** | 1536 | $0.02 | 75-80% | Paras hinta/laatu |
| OpenAI text-embedding-3-large | 3072 | $0.13 | 80-85% | Premium |
| **Voyage-3-lite** | 512 | $0.02 | 66% | Pieni dimensio |
| Voyage-3 | 1024 | $0.06 | 77% | Hyvä kompromissi |
| **Ollama (local)** | 768-1024 | $0 | 70-75% | Ei kustannuksia |
| Mistral-embed | 1024 | ~$0.10 | 78% | Paras tarkkuus |

**Suositus:**
- **MVP:** Ollama (nomic-embed-text tai mxbai-embed-large) - ei kustannuksia
- **Tuotanto:** OpenAI text-embedding-3-small - paras hinta/laatu

**Kriittinen löydös:** Paikallinen embedding-malli (Ollama) toimii riittävän hyvin MVP:lle ja poistaa API-kustannukset kokonaan!

---

#### RAG-arkkitehtuurin tasot

```
┌─────────────────────────────────────────────────────┐
│                 RAG KYPSYYSTASOT                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  TASO 1: Naive RAG (MVP)                           │
│  ├── Yksinkertainen vector search                  │
│  ├── Top-K dokumenttien haku                       │
│  └── Suora syöttö LLM:lle                          │
│                                                     │
│  TASO 2: Advanced RAG                              │
│  ├── Query rewriting                               │
│  ├── Hybrid search (keyword + semantic)            │
│  └── Re-ranking                                    │
│                                                     │
│  TASO 3: Modular RAG                               │
│  ├── Multi-hop retrieval                           │
│  ├── Memory modules                                │
│  └── Task adapters                                 │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**MVP:n taso:** TASO 1 (Naive RAG) riittää alkuun

---

### Kysymys 3: Olemassa olevien ratkaisujen analyysi

#### mem0 (mem0.ai)

**Mitä se on:** "Universal memory layer for AI agents"

**Arkkitehtuuri:**
- **Hybrid storage:** Key-Value + Graph + Vector
- **Dual-phase pipeline:** Extract facts → Compare with existing → Add/Update/Delete
- **Graph variant (Mem0ᵍ):** Entity nodes + relationship edges

**Tulokset (LOCOMO benchmark):**
- 26% parempi tarkkuus kuin OpenAI Memory
- 91% pienempi p95 latenssi
- 90% token-säästö

**Kriittiset opit:**
1. **LLM-powered memory management** - LLM päättää mitä tallennetaan/poistetaan
2. **Conflict detection** - tunnistaa ristiriitaiset muistit
3. **Salience filtering** - ei tallenna kaikkea, vain tärkeät faktat

**Soveltuvuus meidän projektiin:**
- ✅ Arkkitehtuuriperiaatteet sovellettavissa
- ❌ Täysi mem0 liian raskas MVP:lle
- ✅ Voimme implementoida yksinkertaistetun version

---

#### LangChain Memory

**Muistityypit:**

| Tyyppi | Kuvaus | Token-käyttö | Soveltuvuus |
|--------|--------|--------------|-------------|
| ConversationBufferMemory | Tallentaa kaiken | Korkea | Lyhyet keskustelut |
| ConversationBufferWindowMemory | Viimeiset K viestit | Keskitaso | Keskipitkät |
| ConversationSummaryMemory | Tiivistää vanhan | Alhainen | Pitkät keskustelut |
| **ConversationSummaryBufferMemory** | Hybridi | Optimoitu | **Suositus** |

**Kriittinen havainto:** LangChain memory on DEPRECATED (0.3.x), korvattu RunnableWithMessageHistory + LangGraph

**Soveltuvuus:**
- Konseptit relevantteja
- Ei suoraa riippuvuutta LangChainiin MVP:ssä
- Toteutamme omat versiot

---

#### Obsidian + Semantic Search

**Miten se toimii:**
1. Markdown-tiedostot local vaultissa
2. Plugin (Smart Connections, CoPilot) indeksoi
3. Vector embeddings luodaan taustalla
4. Semantic search kyselyillä

**Opit:**
- Markdown toimii hyvin pohjana
- Embeddings voidaan luoda taustalla
- Käyttäjä ei näe vektoreita - vain hakee

---

### Kysymys 4: Optimaalinen muistiarkkitehtuuri

#### Kolme muistityyppiä (mem0:n malli)

```
┌─────────────────────────────────────────────────────┐
│               MUISTITYYPPIEN HIERARKIA              │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. SHORT-TERM MEMORY (Konteksti-ikkuna)           │
│     ├── Viimeiset N viestiä                        │
│     ├── Elää LLM:n context windowissa             │
│     └── Ei persistenssiä                           │
│                                                     │
│  2. WORKING MEMORY (Session)                       │
│     ├── Tämän session konteksti                    │
│     ├── Tiivistetty historia                       │
│     └── Persistoidaan session loppuessa            │
│                                                     │
│  3. LONG-TERM MEMORY (Ulkoinen muisti)             │
│     ├── Faktamuisti (käyttäjätiedot, päätökset)   │
│     ├── Episodinen muisti (aiemmat sessiot)        │
│     └── Semanttinen muisti (konseptit, linkit)     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

#### Muistin elinkaari

```
SESSION ALKAA
     │
     ▼
[Load relevant long-term memories]
     │
     ▼
[Initialize working memory]
     │
     ▼
┌─────────────────────────────────┐
│         SESSION LOOP            │
│                                 │
│  User input                     │
│       ▼                         │
│  [Query long-term memory]       │
│       ▼                         │
│  [Build context with memories]  │
│       ▼                         │
│  [Send to LLM]                  │
│       ▼                         │
│  [Extract facts from response]  │
│       ▼                         │
│  [Update working memory]        │
│       ▼                         │
│  [Periodically save to LTM]     │
│                                 │
└─────────────────────────────────┘
     │
     ▼
SESSION PÄÄTTYY
     │
     ▼
[Summarize session]
     │
     ▼
[Save to long-term memory]
```

---

### Kysymys 5: Edge Cases ja Sudenkuopat

#### 1. Memory Bloat

**Ongelma:** Muisti kasvaa rajatta, haku hidastuu

**Ratkaisu:**
- Salience scoring (0-1) jokaiselle muistille
- Time decay - vanhat muistit "unohtuvat"
- Periodic cleanup - poista alle kynnyksen muistit

#### 2. Konfliktit

**Ongelma:** "Käyttäjä tykkää kahvista" vs "Käyttäjä ei juo kahvia"

**Ratkaisu (mem0:n malli):**
1. Conflict Detector tunnistaa ristiriidat
2. LLM-based resolver päättää:
   - ADD (uusi fakta)
   - UPDATE (korvaa vanha)
   - DELETE (poista vanha)
   - SKIP (älä tallenna)

#### 3. Hallusinaatiot muistista

**Ongelma:** LLM "muistaa" asioita jotka eivät ole muistissa

**Ratkaisu:**
- Eksplisiittinen citation: "Based on memory item #X..."
- Confidence scoring haun tuloksille
- Selkeä ero haetun vs generoidun välillä

#### 4. Token overflow

**Ongelma:** Liian paljon muisteja konteksti-ikkunaan

**Ratkaisu:**
- Top-K rajaus (esim. max 5 relevanttia muistia)
- Summarization vanhemmille muisteille
- Chunking pitkille dokumenteille

---

## Suositukset SPECiin

### MVP-arkkitehtuuri

| Päätös | Suositus | Perustelu |
|--------|----------|-----------|
| **Tallennusmuoto** | Hybridi: Markdown + SQLite-vec | Ihmisen luettavissa + nopea haku |
| **Embedding-malli** | Ollama (local) MVP:ssä | Ei kustannuksia, riittävä laatu |
| **Hakustrategia** | Naive RAG (Top-K) | Yksinkertainen, toimii |
| **Muistityypit** | Short + Long (ei working memory MVP:ssä) | Vähemmän kompleksisuutta |
| **Konfliktien hallinta** | Manuaalinen MVP:ssä | LLM-resolver myöhemmin |

### Arkkitehtuurikaavio (MVP)

```
┌─────────────────────────────────────────────────────────────┐
│                    MEMORY SERVICE (MVP)                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                    PUBLIC API                        │   │
│  │  ────────────────────────────────────────────────── │   │
│  │  save(content, metadata) → MemoryItem               │   │
│  │  search(query, limit) → List[MemoryItem]            │   │
│  │  get(id) → MemoryItem                               │   │
│  │  delete(id) → bool                                  │   │
│  │  summarize_session() → Summary                      │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                                │
│                            ▼                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │               INTERNAL COMPONENTS                    │   │
│  │                                                      │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌───────────┐ │   │
│  │  │ EmbeddingGen │  │ SearchEngine │  │ FileStore │ │   │
│  │  │   (Ollama)   │  │ (SQLite-vec) │  │ (Markdown)│ │   │
│  │  └──────────────┘  └──────────────┘  └───────────┘ │   │
│  │                                                      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Myöhemmät laajennukset

1. **Prompt caching** - cache embeddings
2. **Hybrid search** - keyword + semantic
3. **Conflict resolver** - LLM-powered
4. **Graph storage** - entity relationships (Mem0ᵍ tyyliin)
5. **Re-ranking** - parempi relevanssi

---

## Avoimet kysymykset (→ Käyttäjälle)

### 1. Embedding-malli MVP:lle

| Vaihtoehto | Kustannus | Laatu | Suositus |
|------------|-----------|-------|----------|
| A) Ollama (local) | $0 | ~70% | ✅ **Suositus** |
| B) OpenAI small | $0.02/M | ~80% | Harkittava |
| C) Molemmat (fallback) | Vaihtelee | Paras | Monimutkaisempi |

**Ehdotus:** Aloitetaan Ollamalla, lisätään OpenAI myöhemmin vaihtoehdon C mukaisesti.

### 2. Muistien kategorisointi

Pitäisikö muistit jaotella kategorioihin (MASTER_FUNCTIONAL:n mukaan)?

```
/memory/
├── decisions/      # Arkkitehtuuripäätökset
├── summaries/      # Sessioyhteenvedot  
├── specs/          # SPEC-dokumentit
├── facts/          # Käyttäjätiedot, preferenssit (NEW?)
└── context/        # Aktiivisen session konteksti
```

**Ehdotus:** Kyllä, mutta vain 3 kategoriaa MVP:ssä:
- `decisions/` - päätökset
- `summaries/` - sessioyhteenvedot
- `context/` - aktiivinen sessio

### 3. Session summarization trigger

Milloin session yhteenveto luodaan?
- A) Session lopussa manuaalisesti
- B) Automaattisesti kun session ikä > X minuuttia
- C) Kun token count > kynnys (esim. 50K)

**Ehdotus:** Vaihtoehto C (token-pohjainen) + A (manuaalinen vaihtoehto)

---

## Lähteet

1. mem0.ai - Memory Architecture Research (2025)
2. Anthropic RAG Patterns (IBM Architecture Center)
3. LangChain Memory Documentation (deprecated)
4. sqlite-vec GitHub (Alex Garcia)
5. Voyage AI Embedding Benchmarks (2024)
6. OpenAI Embedding Pricing (2025)
7. Obsidian Smart Connections Plugin

---

## Lisälöydökset: Chunking-strategiat

### Chunk-koon vaikutus

**NVIDIA 2024 Benchmark tulokset:**

| Strategia | Tarkkuus | Hajonta | Huom |
|-----------|----------|---------|------|
| **Page-level** | 0.648 | 0.107 | Paras strukturoiduille dokumenteille |
| Semantic | ~0.63 | Korkeampi | Parempi vapaamuotoiselle tekstille |
| Fixed 512 | ~0.60 | Keskitaso | Hyvä default |

**Optimal chunk size by query type:**

| Kyselytyyppi | Optimaalinen koko | Esimerkki |
|--------------|-------------------|-----------|
| Faktakysymykset | 256-512 tokenia | "Mikä on X:n hinta?" |
| Analyyttiset | 1024+ tokenia | "Vertaa X:ää ja Y:tä" |
| Kontekstuaaliset | 512-1024 tokenia | "Mitä päätimme X:stä?" |

**Suositus MVP:lle:**
- **Default:** 400-512 tokenia
- **Overlap:** 10-20% (50-100 tokenia)
- **Strategia:** RecursiveCharacterTextSplitter (yksinkertainen, toimii)

---

### Chunking-strategiat

```
┌─────────────────────────────────────────────────────┐
│              CHUNKING-STRATEGIAT                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. FIXED-SIZE (MVP)                               │
│     ├── Yksinkertainen: split every N tokens       │
│     ├── Overlap: 10-20% kontekstin säilyttämiseen  │
│     └── ✅ Suositus: Aloita tästä                   │
│                                                     │
│  2. RECURSIVE (MVP+)                               │
│     ├── Kunnioittaa rakenteita (headers, paragraphs)│
│     ├── Fallback pienempiin yksiköihin             │
│     └── LangChain RecursiveCharacterTextSplitter   │
│                                                     │
│  3. SEMANTIC (Myöhemmin)                           │
│     ├── Embedding-pohjainen: split kun aihe muuttuu │
│     ├── Vaatii embedding-kutsuja chunking-vaiheessa │
│     └── Parempi laatu, korkeampi kustannus         │
│                                                     │
│  4. DOCUMENT-BASED (Myöhemmin)                     │
│     ├── Markdown headers → chunks                  │
│     ├── Code blocks → chunks                       │
│     └── Säilyttää dokumentin rakenteen             │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## Lisälöydökset: Toteutusmallit

### Markdown-pohjainen muisti (Deep Agent CLI malli)

**Periaate:** Muistit ovat markdown-tiedostoja, joita AI voi lukea ja kirjoittaa.

```
/memory/agent_name/
├── project_architecture.md    # Opittu projektin rakenne
├── coding_patterns.md         # Havaitut koodipatternit
├── user_preferences.md        # Käyttäjän mieltymykset
└── decisions/
    ├── 2025-11-25_api_choice.md
    └── 2025-11-25_db_design.md
```

**Edut:**
- ✅ Läpinäkyvä - käyttäjä näkee mitä AI "muistaa"
- ✅ Siirrettävä - kopioi kansio = kopioi muisti
- ✅ Versionhallittava - Git toimii suoraan
- ✅ Ihmisen muokattavissa

**Soveltuvuus meidän projektiin:** ⭐⭐⭐⭐⭐ Erinomainen

---

### Short-term + Long-term Pattern (PocketFlow)

```python
# Shared store pattern
shared = {
    # Short-term memory (current session)
    "messages": [],
    
    # Long-term memory (persistent)
    "vector_index": None,
    "vector_items": []
}
```

**Flow:**
1. Session alkaa → lataa relevantit long-term memories
2. Session aikana → käytä short-term + query long-term
3. Session päättyy → arkistoi short-term → long-term

---

### Multi-Agent Memory Sharing

**Löydös:** Useat agentit voivat jakaa saman muistin

```
┌─────────────────────────────────────────────────────┐
│              SHARED MEMORY PATTERN                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Agent A (Planner)                                 │
│       │                                             │
│       ▼                                             │
│  ┌─────────────┐                                   │
│  │   SHARED    │                                   │
│  │   MEMORY    │ ◄─── Vector DB + Markdown files  │
│  │   STORE     │                                   │
│  └─────────────┘                                   │
│       ▲                                             │
│       │                                             │
│  Agent B (Coder)                                   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Soveltuvuus:** Meidän projektissa yksi agentti, mutta arkkitehtuuri tukee laajennusta.

---

## MemoryItem Primitiivi (Ehdotus)

### Data Model

```python
@dataclass
class MemoryItem:
    """Muistin perusyksikkö"""
    
    # Identity
    id: str                    # UUID
    
    # Content
    content: str               # Muistin sisältö (markdown)
    category: str              # decisions | summaries | context | facts
    
    # Metadata
    created_at: datetime
    updated_at: datetime
    session_id: Optional[str]  # Mistä sessiosta peräisin
    
    # Search
    embedding: Optional[List[float]]  # Vector embedding
    keywords: List[str]               # Manuaaliset tagit
    
    # Lifecycle
    salience: float            # 0.0-1.0, tärkeyspisteet
    access_count: int          # Kuinka usein haettu
    last_accessed: datetime    # Viimeisin haku

    # File reference (hybridi)
    file_path: Optional[str]   # Polku markdown-tiedostoon
```

### Lifecycle

```
CREATE                      ACCESS                    DECAY
───────                     ──────                    ─────
                            
User/AI creates fact        Query matches item        time_decay = 0.95^days
        │                           │                        │
        ▼                           ▼                        ▼
Calculate embedding         Return in results         salience *= time_decay
        │                           │                        │
        ▼                           ▼                        ▼
salience = 0.8 (default)    access_count += 1         if salience < 0.1:
        │                           │                   mark_for_cleanup()
        ▼                           ▼                        │
Save to storage             last_accessed = now       periodic_cleanup()
```

---

## Päivitetyt suositukset

### MVP Toteutus (Tarkennettu)

| Komponentti | Valinta | Perustelu |
|-------------|---------|-----------|
| **Tallennusmuoto** | Markdown + SQLite-vec | Hybridi: luettava + haettava |
| **Embedding** | Ollama nomic-embed-text | Ilmainen, 768 dim, hyvä laatu |
| **Chunking** | Fixed 512 tokens, 50 overlap | Yksinkertainen, toimiva default |
| **Hakustrategia** | Top-K (k=5) | Riittää MVP:lle |
| **Muistikategoriat** | 3: decisions, summaries, context | Vähemmän monimutkaisuutta |

### Tiedostorakenne (Tarkennettu)

```
/memory/
├── index.db                    # SQLite-vec (embeddings, metadata)
├── decisions/
│   ├── 2025-11-25_python_choice.md
│   └── 2025-11-25_api_design.md
├── summaries/
│   ├── session_001.md
│   └── session_002.md
└── context/
    └── current_session.md      # Aktiivinen sessio
```

---

## Seuraavat askeleet

1. ✅ ~~Perustutkimus tallennusvaihtoehdoista~~
2. ✅ ~~Chunking-strategiat~~
3. ✅ ~~Toteutusmallit~~
4. 🔲 MemoryItem API:n tarkempi määrittely
5. 🔲 SPEC_02 kirjoitus

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.0 | 2025-11-25 | Valmis versio - lisätty vahvistetut päätökset |
| 0.2 | 2025-11-25 | Lisätty chunking, toteutusmallit, MemoryItem |
| 0.1 | 2025-11-25 | Ensimmäinen välitallennus - perustiedot |

---

*Tutkimus valmis - käytetty SPEC_02 kirjoituksessa.*
