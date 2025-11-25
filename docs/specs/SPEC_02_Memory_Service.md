# SPEC_02: Memory Service

> **Versio:** 1.1  
> **Päivitetty:** 2025-11-25  
> **Tiedosto:** SPEC_02_Memory_Service.md  
> **Moduuli:** MemoryService  
> **Prioriteetti:** P1 (MVP)  
> **Status:** ✅ Review Complete  
> **Liittyy:** RESEARCH_02_Memory_Architecture.md  
> **Review:** Gemini 2025-11-25

---

## Tiivistelmä

MemoryService vastaa ulkoisen muistin hallinnasta Claude Planning Tool -sovelluksessa. Se tarjoaa **persistentin, haettavan muistin** joka säilyy sessioiden välillä ja mahdollistaa kontekstin rakentamisen pitkissä suunnittelusessioissa.

**Ydinperiaate:** Hybridi-arkkitehtuuri jossa:
- **Markdown-tiedostot** = ihmisen luettava, Git-yhteensopiva tallennus
- **SQLite-vec** = nopea semantic search embeddingien avulla
- **Parent-Child chunking** = pitkät dokumentit pilkotaan hakua varten, mutta säilytetään kokonaisina

---

## Primitiivi: MemoryItem

MemoryItem on muistimoduulin perusyksikkö. Kaikki muistitoiminnot operoivat MemoryItem-instansseilla.

**Huom:** Embeddings tallennetaan erilliseen `memory_chunks`-tauluun (Parent-Child pattern). Yksi MemoryItem voi vastata useaa vektoria pitkissä dokumenteissa.

```python
@dataclass
class MemoryItem:
    """Muistin perusyksikkö - yksi muistettava asia."""
    
    # === IDENTITY ===
    id: str                     # UUID, uniikki tunniste
    
    # === CONTENT ===
    content: str                # Muistin sisältö (markdown-muotoinen)
    title: str                  # Lyhyt otsikko (generoitu tai annettu)
    category: MemoryCategory    # decisions | summaries | context
    
    # === METADATA ===
    created_at: datetime        # Luontiaika
    updated_at: datetime        # Viimeisin päivitys
    session_id: Optional[str]   # Mistä sessiosta peräisin
    source: str                 # "user" | "ai" | "system"
    
    # === SEARCH ===
    chunk_count: int            # Montako vektorichunkia (0 = ei embeddingiä)
    keywords: List[str]         # Manuaaliset tagit hakua varten
    
    # === LIFECYCLE ===
    salience: float             # 0.0-1.0, tärkeyspisteet (default 0.8)
    access_count: int           # Kuinka monta kertaa haettu
    last_accessed: Optional[datetime]  # Viimeisin hakuaika
    
    # === FILE REFERENCE ===
    file_path: Optional[str]    # Polku markdown-tiedostoon (/memory/...)
```

### MemoryCategory

```python
class MemoryCategory(Enum):
    DECISIONS = "decisions"     # Arkkitehtuuripäätökset, teknologiavalinnat
    SUMMARIES = "summaries"     # Sessioyhteenvedot, tiivistelmät
    CONTEXT = "context"         # Aktiivisen session konteksti
```

### Esimerkkejä

**Decision-muisti:**
```markdown
# Python + FastAPI valinta

**Päätös:** Käytetään Pythonia ja FastAPI:a backendin toteutuksessa.

**Perustelu:**
- Anthropic SDK on Python-first
- FastAPI tukee async/await ja WebSocketteja
- OpenAPI-dokumentaatio automaattisesti

**Vaihtoehdot (hylätyt):**
- Node.js - ei yhtä hyvää Anthropic SDK tukea
- Go - tiimin Python-osaaminen vahvempi

**Päätetty:** 2025-11-25, Sessio #1
```

**Summary-muisti:**
```markdown
# Session #4 Yhteenveto

**Päivämäärä:** 2025-11-25
**Kesto:** ~2h
**Fokus:** SPEC-prosessin testaus

**Saavutukset:**
- RESEARCH_01 valmistui (Claude API patterns)
- SPEC_01 päivitetty v1.0 → v1.1
- Circuit breaker ja prompt caching lisätty

**Avoimet:**
- SPEC_02 aloittamatta
- TECH_SPEC dokumentit jonossa

**Seuraavaksi:** RESEARCH_02 + SPEC_02 (MemoryService)
```

