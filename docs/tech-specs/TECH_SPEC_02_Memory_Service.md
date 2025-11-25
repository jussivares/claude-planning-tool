# TECH_SPEC_02: Memory Service - Tekninen määrittely

> **Versio:** 1.0  
> **Päivitetty:** 2025-11-25  
> **Status:** ✅ Approved  
> **Perustuu:** SPEC_02 v1.1, TECH_RESEARCH_02 v1.0  
> **Laatija:** Gemini (Google AI) + Claude (Anthropic)  
> **Review:** Gemini ✅

---

## 1. Yleiskuvaus

Tämä dokumentti määrittelee MemoryServicen teknisen toteutuksen. Se on "copy-paste -valmis" ohje kehittäjälle (Claude Code) koodin kirjoittamiseen.

**Scope:**
- Python-luokkien rajapinnat ja toteutuslogiikka
- SQL-skeemat ja kyselyt
- Algoritmien pseudokoodi
- Konfiguraatiomalli
- Virheenkäsittely
- Testausstrategia

---

## 2. Teknologiavalinnat

### 2.1 Tech Stack

| Komponentti | Teknologia | Versio | Perustelu |
|-------------|------------|--------|-----------|
| Runtime | Python | 3.11+ | Async/await, type hints |
| Web Framework | FastAPI | ≥0.100.0 | Async endpoints, OpenAPI |
| Data Validation | Pydantic | ≥2.0.0 | Strict typing, serialisointi |
| Database | aiosqlite | ≥0.19.0 | Async SQLite |
| Vector Extension | sqlite-vec | ≥0.1.6 | Vektorihaku |
| File I/O | aiofiles | ≥23.0.0 | Non-blocking I/O |
| HTTP Client | httpx | ≥0.25.0 | Async, Ollama-kutsut |
| Embeddings | Ollama | - | nomic-embed-text (768 dim) |

### 2.2 Riippuvuudet (requirements.txt)

```
# Core
fastapi>=0.100.0
uvicorn[standard]>=0.23.0
pydantic>=2.0.0
pydantic-settings>=2.0.0

# Async I/O
aiosqlite>=0.19.0
aiofiles>=23.0.0

# Vector Search
sqlite-vec>=0.1.6

# HTTP Client
httpx>=0.25.0

# Utilities
python-dateutil>=2.8.0
```

---

## 3. Arkkitehtuuri

### 3.1 Komponenttikaavio

```
┌─────────────────────────────────────────────────────────────────┐
│                    MemoryService (Facade)                       │
│   save() | search() | get() | delete() | decay_salience()       │
│   get_context_for_prompt() | reindex_missing()                  │
└──────────────────────────────┬──────────────────────────────────┘
                               │
       ┌───────────────────────┼───────────────────────┐
       ▼                       ▼                       ▼
┌──────────────┐       ┌──────────────┐        ┌──────────────┐
│  FileStore   │       │ SQLiteRepo   │        │EmbeddingClient│
│  (aiofiles)  │       │ (aiosqlite)  │        │   (httpx)    │
└──────┬───────┘       └───────┬──────┘        └──────┬───────┘
       │                       │                      │
   [Markdown]            [index.db]              [Ollama API]
   /memory/*           (sqlite-vec)           localhost:11434
```

### 3.2 Tiedostorakenne

```
src/
├── memory/
│   ├── __init__.py
│   ├── service.py          # MemoryService (Facade)
│   ├── models.py           # Pydantic models
│   ├── repository.py       # SQLiteRepository
│   ├── file_store.py       # FileStore
│   ├── embedding_client.py # EmbeddingClient
│   ├── text_splitter.py    # TextSplitter
│   └── config.py           # Settings
│
data/
├── memory/
│   ├── index.db            # SQLite + sqlite-vec
│   ├── decisions/          # Markdown files
│   ├── summaries/
│   └── context/
```

---

## 4. Tietomalli

### 4.1 Pydantic Models (models.py)

```python
from enum import Enum
from typing import List, Optional
from datetime import datetime
from pydantic import BaseModel, Field, ConfigDict
import uuid


class MemoryCategory(str, Enum):
    """Muistikategoriat eri decay-rateilla."""
    DECISIONS = "decisions"   # decay: 1.0 (ei decay)
    SUMMARIES = "summaries"   # decay: 0.99
    CONTEXT = "context"       # decay: 0.90


class MemoryChunk(BaseModel):
    """Yksittäinen vektoroitu tekstinpätkä (internal)."""
    chunk_id: str = Field(default_factory=lambda: str(uuid.uuid4()))
    memory_id: str
    chunk_index: int
    content: str
    content_snippet: str = Field(max_length=200)
    embedding: List[float] = Field(default_factory=list)


class MemoryItem(BaseModel):
    """Täysi muisti-item (public API)."""
    id: str = Field(default_factory=lambda: str(uuid.uuid4()))
    title: str
    content: str  # Täysi markdown-sisältö
    category: MemoryCategory
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)
    last_accessed: Optional[datetime] = None
    access_count: int = 0
    salience: float = Field(default=1.0, ge=0.0, le=1.0)
    file_path: str
    
    model_config = ConfigDict(from_attributes=True)


class MemoryMetadata(BaseModel):
    """Kevyt versio ilman sisältöä (listauksiin)."""
    id: str
    title: str
    category: MemoryCategory
    created_at: datetime
    salience: float
    file_path: str


class SearchResult(BaseModel):
    """Hakutulos relevanssipisteillä."""
    memory: MemoryItem
    score: float = Field(ge=0.0, le=1.0)
    matched_snippet: str  # Mikä chunk matchasi


class ContextBlock(BaseModel):
    """Kontekstilohko promptiin liitettäväksi."""
    memory_id: str
    title: str
    category: MemoryCategory
    content: str
    relevance_score: float
```

