# TECH_RESEARCH_02: Memory Service - Teknologiavalinnat

> **Versio:** 1.0  
> **Päivitetty:** 2025-11-25  
> **Perustuu:** SPEC_02 v1.1  
> **Tutkimuksen taso:** Keskitaso  
> **Tutkija:** Gemini (Google AI)  
> **Review:** Claude (Anthropic) - Pääarkkitehti

---

## 1. Tutkimuskysymykset

SPEC_02 määrittelee hybridimuistin (tiedostot + vektorikanta) ja asynkronisen API:n. Tämän toteuttamiseksi on ratkaistava seuraavat tekniset kysymykset:

| # | Kysymys | Prioriteetti | Vaikutus |
|---|---------|--------------|----------|
| 1 | Tietokanta-ajuri: Miten SQLiteä ajetaan asynkronisesti FastAPI:ssa? | Kriittinen | Suorituskyky, I/O-blokkaukset |
| 2 | Abstraktiotaso: ORM (SQLModel) vai Raw SQL? | Korkea | Yhteensopivuus sqlite-vec:n kanssa |
| 3 | Chunking: Oma toteutus vai valmis kirjasto (LangChain)? | Keskitaso | Riippuvuuksien määrä, ylläpito |
| 4 | Tiedosto-I/O: Miten markdown-tiedostot luetaan blokkaamatta? | Keskitaso | Latenssi kuormituksen alla |
| 5 | Datamallit: Pydantic vai dataclasses? | Keskitaso | Validointi, serialisointi |

---

## 2. Vaihtoehtojen analyysi

### 2.1 Kysymys 1: Tietokanta-ajuri (Async SQLite)

**Konteksti:** FastAPI on asynkroninen. Pythonin standardikirjaston `sqlite3` on synkroninen (blocking). Jos käytämme sitä suoraan, yksi tietokantahaku pysäyttää koko palvelimen muiden pyyntöjen käsittelyn.

**Vaihtoehdot:**

| Vaihtoehto | Kuvaus | Pros | Cons |
|------------|--------|------|------|
| A) **aiosqlite** | Wrapper, joka ajaa SQLiteä threadissa async/await -rajapinnalla | Native await syntax, de-facto standardi asynciin, helppo käyttää | Pieni overhead (merkityksetön tässä skaalassa) |
| B) sqlite3 + executor | Ajetaan standardikirjastoa `loop.run_in_executor`:lla | Ei ulkoisia riippuvuuksia | Koodi on verbosea ja virhealtista (thread safety) |

**Suositus:** ✅ **A) aiosqlite**

**Perustelu:** Tuottaa puhtainta koodia (`async with db.execute(...)`) ja integroituu saumattomasti FastAPI:n elinkaareen.

---

### 2.2 Kysymys 2: Abstraktiotaso (ORM vs Raw SQL)

**Konteksti:** Käytämme `sqlite-vec` -laajennosta vektorihakuihin. Se vaatii erityisiä SQL-funktioita (esim. `vec_distance_cosine`) ja mahdollisesti virtuaalitauluja.

**Vaihtoehdot:**

| Vaihtoehto | Kuvaus | Pros | Cons |
|------------|--------|------|------|
| A) SQLModel / SQLAlchemy | Täysi ORM (Object Relational Mapper) | Tyyppiturvallisuus, migraatiot, tuttu syntaksi | Huono tuki kustomoiduille C-laajennoksille. Vaatii purkkaa, jotta `vec_distance` toimii |
| B) **Raw SQL** | SQL-kyselyt kirjoitetaan merkkijonoina | Täysi kontrolli sqlite-vec:n ominaisuuksiin. Optimoitu suorituskyky. Yksinkertainen | Ei automaattista tyypitystä tuloksille (pitää mäpätä manuaalisesti) |

**Suositus:** ✅ **B) Raw SQL**

**Perustelu:** Koska tietokantaschema on yksinkertainen (2 taulua) ja logiikka nojaa vahvasti sqlite-vec:n erikoisominaisuuksiin, ORM toisi enemmän ongelmia kuin hyötyä. Pidämme repository-tason "lähellä rautaa".

**Arkkitehdin huomio:** Tämä on kriittinen oikea päätös. sqlite-vec:n funktiot kuten `vec_distance_cosine()` eivät toimi ORM:n kautta ilman merkittävää hackailua.

---

### 2.3 Kysymys 3: Chunking-strategia (Build vs Buy)

**Konteksti:** Pitkät tekstit pitää pilkkoa osiin (chunking) embeddingejä varten.

**Vaihtoehdot:**

| Vaihtoehto | Kuvaus | Pros | Cons |
|------------|--------|------|------|
| A) langchain-text-splitters | Valmis kirjasto (RecursiveCharacterTextSplitter) | Testattu, käsittelee kaikki edge caset, standardi | Tuo mukanaan valtavan määrän riippuvuuksia (bloat) |
| B) **Oma apufunktio** | Koodataan itse yksinkertainen splitteri (~50 riviä) | 0 riippuvuutta, kevyt, täysi kontrolli | Ylläpitovastuu, pitää testata huolella |

**Suositus:** ✅ **B) Oma apufunktio**