---

## Public API

### MemoryService Interface

```python
class MemoryService:
    """
    Ulkoisen muistin hallinta.
    
    Vastaa muistien tallennuksesta, hausta ja elinkaaren hallinnasta.
    Käyttää hybridi-arkkitehtuuria: Markdown + SQLite-vec.
    """
    
    # === CRUD Operations ===
    
    async def save(
        self,
        content: str,
        category: MemoryCategory,
        *,
        title: Optional[str] = None,
        keywords: List[str] = None,
        session_id: Optional[str] = None,
        source: str = "user"
    ) -> MemoryItem:
        """
        Tallenna uusi muisti.
        
        Args:
            content: Muistin sisältö (markdown)
            category: Kategoria (decisions/summaries/context)
            title: Otsikko (generoidaan jos ei annettu)
            keywords: Hakusanat
            session_id: Session tunniste
            source: Lähde ("user", "ai", "system")
            
        Returns:
            Tallennettu MemoryItem
            
        Side effects:
            - Luo markdown-tiedoston /memory/{category}/
            - Pilkkoo sisällön chunkeihin jos > 512 tokenia
            - Tallentaa embeddingin/embeddings SQLite-vec kantaan
            
        Chunking (Parent-Child):
            1. Tallenna tiedosto kokonaisena (parent)
            2. Jos content > 512 tokenia: pilko chunkeihin (overlap 50)
            3. Generoi embedding jokaiselle chunkille
            4. Tallenna chunkit memory_chunks-tauluun (linkki parent id:hen)
        """
        ...
    
    async def get(self, memory_id: str) -> Optional[MemoryItem]:
        """Hae muisti ID:llä."""
        ...
    
    async def update(
        self,
        memory_id: str,
        *,
        content: Optional[str] = None,
        title: Optional[str] = None,
        keywords: Optional[List[str]] = None
    ) -> MemoryItem:
        """
        Päivitä olemassa oleva muisti.
        
        Huom: Päivitys laskee uuden embeddingin jos content muuttuu.
        """
        ...
    
    async def delete(self, memory_id: str) -> bool:
        """
        Poista muisti.
        
        Returns:
            True jos poisto onnistui, False jos muistia ei löytynyt.
            
        Side effects:
            - Poistaa markdown-tiedoston
            - Poistaa embeddingin SQLite-vec kannasta
        """
        ...
    
    # === Search Operations ===
    
    async def search(
        self,
        query: str,
        *,
        limit: int = 5,
        category: Optional[MemoryCategory] = None,
        min_salience: float = 0.0,
        session_id: Optional[str] = None
    ) -> List[SearchResult]:
        """
        Hae muisteja semantic searchilla.
        
        Args:
            query: Hakukysely (luonnollinen kieli)
            limit: Maksimimäärä tuloksia (default 5)
            category: Rajaa kategoriaan (None = kaikki)
            min_salience: Minimi-tärkeys (0.0-1.0)
            session_id: Rajaa sessioon
            
        Returns:
            Lista SearchResult-objekteja relevanssin mukaan järjestettynä.
            
        Algorithm:
            1. Generoi embedding querylle (Ollama)
            2. Hae top-K lähimmät vektorit (SQLite-vec)
            3. Suodata category/salience/session mukaan
            4. Päivitä access_count ja last_accessed
            5. Palauta tulokset
        """
        ...
    
    async def search_by_keywords(
        self,
        keywords: List[str],
        *,
        limit: int = 10,
        match_all: bool = False
    ) -> List[MemoryItem]:
        """
        Hae muisteja keyword-matchilla (fallback jos embedding ei toimi).
        """
        ...
    
    # === Session Operations ===
    
    async def get_session_context(
        self,
        session_id: str
    ) -> List[MemoryItem]:
        """Hae kaikki session muistit aikajärjestyksessä."""
        ...
    
    async def get_raw_session_content(
        self,
        session_id: str
    ) -> str:
        """
        Palauta session kaikkien muistien sisältö yhtenä merkkijonona.
        
        Käyttötarkoitus: SessionManager hakee tällä raakatekstin,
        joka syötetään ClaudeServicelle tiivistettäväksi.
        
        Returns:
            Muistien sisältö yhdistettynä (\\n\\n eroteltuna)
            
        Note:
            MemoryService EI kutsu ClaudeServiceä suoraan (no circular dependency).
            Tiivistyslogiikka kuuluu SessionManagerille.
        """
        ...
    
    async def get_session_token_count(
        self,
        session_id: str
    ) -> int:
        """
        Laske session muistien arvioitu token-määrä.
        
        Käyttötarkoitus: SessionManager voi tarkistaa pitäisikö tiivistää.
        
        Returns:
            Arvioitu token count (chars / 4)
        """
        ...
    
    # === Lifecycle Operations ===
    
    async def decay_salience(
        self,
        *,
        days_inactive: int = 7
    ) -> int:
        """
        Laske vanhojen muistien salienssia kategoriaperusteisesti.
        
        **KRIITTINEN:** Eri kategorioilla eri decay-nopeus:
        - DECISIONS: Ei haalistu (rate=1.0) - pysyvät päätökset
        - SUMMARIES: Haalistuu hitaasti (rate=0.99)
        - CONTEXT: Haalistuu nopeasti (rate=0.90) - working memory
        
        Args:
            days_inactive: Montako päivää ilman käyttöä ennen decayta
            
        Returns:
            Päivitettyjen muistien määrä
            
        Algorithm:
            for category, rate in CATEGORY_DECAY_RATES.items():
                if rate < 1.0:
                    UPDATE memories 
                    SET salience = salience * rate
                    WHERE category = :cat 
                    AND last_accessed < :cutoff
        """
        ...
    
    async def cleanup(
        self,
        *,
        min_salience: float = 0.1
    ) -> int:
        """
        Poista muistit joiden salience on alle kynnyksen.
        
        Returns:
            Poistettujen muistien määrä
        """
        ...
    
    # === Utility Operations ===
    
    async def get_stats(self) -> MemoryStats:
        """Palauta muistitilastot."""
        ...
    
    async def export_memories(
        self,
        *,
        category: Optional[MemoryCategory] = None,
        format: str = "markdown"  # "markdown" | "json"
    ) -> str:
        """Vie muistit tiedostoon."""
        ...
    
    async def import_memories(
        self,
        source_path: str,
        *,
        category: MemoryCategory
    ) -> int:
        """Tuo muistit tiedostosta."""
        ...
```