### 4.2 Database Schema (DDL)

```sql
-- ============================================================
-- MEMORIES: Parent table (metadata)
-- ============================================================
CREATE TABLE IF NOT EXISTS memories (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    category TEXT NOT NULL CHECK(category IN ('decisions', 'summaries', 'context')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_accessed TIMESTAMP,
    access_count INTEGER DEFAULT 0,
    salience REAL DEFAULT 1.0 CHECK(salience >= 0.0 AND salience <= 1.0),
    file_path TEXT NOT NULL UNIQUE
);

-- ============================================================
-- MEMORY_CHUNKS: Child table (vectors)
-- sqlite-vec tallentaa vektorit BLOB-muodossa
-- ============================================================
CREATE TABLE IF NOT EXISTS memory_chunks (
    chunk_id TEXT PRIMARY KEY,
    memory_id TEXT NOT NULL,
    chunk_index INTEGER NOT NULL,
    content_snippet TEXT,
    embedding BLOB,  -- sqlite-vec: 768-dim float vector as BLOB
    FOREIGN KEY(memory_id) REFERENCES memories(id) ON DELETE CASCADE
);

-- ============================================================
-- INDEXES
-- ============================================================
CREATE INDEX IF NOT EXISTS idx_memories_category ON memories(category);
CREATE INDEX IF NOT EXISTS idx_memories_salience ON memories(salience DESC);
CREATE INDEX IF NOT EXISTS idx_memories_updated ON memories(updated_at DESC);
CREATE INDEX IF NOT EXISTS idx_chunks_memory_id ON memory_chunks(memory_id);
```

---

## 5. Komponentit

### 5.1 Settings (config.py)

```python
from pydantic_settings import BaseSettings
from pathlib import Path


class MemorySettings(BaseSettings):
    """MemoryServicen konfiguraatio (.env)."""
    
    # Paths
    MEMORY_STORAGE_PATH: Path = Path("./data/memory")
    DB_PATH: Path = Path("./data/memory/index.db")
    
    # Ollama
    OLLAMA_BASE_URL: str = "http://localhost:11434"
    OLLAMA_TIMEOUT: float = 30.0
    EMBEDDING_MODEL: str = "nomic-embed-text"
    EMBEDDING_DIMENSIONS: int = 768
    
    # Chunking
    CHUNK_SIZE: int = 512
    CHUNK_OVERLAP: int = 50
    
    # Search
    DEFAULT_SEARCH_LIMIT: int = 5
    SIMILARITY_THRESHOLD: float = 0.7
    
    # Salience
    DECAY_RATES: dict = {
        "decisions": 1.0,   # Ei decay
        "summaries": 0.99,
        "context": 0.90
    }
    
    class Config:
        env_file = ".env"
        env_prefix = "MEMORY_"


settings = MemorySettings()
```

### 5.2 TextSplitter (text_splitter.py)

```python
from typing import List


class TextSplitter:
    """
    Rekursiivinen tekstin pilkkominen embeddingejä varten.
    Pilkkoo: paragraphs → lines → sentences → words
    """
    
    SEPARATORS = ["\n\n", "\n", ". ", " "]
    
    def __init__(self, chunk_size: int = 512, overlap: int = 50):
        self.chunk_size = chunk_size
        self.overlap = overlap
    
    def split_text(self, text: str) -> List[str]:
        """Pilkkoo tekstin chunk_size-kokoisiin paloihin overlapilla."""
        if len(text) <= self.chunk_size:
            return [text] if text.strip() else []
        
        chunks = self._recursive_split(text, self.SEPARATORS)
        return self._merge_with_overlap(chunks)
    
    def _recursive_split(self, text: str, separators: List[str]) -> List[str]:
        """Rekursiivinen pilkkominen separaattoreilla."""
        if not separators:
            # Viimeinen keino: pilko merkki kerrallaan
            return [text[i:i+self.chunk_size] 
                    for i in range(0, len(text), self.chunk_size)]
        
        sep = separators[0]
        parts = text.split(sep)
        
        result = []
        for part in parts:
            if len(part) <= self.chunk_size:
                if part.strip():
                    result.append(part)
            else:
                # Liian iso, pilko pienemmällä separaattorilla
                result.extend(self._recursive_split(part, separators[1:]))
        
        return result
    
    def _merge_with_overlap(self, chunks: List[str]) -> List[str]:
        """Yhdistää pienet chunkit ja lisää overlapin."""
        if not chunks:
            return []
        
        merged = []
        current = chunks[0]
        
        for next_chunk in chunks[1:]:
            if len(current) + len(next_chunk) + 1 <= self.chunk_size:
                current = f"{current} {next_chunk}"
            else:
                merged.append(current)
                # Overlap: ota edellisen lopusta
                overlap_text = current[-self.overlap:] if len(current) > self.overlap else current
                current = f"{overlap_text} {next_chunk}".strip()
        
        if current:
            merged.append(current)
        
        return merged
    
    @staticmethod
    def create_snippet(text: str, max_length: int = 200) -> str:
        """Luo lyhyt esikatselu tekstistä."""
        if len(text) <= max_length:
            return text
        return text[:max_length-3] + "..."
```