**Perustelu:** Tämä on ainoa NLP-toiminto jota tarvitsemme. Emme halua asentaa koko LangChain-ekosysteemiä vain yhden funktion takia. Toteutamme yksinkertaisen rekursiivisen splitterin.

**Arkkitehdin huomio:** LangChain tuo ~50+ transitiivista riippuvuutta. Rekursiivinen tekstin pilkkominen on yksinkertainen algoritmi joka pilkkoo ensin `\n\n` → `\n` → `. ` → ` ` -merkeillä kunnes chunk on tarpeeksi pieni.

---

### 2.4 Kysymys 4: Tiedosto-I/O

**Konteksti:** Markdown-tiedostojen luku levyltä on I/O-operaatio.

**Vaihtoehdot:**

| Vaihtoehto | Kuvaus | Pros | Cons |
|------------|--------|------|------|
| A) **aiofiles** | Asynkroninen tiedostokäsittely | Non-blocking, integroituu async-koodiin | Ulkoinen riippuvuus |
| B) Synkroninen open() | Standardikirjasto | Ei riippuvuuksia | Blokkaa event loopin |

**Suositus:** ✅ **A) aiofiles**

**Perustelu:** Standardikirjasto asynkroniseen tiedostokäsittelyyn. Estää event loopin blokkaantumisen kun luetaan isompia muistitiedostoja.

---

### 2.5 Kysymys 5: Datamallit (Pydantic vs dataclasses)

**Konteksti:** Tarvitsemme tyypitettyjä dataluokkia MemoryItem, MemoryChunk ym. mallintamiseen.

**Vaihtoehdot:**

| Vaihtoehto | Kuvaus | Pros | Cons |
|------------|--------|------|------|
| A) **Pydantic** | Data validation library | Automaattinen validointi, JSON serialisointi, FastAPI-integraatio | Hieman raskaampi |
| B) dataclasses | Standardikirjasto | Kevyt, ei riippuvuuksia | Ei automaattista validointia |

**Suositus:** ✅ **A) Pydantic**

**Perustelu (arkkitehti):** Pydantic on jo FastAPI:n riippuvuus, joten se ei lisää kuormaa. Saamme ilmaiseksi:
- Automaattisen input-validoinnin
- JSON serialisoinnin (`model_dump()`, `model_validate()`)
- OpenAPI-skeeman generoinnin
- Type coercion (esim. string → datetime)

---

## 3. Päätösyhteenveto

| # | Kysymys | Päätös | Perustelu |
|---|---------|--------|-----------|
| 1 | DB Driver | **aiosqlite** | Moderni async-tuki, helppo syntaksi |
| 2 | SQL Abstraktio | **Raw SQL** | Välttämätön sqlite-vec:n täyden hyödyntämisen kannalta |
| 3 | Chunking | **Oma toteutus** | Vältetään dependency bloat (LangChain) |
| 4 | File I/O | **aiofiles** | Non-blocking tiedostokäsittely |
| 5 | Datamallit | **Pydantic** | Validointi, FastAPI-integraatio (jo riippuvuus) |
| 6 | Vector Ext | **sqlite-vec** | Pip-asennettava versio (v0.1.6+) |
| 7 | HTTP Client | **httpx** | Async-yhteensopiva, moderni (Ollama-kutsuihin) |

---

## 4. Vaikutus TECH_SPEC:iin

Nämä päätökset muovaavat TECH_SPEC_02:n seuraavasti:

### Riippuvuudet (requirements.txt)

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

# HTTP Client (Ollama)
httpx>=0.25.0

# Utilities
python-dateutil>=2.8.0
```

### Arkkitehtuuri

```
MemoryService (Public API)
    │
    ├── SQLiteRepository (Raw SQL, aiosqlite)
    │       └── sqlite-vec extension
    │
    ├── FileStore (aiofiles)
    │       └── Markdown read/write
    │
    ├── EmbeddingClient (httpx → Ollama)
    │
    └── TextChunker (oma toteutus)
```

### sqlite-vec lataus

```python
import sqlite_vec
import aiosqlite

async def init_database(db_path: str) -> aiosqlite.Connection:
    db = await aiosqlite.connect(db_path)
    await db.enable_load_extension(True)
    sqlite_vec.load(db)
    await db.enable_load_extension(False)  # Turvallisuus
    return db
```

### Tietomalli

Tarvitaan DDL-scriptit (`CREATE TABLE IF NOT EXISTS...`) sovelluksen käynnistykseen, koska ei ole ORM-migraatioita.

---

## 5. Avoimet kysymykset

**Status:** ✅ Kaikki päätökset hyväksytty arkkitehdin toimesta.

| Kysymys | Vastaus |
|---------|---------|
| Raw SQL vs ORM? | ✅ Raw SQL hyväksytty |
| Oma chunker vs LangChain? | ✅ Oma toteutus hyväksytty |
| Pydantic datamallit? | ✅ Lisätty arkkitehdin toimesta |

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.0 | 2025-11-25 | Ensimmäinen versio (Gemini + Claude review) |

---

*Tämä dokumentti on osa claude-planning-tool -projektin teknistä dokumentaatiota.*