### SearchResult

```python
@dataclass
class SearchResult:
    """Hakutulos relevanssipisteillä."""
    
    memory: MemoryItem          # Löydetty muisti (parent)
    score: float                # Relevanssi 0.0-1.0 (cosine similarity)
    distance: float             # Vektorietäisyys (pienempi = parempi)
    matched_snippet: str        # Osuma-chunk (512 tokens) - token budget optimointi
    matched_keywords: List[str] # Osuneet keywordit (jos keyword-haku)
```

**Huom (v1.1):** `matched_snippet` palauttaa vain osunut chunk, ei koko dokumenttia. Tämä säästää token-budjettia kun haetaan useita tuloksia.

### MemoryStats

```python
@dataclass
class MemoryStats:
    """Muistitilastot."""
    
    total_count: int                    # Muistien kokonaismäärä
    by_category: Dict[str, int]         # Määrä per kategoria
    total_tokens: int                   # Arvioitu token-määrä
    avg_salience: float                 # Keskimääräinen tärkeys
    oldest_memory: Optional[datetime]   # Vanhin muisti
    newest_memory: Optional[datetime]   # Uusin muisti
    index_size_bytes: int               # SQLite-vec indeksin koko
```

---

## Sisäinen arkkitehtuuri

### Komponenttien jako