### 5.3 EmbeddingClient (embedding_client.py)

```python
from typing import List, Optional
import httpx
import logging

from .config import settings

logger = logging.getLogger(__name__)


class EmbeddingError(Exception):
    """Embedding-operaatio epäonnistui."""
    pass


class EmbeddingClient:
    """
    Ollama API -client embeddingien generointiin.
    Käyttää nomic-embed-text -mallia (768 dimensiota).
    """
    
    def __init__(
        self,
        base_url: str = settings.OLLAMA_BASE_URL,
        model: str = settings.EMBEDDING_MODEL,
        timeout: float = settings.OLLAMA_TIMEOUT
    ):
        self.base_url = base_url.rstrip("/")
        self.model = model
        self.timeout = timeout
        self._client: Optional[httpx.AsyncClient] = None
    
    async def _get_client(self) -> httpx.AsyncClient:
        """Lazy initialization of HTTP client."""
        if self._client is None:
            self._client = httpx.AsyncClient(timeout=self.timeout)
        return self._client
    
    async def close(self):
        """Sulje HTTP client."""
        if self._client:
            await self._client.aclose()
            self._client = None
    
    async def embed(self, text: str) -> List[float]:
        """Generoi embedding yksittäiselle tekstille."""
        embeddings = await self.embed_batch([text])
        return embeddings[0] if embeddings else []
    
    async def embed_batch(self, texts: List[str]) -> List[List[float]]:
        """
        Generoi embeddingt usealle tekstille.
        
        Returns:
            Lista embedding-vektoreista. Tyhjä lista jos Ollama ei vastaa.
        """
        if not texts:
            return []
        
        client = await self._get_client()
        embeddings = []
        
        try:
            for text in texts:
                response = await client.post(
                    f"{self.base_url}/api/embeddings",
                    json={
                        "model": self.model,
                        "prompt": text
                    }
                )
                response.raise_for_status()
                data = response.json()
                embeddings.append(data.get("embedding", []))
            
            return embeddings
            
        except httpx.TimeoutException:
            logger.warning("Ollama timeout - returning empty embeddings")
            return []
        except httpx.HTTPError as e:
            logger.warning(f"Ollama HTTP error: {e} - returning empty embeddings")
            return []
        except Exception as e:
            logger.error(f"Unexpected embedding error: {e}")
            return []
    
    async def is_available(self) -> bool:
        """Tarkista onko Ollama saavutettavissa."""
        try:
            client = await self._get_client()
            response = await client.get(f"{self.base_url}/api/tags")
            return response.status_code == 200
        except Exception:
            return False
```

### 5.4 FileStore (file_store.py)

