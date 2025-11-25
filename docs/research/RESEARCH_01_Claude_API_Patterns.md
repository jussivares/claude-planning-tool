# RESEARCH_01: Claude API Patterns

> **Päivitetty:** 2025-11-25  
> **Status:** ✅ VALMIS  
> **Liittyy:** SPEC_01_Claude_Service  
> **Tiedonhaun taso:** Keskitaso (~45 min)

---

## Tutkimuskysymykset

1. **Virhekäsittely:** Mitkä ovat best practices Claude API:n virhekäsittelyyn? Retry-strategiat?
2. **Streaming:** Miten streaming toteutetaan optimaalisesti? Mitkä ovat edge caset?
3. **Token-laskenta:** Miten token-määrä arvioidaan ennen API-kutsua? Tiktoken vs API?
4. **1M konteksti:** Mitä erityiskysymyksiä liittyy 1M kontekstin käyttöön?
5. **Kustannusoptimointi:** Miten minimoidaan kustannukset tehokkaasti?

---

## Löydökset

### Kysymys 1: Virhekäsittely ja Retry-strategiat

**Status:** ✅ Valmis

**Lähteet:**
- [Anthropic Errors Docs](https://docs.claude.com/en/api/errors) - Virallinen dokumentaatio
- [Stack Overflow keskustelut](https://stackoverflow.com) - Käytännön kokemuksia
- GitHub Issues - Tunnetut ongelmat

**Keskeiset löydökset:**

| HTTP-koodi | Tyyppi | Merkitys | Toimenpide |
|------------|--------|----------|------------|
| 429 | rate_limit_error | Liikaa pyyntöjä | Retry with backoff |
| 529 | overloaded_error | API ylikuormitettu | Retry after delay |
| 413 | request_too_large | Pyyntö liian iso | Pienennä kontekstia |
| 500 | api_error | Sisäinen virhe | Retry |

**Best Practices:**

1. **Exponential Backoff with Jitter** - Standardi retry-strategia:
```python
wait_time = (2 ** attempt) + random.uniform(0, 1)
```

2. **Circuit Breaker Pattern** - Suojaa kaskadivirheiltä:
   - CLOSED → OPEN (virheraja ylittyy)
   - OPEN → HALF_OPEN (timeout päättyy)
   - HALF_OPEN → CLOSED (pyyntö onnistuu)

3. **Retry-After Header** - Palvelin voi antaa suositellun odotusajan

4. **⚠️ KRIITTINEN: Streaming-virheet**
   - Virhe voi tulla **200-vastauksen jälkeen** SSE:ssä!
   - Käsittele `error` eventit streaming-loopissa

**Johtopäätös:** Toteuta 3-tasoinen virheenkäsittely: (1) välitön retry rate limiteille, (2) exponential backoff yleisille virheille, (3) circuit breaker järjestelmätason suojaksi.

---

### Kysymys 2: Streaming-toteutus

**Status:** ✅ Valmis

**Lähteet:**
- [Anthropic Streaming Docs](https://docs.anthropic.com/en/docs/build-with-claude/streaming)
- [anthropic-sdk-python GitHub](https://github.com/anthropics/anthropic-sdk-python)

**Keskeiset löydökset:**

**Python SDK tarjoaa kaksi tapaa:**

1. **Low-level** (`stream=True`):
```python
stream = client.messages.create(..., stream=True)
for event in stream:
    print(event.type)
```

2. **High-level** (`.stream()` context manager) - SUOSITELTU:
```python
with client.messages.stream(...) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
    
    # Lopussa:
    final = stream.get_final_message()
```

**SSE Event-tyypit:**

| Event | Sisältö |
|-------|---------|
| `message_start` | Message-objekti, tyhjä content |
| `content_block_start` | Uuden sisältöblokin alku |
| `content_block_delta` | Tekstipätkä (delta) |
| `content_block_stop` | Blokin loppu |
| `message_delta` | Stop reason, usage |
| `message_stop` | Viestin loppu |
| `error` | ⚠️ Virhe (voi tulla 200:n jälkeen!) |

**Async-tuki:**
```python
async with client.messages.stream(...) as stream:
    async for text in stream.text_stream:
        ...
```

**Johtopäätös:** Käytä `.stream()` context manageria - se hoitaa resurssien siivouksen automaattisesti ja tarjoaa helper-metodit kuten `text_stream` ja `get_final_message()`.

---

### Kysymys 3: Token-laskenta

**Status:** ✅ Valmis

**Lähteet:**
- [Anthropic Token Counting Docs](https://docs.claude.com/en/docs/build-with-claude/token-counting)
- [anthropic-sdk-python releases](https://github.com/anthropics/anthropic-sdk-python/releases)

**Keskeiset löydökset:**

**VIRALLINEN TAPA (ILMAINEN!):**
```python
response = client.messages.count_tokens(
    model="claude-sonnet-4-5",
    system="You are a scientist",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.input_tokens)  # Tarkka määrä
```

**⚠️ TÄRKEÄÄ:**
- Vanha `client.count_tokens()` metodi **POISTETTU** SDK v0.39.0:ssa (2024-12)!
- Token counting on **ilmaista** mutta rate-limitoitua
- Tukee: tekstiä, kuvia, PDF:iä, tool-määrittelyjä

**Vaihtoehtoinen arvio (offline):**
- Tiktoken `p50k_base` encoding antaa **karkean arvion**
- EI tarkka Claude 3+ malleille!
- Käytä vain kun API ei ole käytettävissä

**Response usage (tarkka):**
```python
response = client.messages.create(...)
print(response.usage.input_tokens)
print(response.usage.output_tokens)
```

**Johtopäätös:** Käytä AINA `count_tokens()` API:a ennen suuria pyyntöjä. Se on ilmainen ja tarkka. Response `usage` on auktoritatiivinen laskutusta varten.

---

### Kysymys 4: 1M kontekstin erityiskysymykset

**Status:** ✅ Valmis

**Lähteet:**
- [Anthropic 1M Context Announcement](https://www.anthropic.com/news/1m-context)
- [Context Windows Docs](https://docs.claude.com/en/docs/build-with-claude/context-windows)

**Keskeiset löydökset:**

**Aktivointi:**
- Header: `anthropic-beta: context-1m-2025-08-07`
- Vain Sonnet 4 ja Sonnet 4.5
- Vaatii Usage Tier 4

**Hinnoittelu (>200K tokenia):**
| Tyyppi | Normaali | Premium (>200K) |
|--------|----------|-----------------|
| Input | $3/M | $6/M (2x) |
| Output | $15/M | $22.50/M (1.5x) |

**⚠️ Käyttäytymismuutos (Claude 3.7+):**
- **Validation error** jos `prompt + output > context window`
- EI enää hiljaista katkaisua!
- Käytä `count_tokens()` ennen lähetystä

**Best Practices:**
1. **Vältä viimeistä 20%** kontekstista kriittisille tehtäville
2. **Token counting** AINA ennen suuria pyyntöjä
3. **Chunking** suurille dokumenteille
4. **RAG** vain relevantin sisällön lataamiseen
5. **Prompt caching** toistuville konteksteille

**Extended Thinking huomio:**
- Thinking-tokenit lasketaan konteksti-ikkunaan
- Mutta **aiemmat thinking-blokit strippaantuvat** automaattisesti seuraavissa vuoroissa

**Johtopäätös:** 1M konteksti on tehokas mutta vaatii huolellista hallintaa. Toteuta proaktiivinen token-seuranta ja varoitukset kynnysarvoilla (esim. 80%, 90%, 95%).

---

### Kysymys 5: Kustannusoptimointi

**Status:** ✅ Valmis

**Lähteet:**
- Anthropic pricing documentation
- Best practices -artikkelit

**Strategiat:**

| Strategia | Säästö | Milloin käyttää |
|-----------|--------|-----------------|
| **Prompt Caching** | ~90% input | Toistuva system prompt/konteksti |
| **Batch Processing** | 50% | Ei-kiireelliset tehtävät |
| **Haiku vs Sonnet** | ~85% | Yksinkertaiset tehtävät |
| **Token budjetointi** | Vaihtelee | Hallittu kontekstin käyttö |
| **RAG** | Vaihtelee | Suuret dokumenttikokoelmat |

**Prompt Caching -esimerkki:**
```python
# Cachetettu system prompt
response = client.messages.create(
    model="claude-sonnet-4-5",
    system=[{
        "type": "text",
        "text": "Long system prompt...",
        "cache_control": {"type": "ephemeral"}
    }],
    messages=[...]
)
```

**Johtopäätös:** Yhdistä useita strategioita: prompt caching toistuvalle kontekstille + batch processing ei-kiireellisille + mallin valinta tehtävän mukaan.

---

## Suositukset SPEC_01:een

| # | Päätös | Suositus | Perustelu |
|---|--------|----------|-----------|
| 1 | Retry-strategia | Exponential backoff + jitter + circuit breaker | Teollisuusstandardi, kattava suoja |
| 2 | Streaming-toteutus | `.stream()` context manager | Automaattinen resurssinhallinta, helppokäyttöinen |
| 3 | Token-laskenta | `count_tokens()` API | Ilmainen, tarkka, virallinen |
| 4 | Streaming-virheet | Käsittele error eventit loopissa | 200 ei takaa onnistumista! |
| 5 | 1M kontekstin hallinta | Proaktiivinen seuranta + kynnysvaroitukset | Välttää validation errorit |
| 6 | Kustannusoptimointi | Prompt caching + mallin valinta | Merkittävät säästöt |

---

## SPEC_01:n puutteet tunnistettu

Tutkimuksen perusteella SPEC_01 v1.0:sta **puuttuu tai on puutteellisesti**:

1. **Streaming-virheiden käsittely** - Ei mainita että virhe voi tulla 200:n jälkeen
2. **Circuit breaker** - Ei mainita, vain retry
3. **Token counting API** - Mainitaan count_tokens(), mutta ei selitetä että se on ILMAINEN API-kutsu
4. **Prompt caching** - Merkitty "ei MVP:ssä" mutta olisi helppo lisätä
5. **Kynnysvaroitukset** - Ei mainita proaktiivista token-seurantaa
6. **Extended thinking -yhteensopivuus** - Ei mainita

---

## Avoimet kysymykset (→ Käyttäjälle)

1. **Circuit breaker MVP:ssä?** 
   - Lisääkö monimutkaisuutta, mutta parantaa resilienssiä
   - Ehdotus: Yksinkertainen toteutus, konfiguroitavat kynnysarvot

2. **Prompt caching MVP:ssä?**
   - SPEC sanoo "ei MVP:ssä" mutta on helppo lisätä
   - Ehdotus: Lisää MVP:hen (merkittävä kustannussäästö)

---

*Tutkimus valmis: 2025-11-25*