```
┌─────────────────────────────────────────────────────────────────┐
│                      MEMORY SERVICE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    PUBLIC API                            │   │
│  │  MemoryService (facade)                                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                    │
│              ┌─────────────┼─────────────┐                     │
│              ▼             ▼             ▼                     │
│  ┌───────────────┐ ┌───────────────┐ ┌───────────────┐        │
│  │ FileStorage   │ │ VectorIndex   │ │ EmbeddingGen  │        │
│  │               │ │               │ │               │        │
│  │ - save_file() │ │ - add()       │ │ - embed()     │        │
│  │ - read_file() │ │ - search()    │ │ - embed_batch()│       │
│  │ - delete()    │ │ - delete()    │ │               │        │
│  │ - list()      │ │ - get_stats() │ │               │        │
│  │               │ │               │ │               │        │
│  │ Markdown      │ │ SQLite-vec    │ │ Ollama        │        │
│  │ /memory/*     │ │ index.db      │ │ nomic-embed   │        │
│  └───────────────┘ └───────────────┘ └───────────────┘        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### FileStorage (Internal)

```python
class FileStorage:
    """
    Markdown-tiedostojen hallinta.
    
    Vastaa muistien tallennuksesta ihmisen luettavaan muotoon.
    Git-yhteensopiva - versionhallinta toimii suoraan.
    """
    
    def __init__(self, base_path: str = "/memory"):
        self.base_path = Path(base_path)
    
    async def save_file(
        self,
        memory: MemoryItem
    ) -> str:
        """
        Tallenna muisti markdown-tiedostoksi.
        
        File naming: {date}_{slugified_title}.md
        Example: /memory/decisions/2025-11-25_python_fastapi.md
        
        Returns:
            Tiedostopolku
        """
        ...
    
    async def read_file(self, file_path: str) -> str:
        """Lue tiedoston sisältö."""
        ...
    
    async def delete_file(self, file_path: str) -> bool:
        """Poista tiedosto."""
        ...
    
    async def list_files(
        self,
        category: Optional[MemoryCategory] = None
    ) -> List[str]:
        """Listaa tiedostot."""
        ...
```

### VectorIndex (Internal)

```python
class VectorIndex:
    """
    SQLite-vec pohjainen vektori-indeksi.
    
    Käyttää sqlite-vec laajennusta semantic searchiin.
    Tukee Parent-Child chunkingia: yksi MemoryItem → useita vektoreita.
    """
    
    def __init__(self, db_path: str = "/memory/index.db"):
        self.db_path = db_path
        self._init_db()
    
    def _init_db(self):
        """
        Alusta tietokanta Parent-Child rakenteella.
        
        SQL Schema (v1.1):
        
        -- Parent table: Muistien metadata
        CREATE TABLE memories (
            id TEXT PRIMARY KEY,
            title TEXT,
            category TEXT,
            file_path TEXT,
            salience FLOAT,
            created_at TEXT,
            updated_at TEXT,
            session_id TEXT,
            source TEXT,
            access_count INTEGER DEFAULT 0,
            last_accessed TEXT
        );
        
        -- Child table: Vektorichunkit (One-to-Many)
        CREATE TABLE memory_chunks (
            chunk_id TEXT PRIMARY KEY,
            memory_id TEXT REFERENCES memories(id) ON DELETE CASCADE,
            chunk_index INTEGER,           -- Järjestysnumero dokumentissa
            content_snippet TEXT,          -- Chunkin teksti (debug/preview)
            embedding BLOB                 -- 768-dim vektori (sqlite-vec)
        );
        
        -- Index for fast parent lookup
        CREATE INDEX idx_chunks_memory_id ON memory_chunks(memory_id);
        """
        ...
    
    async def add_chunks(
        self,
        memory_id: str,
        chunks: List[str],
        embeddings: List[List[float]]
    ) -> int:
        """
        Lisää useita vektorichunkeja yhdelle muistille.
        
        Args:
            memory_id: Parent MemoryItem ID
            chunks: Tekstichunkit (snippetit)
            embeddings: Vastaavat embedding-vektorit
            
        Returns:
            Lisättyjen chunkien määrä
        """
        ...
    
    async def search(
        self,
        query_embedding: List[float],
        limit: int = 5,
        filters: Optional[Dict] = None
    ) -> List[Tuple[str, float, str]]:
        """
        Hae lähimmät vektorit (KNN).
        
        Returns:
            Lista (memory_id, distance, matched_snippet) tupleja
            
        Note:
            Palauttaa UNIIKIT memory_id:t - jos sama dokumentti
            osuu usealla chunkilla, palauttaa vain parhaan osuman.
        """
        ...
    
    async def delete(self, memory_id: str) -> bool:
        """Poista muistin kaikki chunkit (CASCADE)."""
        ...
    
    async def get_stats(self) -> Dict[str, Any]:
        """Palauta indeksin tilastot."""
        ...