```python
from typing import Optional
from pathlib import Path
import aiofiles
import aiofiles.os
import logging

from .models import MemoryCategory
from .config import settings

logger = logging.getLogger(__name__)


class FileStoreError(Exception):
    """Tiedosto-operaatio epäonnistui."""
    pass


class FileStore:
    """
    Markdown-tiedostojen hallinta asynkronisesti.
    Tiedostorakenne: {base_path}/{category}/{filename}.md
    """
    
    def __init__(self, base_path: Path = settings.MEMORY_STORAGE_PATH):
        self.base_path = Path(base_path)
    
    async def init(self):
        """Luo hakemistorakenne."""
        for category in MemoryCategory:
            category_path = self.base_path / category.value
            await aiofiles.os.makedirs(category_path, exist_ok=True)
    
    def _get_file_path(self, category: MemoryCategory, filename: str) -> Path:
        """Muodosta tiedostopolku."""
        if not filename.endswith(".md"):
            filename = f"{filename}.md"
        return self.base_path / category.value / filename
    
    async def write(
        self,
        category: MemoryCategory,
        filename: str,
        content: str
    ) -> Path:
        """
        Kirjoita markdown-tiedosto.
        
        Returns:
            Tiedoston polku
        """
        file_path = self._get_file_path(category, filename)
        
        try:
            async with aiofiles.open(file_path, "w", encoding="utf-8") as f:
                await f.write(content)
            return file_path
        except Exception as e:
            logger.error(f"Failed to write {file_path}: {e}")
            raise FileStoreError(f"Write failed: {file_path}") from e
    
    async def read(self, file_path: str | Path) -> str:
        """Lue markdown-tiedoston sisältö."""
        try:
            async with aiofiles.open(file_path, "r", encoding="utf-8") as f:
                return await f.read()
        except FileNotFoundError:
            raise FileStoreError(f"File not found: {file_path}")
        except Exception as e:
            logger.error(f"Failed to read {file_path}: {e}")
            raise FileStoreError(f"Read failed: {file_path}") from e
    
    async def delete(self, file_path: str | Path) -> bool:
        """Poista tiedosto."""
        try:
            await aiofiles.os.remove(file_path)
            return True
        except FileNotFoundError:
            return False
        except Exception as e:
            logger.error(f"Failed to delete {file_path}: {e}")
            return False
    
    async def exists(self, file_path: str | Path) -> bool:
        """Tarkista onko tiedosto olemassa."""
        return await aiofiles.os.path.exists(file_path)
```

### 5.5 SQLiteRepository (repository.py)