```

### EmbeddingGenerator (Internal)

```python
class EmbeddingGenerator:
    """
    Embedding-vektorien generointi.
    
    MVP: Ollama + nomic-embed-text (768 dimensiota, ilmainen)
    Myöhemmin: Abstraktio eri providereille (OpenAI, Voyage, etc.)
    """
    
    def __init__(
        self,
        model: str = "nomic-embed-text",
        base_url: str = "http://localhost:11434"
    ):
        self.model = model
        self.base_url = base_url
    
    async def embed(self, text: str) -> List[float]:
        """
        Generoi embedding yksittäiselle tekstille.
        
        Returns:
            768-dimensioinen vektori
            
        Raises:
            EmbeddingError: Jos Ollama ei vastaa
        """
        ...
    
    async def embed_batch(
        self,
        texts: List[str],
        *,
        batch_size: int = 10
    ) -> List[List[float]]:
        """Generoi embeddings usealle tekstille."""
        ...
    
    @property
    def dimensions(self) -> int:
        """Embedding-vektorin dimensioiden määrä."""
        return 768  # nomic-embed-text
```

---

## Tiedostorakenne

```
/memory/
├── index.db                           # SQLite-vec tietokanta
├── decisions/
│   ├── 2025-11-25_python_fastapi.md
│   ├── 2025-11-25_markdown_storage.md
│   └── 2025-11-26_sqlite_vec.md
├── summaries/
│   ├── session_001_2025-11-25.md
│   └── session_002_2025-11-25.md
└── context/
    └── current_session.md             # Aktiivisen session muistit
```

### Markdown-tiedoston formaatti

```markdown
---
id: 550e8400-e29b-41d4-a716-446655440000
category: decisions
created_at: 2025-11-25T14:30:00Z
updated_at: 2025-11-25T14:30:00Z
session_id: session_004
source: user
keywords:
  - python
  - backend
  - framework
salience: 0.85
---

# Python + FastAPI valinta

**Päätös:** Käytetään Pythonia ja FastAPI:a backendin toteutuksessa.

**Perustelu:**
- Anthropic SDK on Python-first
- FastAPI tukee async/await ja WebSocketteja
- OpenAPI-dokumentaatio automaattisesti

**Vaihtoehdot (hylätyt):**
- Node.js - ei yhtä hyvää Anthropic SDK tukea
- Go - tiimin Python-osaaminen vahvempi

**Päätetty:** 2025-11-25, Sessio #1
```

---

## Virhetilanteet

### Virhetyypit

```python
class MemoryError(Exception):
    """Base exception for memory errors."""
    pass

class MemoryNotFoundError(MemoryError):
    """Muistia ei löydy annetulla ID:llä."""
    pass

class EmbeddingError(MemoryError):
    """Embedding-generointi epäonnistui (Ollama ei vastaa)."""
    pass

class StorageError(MemoryError):
    """Tiedoston luku/kirjoitus epäonnistui."""
    pass

class IndexError(MemoryError):
    """SQLite-vec operaatio epäonnistui."""
    pass
```

### Virheenkäsittely

| Virhe | Syy | Toimenpide |
|-------|-----|------------|
| `EmbeddingError` | Ollama ei käynnissä | Log warning, fallback keyword-hakuun |
| `StorageError` | Levytila loppu / oikeudet | Raise, näytä käyttäjälle |
| `IndexError` | Korruptoitunut indeksi | Rebuild index from markdown files |
| `MemoryNotFoundError` | ID ei löydy | Palauta None (get) tai False (delete) |

### Graceful Degradation

```
Jos Ollama ei ole käytettävissä:

1. save() → Tallenna markdown, skip embedding
2. search() → Fallback keyword-hakuun
3. Log warning: "Embeddings disabled, using keyword search"
4. Periodic retry: Yritä reconnetia 60s välein
```

---

## Konfiguraatio

```python
@dataclass
class MemoryConfig:
    """MemoryService konfiguraatio."""
    
    # Storage
    base_path: str = "/memory"
    db_name: str = "index.db"
    
    # Embedding
    embedding_model: str = "nomic-embed-text"
    embedding_url: str = "http://localhost:11434"
    embedding_dimensions: int = 768
    
    # Search
    default_search_limit: int = 5
    max_search_limit: int = 20
    
    # Lifecycle - Category-based decay rates (v1.1)
    category_decay_rates: Dict[MemoryCategory, float] = field(default_factory=lambda: {
        MemoryCategory.DECISIONS: 1.0,   # Ei haalistu koskaan
        MemoryCategory.SUMMARIES: 0.99,  # Haalistuu hyvin hitaasti
        MemoryCategory.CONTEXT: 0.90     # Haalistuu nopeasti
    })
    salience_decay_days: int = 7
    cleanup_threshold: float = 0.1
    
    # Session
    summarize_token_threshold: int = 50000
    
    # Chunking (for long content)
    chunk_size: int = 512       # tokens
    chunk_overlap: int = 50     # tokens
```

---

## Integraatio muihin moduuleihin

### ClaudeService → MemoryService

```python
# ContextManager hakee relevantin kontekstin
relevant_memories = await memory_service.search(
    query=user_message,
    limit=5,
    min_salience=0.3
)

# Rakenna konteksti
context = build_context(relevant_memories)

# Lähetä Claudelle
response = await claude_service.send_message(
    message=user_message,
    context=context
)
```

### Session lifecycle (SessionManager vastaa)

```python
# Session alku
session_id = generate_session_id()

# Session aikana - tallenna päätökset
await memory_service.save(
    content="Valittiin SQLite-vec...",
    category=MemoryCategory.DECISIONS,
    session_id=session_id
)

# Tarkista tarvitseeko tiivistää (SessionManager logiikka)
token_count = await memory_service.get_session_token_count(session_id)
if token_count > config.summarize_token_threshold:
    # 1. Hae raakasisältö
    raw_content = await memory_service.get_raw_session_content(session_id)
    
    # 2. Tiivistä ClaudeServicellä (EI MemoryServicessä!)
    summary = await claude_service.send_message(
        message="Tiivistä tämä sessio...",
        context=raw_content
    )
    
    # 3. Tallenna tiivistelmä
    await memory_service.save(
        content=summary,
        category=MemoryCategory.SUMMARIES,
        session_id=session_id,
        source="ai"
    )
```

**Huom:** MemoryService ei kutsu ClaudeServiceä suoraan → ei circular dependency.

---

## MVP Rajoitukset

Nämä ominaisuudet jätetään myöhempään versioon:

| Ominaisuus | Syy | Myöhemmin |
|------------|-----|-----------|
| Multi-user | Yksi käyttäjä MVP:ssä | v2.0 |
| Graph memory | mem0-tyylinen | v2.0 |
| Hybrid search | Keyword + semantic | v1.1 |
| OpenAI embeddings | Ollama riittää | v1.1 |
| Conflict detection | LLM-powered | v2.0 |
| Memory sharing | Agentit jakavat | v2.0 |
| **reindex_missing()** | Offline recovery (Gemini #4) | v1.2 |
| **save_batch()** | Bulk import (Gemini #5) | v1.2 |

### Kehityspolku v1.2 (Gemini review)

**reindex_missing_embeddings()** - Offline recovery:
```python
async def reindex_missing(self) -> int:
    """
    Etsii muistit joilta puuttuu embedding (Ollama oli alhaalla).
    Generoi embeddings uudelleen.
    
    Returns:
        Korjattujen muistien määrä
    """
    missing_ids = await self.vector_index.get_ids_without_vectors()
    # ... retry embedding generation