```python
from typing import List, Optional, Tuple
from datetime import datetime
import sqlite_vec
import aiosqlite
import struct
import logging

from .models import MemoryItem, MemoryMetadata, MemoryChunk, MemoryCategory
from .config import settings

logger = logging.getLogger(__name__)


def serialize_vector(vec: List[float]) -> bytes:
    """Muunna float-lista BLOB:iksi sqlite-vec:lle."""
    return struct.pack(f"{len(vec)}f", *vec)


def deserialize_vector(blob: bytes) -> List[float]:
    """Muunna BLOB float-listaksi."""
    count = len(blob) // 4  # 4 bytes per float
    return list(struct.unpack(f"{count}f", blob))


class SQLiteRepository:
    """
    Tietokantaoperaatiot Raw SQL:llä.
    Tukee sqlite-vec vektorihakua.
    """
    
    DDL = """
    CREATE TABLE IF NOT EXISTS memories (
        id TEXT PRIMARY KEY,
        title TEXT NOT NULL,
        category TEXT NOT NULL CHECK(category IN ('decisions', 'summaries', 'context')),
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        last_accessed TIMESTAMP,
        access_count INTEGER DEFAULT 0,
        salience REAL DEFAULT 1.0,
        file_path TEXT NOT NULL UNIQUE
    );
    
    CREATE TABLE IF NOT EXISTS memory_chunks (
        chunk_id TEXT PRIMARY KEY,
        memory_id TEXT NOT NULL,
        chunk_index INTEGER NOT NULL,
        content_snippet TEXT,
        embedding BLOB,
        FOREIGN KEY(memory_id) REFERENCES memories(id) ON DELETE CASCADE
    );
    
    CREATE INDEX IF NOT EXISTS idx_memories_category ON memories(category);
    CREATE INDEX IF NOT EXISTS idx_memories_salience ON memories(salience DESC);
    CREATE INDEX IF NOT EXISTS idx_chunks_memory_id ON memory_chunks(memory_id);
    """
    
    def __init__(self, db_path: str = str(settings.DB_PATH)):
        self.db_path = db_path
        self._db: Optional[aiosqlite.Connection] = None
    
    async def init(self):
        """Alusta tietokanta ja lataa sqlite-vec."""
        self._db = await aiosqlite.connect(self.db_path)
        self._db.row_factory = aiosqlite.Row
        
        # Lataa sqlite-vec extension
        # IMPLEMENTATION NOTE (Gemini):
        # aiosqlite wrappaa sqlite3-yhteyden. Jos sqlite_vec.load(self._db)
        # ei toimi suoraan, käytä jompaakumpaa vaihtoehtoa:
        #
        # Vaihtoehto A: loadable_path()
        #   await self._db.enable_load_extension(True)
        #   await self._db.execute("SELECT load_extension(?)", [sqlite_vec.loadable_path()])
        #
        # Vaihtoehto B: Kaiva synkroninen yhteys
        #   await self._db.enable_load_extension(True)
        #   sqlite_vec.load(self._db._conn)  # Underlying sync connection
        #
        await self._db.enable_load_extension(True)
        sqlite_vec.load(self._db)
        await self._db.enable_load_extension(False)
        
        # Luo taulut
        await self._db.executescript(self.DDL)
        await self._db.commit()
        
        logger.info(f"Database initialized: {self.db_path}")
    
    async def close(self):
        """Sulje tietokantayhteys."""
        if self._db:
            await self._db.close()
            self._db = None
    
    # ─────────────────────────────────────────────────────────────
    # MEMORIES (Parent)
    # ─────────────────────────────────────────────────────────────
    
    async def save_memory(self, memory: MemoryItem) -> None:
        """Tallenna muistin metadata."""
        sql = """
            INSERT INTO memories (id, title, category, created_at, updated_at, salience, file_path)
            VALUES (?, ?, ?, ?, ?, ?, ?)
            ON CONFLICT(id) DO UPDATE SET
                title = excluded.title,
                updated_at = excluded.updated_at,
                salience = excluded.salience
        """
        await self._db.execute(sql, (
            memory.id,
            memory.title,
            memory.category.value,
            memory.created_at.isoformat(),
            memory.updated_at.isoformat(),
            memory.salience,
            str(memory.file_path)
        ))
        await self._db.commit()
    
    async def get_memory_metadata(self, memory_id: str) -> Optional[MemoryMetadata]:
        """Hae muistin metadata ID:llä."""
        sql = "SELECT id, title, category, created_at, salience, file_path FROM memories WHERE id = ?"
        async with self._db.execute(sql, (memory_id,)) as cursor:
            row = await cursor.fetchone()
            if row:
                return MemoryMetadata(
                    id=row["id"],
                    title=row["title"],
                    category=MemoryCategory(row["category"]),
                    created_at=datetime.fromisoformat(row["created_at"]),
                    salience=row["salience"],
                    file_path=row["file_path"]
                )
        return None
    
    async def delete_memory(self, memory_id: str) -> bool:
        """Poista muisti (CASCADE poistaa chunkit)."""
        sql = "DELETE FROM memories WHERE id = ?"
        cursor = await self._db.execute(sql, (memory_id,))
        await self._db.commit()
        return cursor.rowcount > 0
    
    async def update_access(self, memory_id: str) -> None:
        """Päivitä access-tiedot."""
        sql = """
            UPDATE memories 
            SET last_accessed = ?, access_count = access_count + 1
            WHERE id = ?
        """
        await self._db.execute(sql, (datetime.utcnow().isoformat(), memory_id))
        await self._db.commit()
    
    async def list_memories(
        self,
        category: Optional[MemoryCategory] = None,
        min_salience: float = 0.0
    ) -> List[MemoryMetadata]:
        """Listaa muistit filttereiden mukaan."""
        if category:
            sql = """
                SELECT id, title, category, created_at, salience, file_path 
                FROM memories 
                WHERE category = ? AND salience >= ?
                ORDER BY salience DESC, updated_at DESC
            """
            params = (category.value, min_salience)
        else:
            sql = """
                SELECT id, title, category, created_at, salience, file_path 
                FROM memories 
                WHERE salience >= ?
                ORDER BY salience DESC, updated_at DESC
            """
            params = (min_salience,)
        
        async with self._db.execute(sql, params) as cursor:
            rows = await cursor.fetchall()
            return [
                MemoryMetadata(
                    id=row["id"],
                    title=row["title"],
                    category=MemoryCategory(row["category"]),
                    created_at=datetime.fromisoformat(row["created_at"]),
                    salience=row["salience"],
                    file_path=row["file_path"]
                )
                for row in rows
            ]
    
    # ─────────────────────────────────────────────────────────────
    # CHUNKS (Children)
    # ─────────────────────────────────────────────────────────────
    
    async def save_chunks(self, chunks: List[MemoryChunk]) -> None:
        """Tallenna vektoroidut chunkit."""
        sql = """
            INSERT INTO memory_chunks (chunk_id, memory_id, chunk_index, content_snippet, embedding)
            VALUES (?, ?, ?, ?, ?)
        """
        data = [
            (
                chunk.chunk_id,
                chunk.memory_id,
                chunk.chunk_index,
                chunk.content_snippet,
                serialize_vector(chunk.embedding) if chunk.embedding else None
            )
            for chunk in chunks
        ]
        await self._db.executemany(sql, data)
        await self._db.commit()
    
    async def delete_chunks(self, memory_id: str) -> None:
        """Poista muistin chunkit."""
        sql = "DELETE FROM memory_chunks WHERE memory_id = ?"
        await self._db.execute(sql, (memory_id,))
        await self._db.commit()
    
    async def get_memories_without_chunks(self) -> List[MemoryMetadata]:
        """Hae muistit joilla ei ole chunkeja (reindex)."""
        sql = """
            SELECT m.id, m.title, m.category, m.created_at, m.salience, m.file_path
            FROM memories m
            WHERE NOT EXISTS (
                SELECT 1 FROM memory_chunks mc WHERE mc.memory_id = m.id
            )
        """
        async with self._db.execute(sql) as cursor:
            rows = await cursor.fetchall()
            return [
                MemoryMetadata(
                    id=row["id"],
                    title=row["title"],
                    category=MemoryCategory(row["category"]),
                    created_at=datetime.fromisoformat(row["created_at"]),
                    salience=row["salience"],
                    file_path=row["file_path"]
                )
                for row in rows
            ]
    
    # ─────────────────────────────────────────────────────────────
    # VECTOR SEARCH
    # ─────────────────────────────────────────────────────────────
    
    async def vector_search(
        self,
        query_embedding: List[float],
        limit: int = 5,
        min_salience: float = 0.0
    ) -> List[Tuple[str, str, str, float]]:
        """
        Vektorihaku sqlite-vec:llä.
        
        Returns:
            Lista tupleista: (memory_id, title, content_snippet, distance)
        """
        query_blob = serialize_vector(query_embedding)
        
        # sqlite-vec käyttää vec_distance_cosine funktiota
        sql = """
            SELECT 
                m.id,
                m.title,
                mc.content_snippet,
                vec_distance_cosine(mc.embedding, ?) as distance
            FROM memory_chunks mc
            JOIN memories m ON mc.memory_id = m.id
            WHERE m.salience >= ?
              AND mc.embedding IS NOT NULL
            ORDER BY distance ASC
            LIMIT ?
        """
        
        async with self._db.execute(sql, (query_blob, min_salience, limit * 3)) as cursor:
            rows = await cursor.fetchall()
            
            # Deduplikointi: sama muisti voi matchata useaan chunkkiin
            seen = set()
            results = []
            for row in rows:
                if row["id"] not in seen and len(results) < limit:
                    seen.add(row["id"])
                    results.append((
                        row["id"],
                        row["title"],
                        row["content_snippet"],
                        row["distance"]
                    ))
            
            return results
    
    # ─────────────────────────────────────────────────────────────
    # SALIENCE DECAY
    # ─────────────────────────────────────────────────────────────
    
    async def apply_salience_decay(self, decay_rates: dict) -> int:
        """
        Sovella salience decay kategorioittain.
        
        Returns:
            Päivitettyjen rivien määrä
        """
        total = 0
        for category, rate in decay_rates.items():
            if rate < 1.0:  # Ei päivitetä jos rate == 1.0
                sql = "UPDATE memories SET salience = salience * ? WHERE category = ?"
                cursor = await self._db.execute(sql, (rate, category))
                total += cursor.rowcount
        
        await self._db.commit()
        return total
```

### 5.6 MemoryService (service.py) - Facade

```python
from typing import List, Optional
from datetime import datetime
import asyncio
import uuid
import logging

from .models import (
    MemoryItem, MemoryCategory, SearchResult, 
    ContextBlock, MemoryChunk, MemoryMetadata
)
from .repository import SQLiteRepository
from .file_store import FileStore
from .embedding_client import EmbeddingClient
from .text_splitter import TextSplitter
from .config import settings

logger = logging.getLogger(__name__)


class MemoryService:
    """
    Ulkoisen muistin hallinta (Facade).
    
    Yhdistää:
    - Tiedostotallennus (FileStore)
    - Vektori-indeksi (SQLiteRepository + sqlite-vec)
    - Embeddings (EmbeddingClient → Ollama)
    """
    
    def __init__(self):
        self.repo = SQLiteRepository()
        self.file_store = FileStore()
        self.embedding_client = EmbeddingClient()
        self.splitter = TextSplitter(
            chunk_size=settings.CHUNK_SIZE,
            overlap=settings.CHUNK_OVERLAP
        )
    
    async def init(self):
        """Alusta kaikki komponentit."""
        await self.repo.init()
        await self.file_store.init()
        logger.info("MemoryService initialized")
    
    async def close(self):
        """Sulje yhteydet."""
        await self.repo.close()
        await self.embedding_client.close()
    
    # ─────────────────────────────────────────────────────────────
    # PUBLIC API
    # ─────────────────────────────────────────────────────────────
    
    async def save(
        self,
        title: str,
        content: str,
        category: MemoryCategory
    ) -> MemoryItem:
        """
        Tallenna uusi muisti.
        
        1. Generoi ID ja tiedostopolku
        2. Rinnakkain: kirjoita tiedosto + generoi embeddingt
        3. Transaktiona: tallenna metadata + chunkit
        """
        memory_id = str(uuid.uuid4())
        filename = f"{memory_id}.md"
        
        # Rinnakkaiset operaatiot
        async with asyncio.TaskGroup() as tg:
            # Task A: Kirjoita tiedosto
            file_task = tg.create_task(
                self.file_store.write(category, filename, content)
            )
            
            # Task B: Generoi embeddingt
            chunks_text = self.splitter.split_text(content)
            embed_task = tg.create_task(
                self.embedding_client.embed_batch(chunks_text)
            )
        
        file_path = file_task.result()
        embeddings = embed_task.result()
        
        # Luo MemoryItem
        memory = MemoryItem(
            id=memory_id,
            title=title,
            content=content,
            category=category,
            file_path=str(file_path)
        )
        
        # Tallenna metadata
        await self.repo.save_memory(memory)
        
        # Tallenna chunkit (jos embeddingt onnistuivat)
        if embeddings and len(embeddings) == len(chunks_text):
            chunks = [
                MemoryChunk(
                    memory_id=memory_id,
                    chunk_index=i,
                    content=text,
                    content_snippet=self.splitter.create_snippet(text),
                    embedding=emb
                )
                for i, (text, emb) in enumerate(zip(chunks_text, embeddings))
            ]
            await self.repo.save_chunks(chunks)
            logger.info(f"Saved memory {memory_id} with {len(chunks)} chunks")
        else:
            logger.warning(f"Saved memory {memory_id} without embeddings (Ollama unavailable)")
        
        return memory
    
    async def search(
        self,
        query: str,
        limit: int = settings.DEFAULT_SEARCH_LIMIT,
        min_salience: float = 0.0
    ) -> List[SearchResult]:
        """
        Semanttinen haku muisteista.
        
        1. Embedding querylle
        2. Vektorihaku
        3. Hydratoi tulokset (lue tiedostot)
        """
        # Embedding querylle
        query_embedding = await self.embedding_client.embed(query)
        if not query_embedding:
            logger.warning("Could not embed query - returning empty results")
            return []
        
        # Vektorihaku
        raw_results = await self.repo.vector_search(
            query_embedding, limit, min_salience
        )
        
        # Hydratoi tulokset
        results = []
        for memory_id, title, snippet, distance in raw_results:
            metadata = await self.repo.get_memory_metadata(memory_id)
            if metadata:
                content = await self.file_store.read(metadata.file_path)
                
                # Päivitä access stats
                await self.repo.update_access(memory_id)
                
                # Muunna distance → score (1 - distance)
                score = max(0.0, 1.0 - distance)
                
                results.append(SearchResult(
                    memory=MemoryItem(
                        id=memory_id,
                        title=title,
                        content=content,
                        category=metadata.category,
                        created_at=metadata.created_at,
                        updated_at=metadata.created_at,
                        salience=metadata.salience,
                        file_path=metadata.file_path
                    ),
                    score=score,
                    matched_snippet=snippet or ""
                ))
        
        return results
    
    async def get(self, memory_id: str) -> Optional[MemoryItem]:
        """Hae yksittäinen muisti ID:llä."""
        metadata = await self.repo.get_memory_metadata(memory_id)
        if not metadata:
            return None
        
        content = await self.file_store.read(metadata.file_path)
        await self.repo.update_access(memory_id)
        
        return MemoryItem(
            id=metadata.id,
            title=metadata.title,
            content=content,
            category=metadata.category,
            created_at=metadata.created_at,
            updated_at=metadata.created_at,
            salience=metadata.salience,
            file_path=metadata.file_path
        )
    
    async def delete(self, memory_id: str) -> bool:
        """Poista muisti (tietokanta + tiedosto)."""
        metadata = await self.repo.get_memory_metadata(memory_id)
        if not metadata:
            return False
        
        # Poista tiedosto
        await self.file_store.delete(metadata.file_path)
        
        # Poista tietokannasta (CASCADE poistaa chunkit)
        return await self.repo.delete_memory(memory_id)
    
    async def list(
        self,
        category: Optional[MemoryCategory] = None,
        min_salience: float = 0.0
    ) -> List[MemoryMetadata]:
        """Listaa muistit (kevyt, ilman sisältöä)."""
        return await self.repo.list_memories(category, min_salience)
    
    # ─────────────────────────────────────────────────────────────
    # CONTEXT INTEGRATION (for ContextManager)
    # ─────────────────────────────────────────────────────────────
    
    async def get_context_for_prompt(
        self,
        query: str,
        max_tokens: int = 4000,
        min_salience: float = 0.3
    ) -> List[ContextBlock]:
        """
        Hae relevantti konteksti promptiin liitettäväksi.
        
        Käyttö: ContextManager kutsuu tätä rakentaessaan promptia.
        
        Args:
            query: Käyttäjän viesti/kysymys
            max_tokens: Maksimi token-budjetti muisteille
            min_salience: Minimikynnys muisteille
        
        Returns:
            Lista ContextBlock-olioita prioriteettijärjestyksessä
        """
        # Hae relevantit muistit
        results = await self.search(query, limit=10, min_salience=min_salience)
        
        blocks = []
        estimated_tokens = 0
        
        for result in results:
            # Karkea token-estimaatti (4 merkkiä ≈ 1 token)
            content_tokens = len(result.memory.content) // 4
            
            if estimated_tokens + content_tokens > max_tokens:
                break
            
            blocks.append(ContextBlock(
                memory_id=result.memory.id,
                title=result.memory.title,
                category=result.memory.category,
                content=result.memory.content,
                relevance_score=result.score
            ))
            estimated_tokens += content_tokens
        
        return blocks
    
    async def get_raw_session_content(self, session_id: str) -> Optional[str]:
        """
        Hae session raaka sisältö (ilman embedding-prosessointia).
        
        Käyttö: ContextManager tarvitsee tämän kontekstin rakentamiseen
        kun ladataan aiempaa sessiota.
        
        Args:
            session_id: Session ID (= memory ID jos tallennettu muistina)
        
        Returns:
            Session markdown-sisältö tai None
        """
        metadata = await self.repo.get_memory_metadata(session_id)
        if not metadata:
            return None
        
        return await self.file_store.read(metadata.file_path)
    
    # ─────────────────────────────────────────────────────────────
    # MAINTENANCE
    # ─────────────────────────────────────────────────────────────
    
    async def decay_salience(self) -> int:
        """
        Sovella salience decay.
        Kutsutaan scheduled taskina (esim. kerran päivässä).
        """
        return await self.repo.apply_salience_decay(settings.DECAY_RATES)
    
    async def reindex_missing(self) -> int:
        """
        Uudelleenindeksoi muistit joilta puuttuu chunkit.
        Käyttö: Recovery jos Ollama oli alhaalla tallennushetkellä.
        """
        missing = await self.repo.get_memories_without_chunks()
        count = 0
        
        for metadata in missing:
            try:
                content = await self.file_store.read(metadata.file_path)
                chunks_text = self.splitter.split_text(content)
                embeddings = await self.embedding_client.embed_batch(chunks_text)
                
                if embeddings:
                    chunks = [
                        MemoryChunk(
                            memory_id=metadata.id,
                            chunk_index=i,
                            content=text,
                            content_snippet=self.splitter.create_snippet(text),
                            embedding=emb
                        )
                        for i, (text, emb) in enumerate(zip(chunks_text, embeddings))
                    ]
                    await self.repo.save_chunks(chunks)
                    count += 1
                    logger.info(f"Reindexed memory {metadata.id}")
            except Exception as e:
                logger.error(f"Failed to reindex {metadata.id}: {e}")
        
        return count
```