```

**save_batch()** - Bulk operations:
```python
async def save_batch(
    self, 
    items: List[MemoryItemRequest]
) -> List[MemoryItem]:
    """
    Optimoitu massatallennus.
    1. Kirjoita tiedostot rinnakkain (asyncio.gather)
    2. Generoi embeddingit batchina
    3. Yksi DB-transaktio
    """
```

---

## Testausstrategia

### Unit Tests

```python
# test_memory_service.py

async def test_save_and_get():
    """Tallenna ja hae muisti."""
    memory = await service.save(
        content="Test content",
        category=MemoryCategory.DECISIONS
    )
    retrieved = await service.get(memory.id)
    assert retrieved.content == "Test content"

async def test_search_returns_relevant():
    """Haku palauttaa relevantteja tuloksia."""
    await service.save(content="Python is great", ...)
    await service.save(content="JavaScript is okay", ...)
    
    results = await service.search("Python programming")
    assert results[0].memory.content == "Python is great"

async def test_delete_removes_file_and_index():
    """Poisto poistaa sekä tiedoston että indeksin."""
    memory = await service.save(...)
    await service.delete(memory.id)
    
    assert await service.get(memory.id) is None
    assert not Path(memory.file_path).exists()
```

### Integration Tests

```python
async def test_full_session_flow():
    """Testaa koko session elinkaari."""
    # 1. Luo sessio
    # 2. Tallenna useita muisteja
    # 3. Hae relevantteja
    # 4. Tiivistä sessio
    # 5. Varmista summary tallennettu
```

### Fallback Tests

```python
async def test_search_without_ollama():
    """Haku toimii vaikka Ollama ei ole käytettävissä."""
    # Mock Ollama unavailable
    results = await service.search("test query")
    # Should use keyword fallback
```

---

## Black Box Checklist

- [x] **Primitiivi tunnistettu:** MemoryItem
- [x] **Black box -rajat selkeät:** MemoryService ei paljasta SQLite/Ollama toteutusta
- [x] **API dokumentoitu:** Täysi interface yllä
- [x] **Ulkoiset riippuvuudet wrapattu:** 
  - Ollama → EmbeddingGenerator
  - SQLite-vec → VectorIndex
  - Filesystem → FileStorage
- [x] **Yksi omistaja:** Yksi kehittäjä voi ymmärtää koko moduulin
- [x] **Voidaan kirjoittaa uudelleen:** API:n perusteella mahdollista
- [x] **Toimii 10x vaatimuksilla:** 
  - 10x muisteja → SQLite skaalautuu
  - 10x hakuja → Indeksit toimivat

---

## Avoimet kysymykset (Ratkaistu)

| Kysymys | Ratkaisu | Perustelu |
|---------|----------|-----------|
| Embedding-malli? | Ollama nomic-embed-text | Ilmainen, riittävä laatu |
| Kategoriat? | 3 kpl (decisions, summaries, context) | MVP yksinkertaisuus |
| Summarize trigger? | Token-based (50K) + manual | Automaattinen + kontrolli |

---

## Viittaukset

- **RESEARCH_02_Memory_Architecture.md** - Tutkimustulokset
- **ARCHITECTURE_OVERVIEW.md** - Kokonaisarkkitehtuuri
- **SPEC_01_Claude_Service.md** - Integraatio Claudeen

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.1 | 2025-11-25 | **Gemini Review:** Parent-Child chunking, kategoriaperusteinen salience decay, circular dependency fix |
| 1.0 | 2025-11-25 | Ensimmäinen versio |

### v1.1 Korjaukset (Gemini Review)

| # | Havainto | Korjaus |
|---|----------|---------|
| #1 | Chunking + MemoryItem ristiriita | Parent-Child skeema: `memories` + `memory_chunks` taulut |
| #2 | Salience decay sokea kategorioille | `category_decay_rates`: DECISIONS=1.0, SUMMARIES=0.99, CONTEXT=0.90 |
| #3 | Circular dependency | Poistettu `summarize_session()`, lisätty `get_raw_session_content()` |

---

*Tämä dokumentti on toiminnallinen määrittely. Tekninen toteutus kuvataan TECH_SPEC_02:ssa.*