---

## 6. Virheenkäsittely

### 6.1 Virheluokat

| Virhe | Syy | Käsittely |
|-------|-----|-----------|
| `EmbeddingError` | Ollama timeout/virhe | Tallenna ilman embeddingejä, merkitse reindex |
| `FileStoreError` | Tiedosto-operaatio epäonnistui | Raise, älä tallenna DB:hen |
| `DatabaseError` | SQLite virhe | Rollback, raise |

### 6.2 Graceful Degradation

```
Ollama ei vastaa:
├── save() → Tallentaa metadatan + tiedoston, EI chunkeja
├── search() → Palauttaa tyhjän listan
└── reindex_missing() → Korjaa kun Ollama palaa

Tiedosto puuttuu:
├── get() → Palauttaa None
├── search() → Ohittaa tuloksen
└── Admin: Siivoa orpo metadata
```

---

## 7. Testausstrategia

### 7.1 Unit Tests (pytest)

```python
# tests/memory/test_text_splitter.py
def test_split_short_text():
    splitter = TextSplitter(chunk_size=100)
    result = splitter.split_text("Short text")
    assert result == ["Short text"]

def test_split_with_paragraphs():
    splitter = TextSplitter(chunk_size=50)
    text = "First paragraph.\n\nSecond paragraph."
    result = splitter.split_text(text)
    assert len(result) == 2

def test_overlap():
    splitter = TextSplitter(chunk_size=20, overlap=5)
    # ... test overlap logic
```

### 7.2 Integration Tests

```python
# tests/memory/test_memory_service.py
import pytest

@pytest.fixture
async def memory_service():
    service = MemoryService()
    await service.init()
    yield service
    await service.close()

async def test_save_and_search(memory_service):
    # Save
    memory = await memory_service.save(
        title="Test Memory",
        content="This is a test about Python programming.",
        category=MemoryCategory.CONTEXT
    )
    assert memory.id is not None
    
    # Search
    results = await memory_service.search("Python")
    assert len(results) > 0
    assert results[0].memory.id == memory.id

async def test_save_without_ollama(memory_service, monkeypatch):
    # Mock Ollama failure
    monkeypatch.setattr(
        memory_service.embedding_client, 
        "embed_batch", 
        lambda x: []
    )
    
    # Should still save metadata and file
    memory = await memory_service.save(...)
    assert memory.id is not None
    
    # But search won't find it
    results = await memory_service.search("test")
    assert len(results) == 0
```

### 7.3 sqlite-vec Tests

```python
async def test_vector_search_integration(memory_service):
    # Requires Ollama running
    if not await memory_service.embedding_client.is_available():
        pytest.skip("Ollama not available")
    
    # Save multiple memories
    await memory_service.save("Python Guide", "Python is a programming language...", ...)
    await memory_service.save("Cooking Recipe", "How to make pasta...", ...)
    
    # Search should find Python, not pasta
    results = await memory_service.search("programming language")
    assert results[0].memory.title == "Python Guide"
```

---

## 8. Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.0 | 2025-11-25 | Ensimmäinen versio (Gemini draft + Claude review) |

---

*Tämä dokumentti on osa claude-planning-tool -projektin teknistä dokumentaatiota.*
