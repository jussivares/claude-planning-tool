# API_REFERENCE

> **Versio:** 1.0  
> **PÃ¤ivitetty:** 2025-11-25  
> **Tiedosto:** v1.0_API_REFERENCE.md  
> **Projekti:** Claude API -suunnittelutyÃ¶kalu  
> **Tarkoitus:** Claude API:n tekninen referenssi

---

## Yleiskatsaus

TÃ¤mÃ¤ dokumentti sisÃ¤ltÃ¤Ã¤ Claude API:n tekniset tiedot, jotka on kerÃ¤tty virallisesta dokumentaatiosta 25.11.2025. Dokumentti toimii referenssinÃ¤ ClaudeService-wrapperin toteutukselle.

---

## Mallit ja hinnoittelu

### Mallivertailu (marraskuu 2025)

| Malli | API-tunnus | Input | Output | Konteksti | Max output |
|-------|------------|-------|--------|-----------|------------|
| **Opus 4.5** | `claude-opus-4-5-20251101` | $5/M | $25/M | 200K | 64K |
| **Sonnet 4.5** | `claude-sonnet-4-5-20250929` | $3/M | $15/M | 200K (1M beta) | 64K |
| **Haiku 4.5** | `claude-haiku-4-5-20251001` | $1/M | $5/M | 200K | 64K |
| **Sonnet 4** | `claude-sonnet-4-20250514` | $3/M | $15/M | 200K (1M beta) | 64K |

### 1M kontekstin premium-hinnoittelu

Kun kÃ¤ytetÃ¤Ã¤n 1M konteksti-ikkunaa (Sonnet 4 / 4.5):

| TokenimÃ¤Ã¤rÃ¤ | Input-hinta | Output-hinta | Kerroin |
|-------------|-------------|--------------|---------|
| â‰¤ 200K | $3/M | $15/M | 1x |
| > 200K | $6/M | $22.50/M | 2x / 1.5x |

### Kustannusoptimointi

| MenetelmÃ¤ | SÃ¤Ã¤stÃ¶ | Kuvaus |
|-----------|--------|--------|
| **Batch API** | 50% | Ei-kiireelliset pyynnÃ¶t, 24h kÃ¤sittelyaika |
| **Prompt caching** | jopa 90% | Toistuvat system promptit |
| Cache read | 0.1x | VÃ¤limuistista luetut tokenit |

---

## API-yhteys

### Endpoint

```
POST https://api.anthropic.com/v1/messages
```

### Pakolliset headerit

```http
x-api-key: YOUR_API_KEY
anthropic-version: 2023-06-01
content-type: application/json
```

### Beta-headerit

| Header | Arvo | Toiminto |
|--------|------|----------|
| 1M konteksti | `anthropic-beta: context-1m-2025-08-07` | Aktivoi 1M token ikkuna |
| Computer use | `anthropic-beta: computer-use-2024-10-22` | Computer use tools |

---

## Messages API

### Perus request-rakenne

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "max_tokens": 8192,
  "messages": [
    {
      "role": "user",
      "content": "Hello, Claude!"
    }
  ]
}
```

### TÃ¤ydellinen request-rakenne

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "max_tokens": 8192,
  "messages": [
    {
      "role": "user", 
      "content": "Analysoi tÃ¤mÃ¤ dokumentti."
    }
  ],
  "system": "Olet avulias assistentti.",
  "temperature": 0.7,
  "top_p": 0.9,
  "top_k": 40,
  "stop_sequences": ["STOP"],
  "stream": false,
  "metadata": {
    "user_id": "user-123"
  }
}
```

### Parametrit

| Parametri | Tyyppi | Pakollinen | Oletusarvo | Kuvaus |
|-----------|--------|------------|------------|--------|
| `model` | string | âœ… | - | Mallin tunnus |
| `max_tokens` | int | âœ… | - | Maksimi output-tokenit (1-64000) |
| `messages` | array | âœ… | - | Keskusteluhistoria |
| `system` | string | âŒ | - | System prompt |
| `temperature` | float | âŒ | 1.0 | Satunnaisuus (0.0-1.0) |
| `top_p` | float | âŒ | - | Nucleus sampling |
| `top_k` | int | âŒ | - | Top-K sampling |
| `stop_sequences` | array | âŒ | - | Lopetusmerkkijonot |
| `stream` | bool | âŒ | false | Streaming-vastaus |
| `metadata` | object | âŒ | - | Metadata (user_id) |

> **Huom:** Ã„lÃ¤ kÃ¤ytÃ¤ `temperature` ja `top_p` parametreja samanaikaisesti.

### Response-rakenne

```json
{
  "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
  "type": "message",
  "role": "assistant",
  "model": "claude-sonnet-4-5-20250929",
  "content": [
    {
      "type": "text",
      "text": "TÃ¤ssÃ¤ on vastaukseni..."
    }
  ],
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 25,
    "output_tokens": 150,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 0
  }
}
```

### Stop reasons

| Arvo | Kuvaus |
|------|--------|
| `end_turn` | Normaali lopetus |
| `max_tokens` | Max tokens saavutettu |
| `stop_sequence` | Stop sequence havaittu |
| `tool_use` | TyÃ¶kalu kutsuttu |
| `refusal` | TurvallisuuskieltÃ¤ytyminen |
| `model_context_window_exceeded` | Konteksti-ikkuna tÃ¤ynnÃ¤ |

---

## 1M konteksti-ikkuna

### Aktivointi

```python
from anthropic import Anthropic

client = Anthropic()

response = client.beta.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=8192,
    messages=[
        {"role": "user", "content": large_document}
    ],
    betas=["context-1m-2025-08-07"]
)
```

### Vaatimukset

| Vaatimus | Kuvaus |
|----------|--------|
| **Usage tier** | Tier 4 tai custom rate limits |
| **Mallit** | Vain Sonnet 4 ja Sonnet 4.5 |
| **Beta status** | Voi muuttua |

### Hinnoittelu

```
â‰¤ 200K tokenia:  $3/M input,  $15/M output
> 200K tokenia:  $6/M input,  $22.50/M output
```

### Rajoitukset

- Opus 4.5:lle **EI** saatavilla
- Erilliset rate limitit pitkille pyynnÃ¶ille
- Multimodal: kuvat/PDF:t kuluttavat tokeneita

---

## Effort-parametri (Opus 4.5)

### KÃ¤yttÃ¶

```python
response = client.messages.create(
    model="claude-opus-4-5-20251101",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000
    },
    messages=[...]
)
```

### Tasot

| Taso | Kuvaus | Token-kÃ¤yttÃ¶ |
|------|--------|--------------|
| `high` | Paras laatu, syvÃ¤ ajattelu | Korkea |
| `medium` | Tasapainoinen | 76% vÃ¤hemmÃ¤n kuin Sonnet 4.5 |
| `low` | Nopea, konservatiivinen | Matala |

### Huomioita

- Vain Opus 4.5 -mallille
- Thinking blockit **sÃ¤ilyvÃ¤t** kontekstissa (eri kuin Sonnet)
- Konteksti tÃ¤yttyy nopeammin kuin Sonnetilla

---

## Streaming

### Aktivointi

```python
with client.messages.stream(
    model="claude-sonnet-4-5-20250929",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### Event-tyypit (SSE)

| Event | Kuvaus |
|-------|--------|
| `message_start` | Viestin alku, sisÃ¤ltÃ¤Ã¤ metadatan |
| `content_block_start` | SisÃ¤ltÃ¶lohkon alku |
| `content_block_delta` | SisÃ¤ltÃ¶pÃ¤ivitys (teksti) |
| `content_block_stop` | SisÃ¤ltÃ¶lohkon loppu |
| `message_delta` | Viestin pÃ¤ivitys (stop_reason, usage) |
| `message_stop` | Viestin loppu |

---

## Tool Use (Function Calling)

### Tool-mÃ¤Ã¤rittely

```json
{
  "tools": [
    {
      "name": "get_weather",
      "description": "Hakee sÃ¤Ã¤tiedot annetulle kaupungille",
      "input_schema": {
        "type": "object",
        "properties": {
          "city": {
            "type": "string",
            "description": "Kaupungin nimi"
          }
        },
        "required": ["city"]
      }
    }
  ]
}
```

### Tool use -flow

```
1. User â†’ Claude (with tools defined)
2. Claude â†’ tool_use block (name, input)
3. User â†’ tool_result block (result)
4. Claude â†’ final response
```

### Tool choice

| Arvo | Kuvaus |
|------|--------|
| `auto` | Claude pÃ¤Ã¤ttÃ¤Ã¤ kÃ¤ytetÃ¤Ã¤nkÃ¶ tyÃ¶kalua |
| `any` | Claude TÃ„YTYY kÃ¤yttÃ¤Ã¤ jotain tyÃ¶kalua |
| `tool` | Claude TÃ„YTYY kÃ¤yttÃ¤Ã¤ tiettyÃ¤ tyÃ¶kalua |
| `none` | Claude EI saa kÃ¤yttÃ¤Ã¤ tyÃ¶kaluja |

### Tool use response

```json
{
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_01A09q90qw90lq917835lgs",
      "name": "get_weather",
      "input": {
        "city": "Helsinki"
      }
    }
  ],
  "stop_reason": "tool_use"
}
```

### Tool result

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01A09q90qw90lq917835lgs",
      "content": "Helsinki: 5Â°C, pilvistÃ¤"
    }
  ]
}
```

---

## Multimodal

### Kuvan lÃ¤hettÃ¤minen (base64)

```json
{
  "role": "user",
  "content": [
    {
      "type": "image",
      "source": {
        "type": "base64",
        "media_type": "image/jpeg",
        "data": "/9j/4AAQSkZJRg..."
      }
    },
    {
      "type": "text",
      "text": "MitÃ¤ kuvassa nÃ¤kyy?"
    }
  ]
}
```

### Kuvan lÃ¤hettÃ¤minen (URL)

```json
{
  "type": "image",
  "source": {
    "type": "url",
    "url": "https://example.com/image.jpg"
  }
}
```

### Rajoitukset

| Rajoitus | Arvo |
|----------|------|
| Max kuvia/pyyntÃ¶ | 20 |
| Max kuvakoko | 3.75 MB |
| Max resoluutio | 8000 x 8000 px |
| Max dokumentteja/pyyntÃ¶ | 5 |
| Max dokumenttikoko | 4.5 MB |

### Tuetut formaatit

- **Kuvat:** JPEG, PNG, GIF, WebP
- **Dokumentit:** PDF

---

## VirheenkÃ¤sittely

### HTTP-statuskoodit

| Koodi | Tyyppi | Kuvaus |
|-------|--------|--------|
| 200 | OK | Onnistunut pyyntÃ¶ |
| 400 | Bad Request | Virheellinen pyyntÃ¶ |
| 401 | Unauthorized | Virheellinen API-avain |
| 403 | Forbidden | Ei oikeuksia |
| 404 | Not Found | Resurssia ei lÃ¶ydy |
| 413 | Payload Too Large | PyyntÃ¶ liian suuri |
| 429 | Rate Limited | Liian monta pyyntÃ¶Ã¤ |
| 500 | Internal Error | Palvelinvirhe |
| 529 | Overloaded | API ylikuormitettu |

### Virhe-response

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "max_tokens must be greater than 0"
  }
}
```

### Virhetyypit

| Tyyppi | Kuvaus |
|--------|--------|
| `invalid_request_error` | Virheellinen pyyntÃ¶ |
| `authentication_error` | Autentikointivirhe |
| `permission_error` | Oikeusvirhe |
| `not_found_error` | Ei lÃ¶ydy |
| `rate_limit_error` | Rate limit ylitetty |
| `api_error` | API:n sisÃ¤inen virhe |
| `overloaded_error` | API ylikuormitettu |

### Retry-strategia

```python
import time
from anthropic import RateLimitError, APIError

def call_with_retry(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except RateLimitError:
            wait = 2 ** attempt  # Exponential backoff
            time.sleep(wait)
        except APIError as e:
            if e.status_code >= 500:
                time.sleep(1)
            else:
                raise
    raise Exception("Max retries exceeded")
```

---

## Rate Limits

### Tier-jÃ¤rjestelmÃ¤

| Tier | Vaatimus | Edut |
|------|----------|------|
| Tier 1 | Uusi kÃ¤yttÃ¤jÃ¤ | Perusrajat |
| Tier 2 | $50+ kÃ¤yttÃ¶Ã¤ | Korotetut rajat |
| Tier 3 | $200+ kÃ¤yttÃ¶Ã¤ | Korkeammat rajat |
| **Tier 4** | $400+ kÃ¤yttÃ¶Ã¤ | **1M konteksti saatavilla** |
| Custom | Yrityssopimus | RÃ¤Ã¤tÃ¤lÃ¶idyt rajat |

### Tyypilliset rajat

| Rajoitus | Kuvaus |
|----------|--------|
| RPM | Requests Per Minute |
| TPM | Tokens Per Minute |
| TPD | Tokens Per Day |

### Rate limit -headerit (response)

```http
x-ratelimit-limit-requests: 1000
x-ratelimit-limit-tokens: 100000
x-ratelimit-remaining-requests: 999
x-ratelimit-remaining-tokens: 99500
x-ratelimit-reset-requests: 2025-11-25T12:00:00Z
x-ratelimit-reset-tokens: 2025-11-25T12:00:00Z
```

---

## Python SDK

### Asennus

```bash
pip install anthropic
```

### Perusesimerkki

```python
from anthropic import Anthropic

client = Anthropic()  # Lukee ANTHROPIC_API_KEY ympÃ¤ristÃ¶muuttujasta

message = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Hello, Claude!"}
    ]
)

print(message.content[0].text)
```

### Async-kÃ¤yttÃ¶

```python
import asyncio
from anthropic import AsyncAnthropic

client = AsyncAnthropic()

async def main():
    message = await client.messages.create(
        model="claude-sonnet-4-5-20250929",
        max_tokens=1024,
        messages=[
            {"role": "user", "content": "Hello!"}
        ]
    )
    print(message.content[0].text)

asyncio.run(main())
```

### Streaming async

```python
async with client.messages.stream(
    model="claude-sonnet-4-5-20250929",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Tell me a story"}]
) as stream:
    async for text in stream.text_stream:
        print(text, end="", flush=True)
```

---

## KÃ¤yttÃ¶tapaukset tÃ¤lle projektille

### Suositeltu mallistrategia

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                    MALLISTRATEGIA                               â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                 â”‚
â”‚  PITKÃ„T SUUNNITTELUSESSIOT (ensisijainen kÃ¤yttÃ¶)               â”‚
â”‚  â””â”€ Malli: Sonnet 4.5 + 1M konteksti                           â”‚
â”‚  â””â”€ Hinta: $3-6/M input, $15-22.50/M output                    â”‚
â”‚  â””â”€ KÃ¤yttÃ¶: Projektidokumentit, koodikatselmukset              â”‚
â”‚                                                                 â”‚
â”‚  VAATIVAT TEHTÃ„VÃ„T (tarvittaessa)                              â”‚
â”‚  â””â”€ Malli: Opus 4.5 + effort=high                              â”‚
â”‚  â””â”€ Hinta: $5/M input, $25/M output                            â”‚
â”‚  â””â”€ KÃ¤yttÃ¶: ArkkitehtuuripÃ¤Ã¤tÃ¶kset, monimutkaiset ongelmat     â”‚
â”‚                                                                 â”‚
â”‚  NOPEAT TEHTÃ„VÃ„T (kustannustehokkuus)                          â”‚
â”‚  â””â”€ Malli: Haiku 4.5                                           â”‚
â”‚  â””â”€ Hinta: $1/M input, $5/M output                             â”‚
â”‚  â””â”€ KÃ¤yttÃ¶: Yksinkertaiset kyselyt, muotoilut                  â”‚
â”‚                                                                 â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Kustannusarvio (kuukausittainen)

| KÃ¤yttÃ¶tapa | Tokeneita/kk | Arvioitu kustannus |
|------------|--------------|---------------------|
| Kevyt (2M) | 2M | ~$10-20 |
| Normaali (10M) | 10M | ~$50-100 |
| Intensiivinen (50M) | 50M | ~$200-400 |

### MVP:n API-konfiguraatio

```python
# Oletuskonfiguraatio MVP:lle
DEFAULT_CONFIG = {
    "model": "claude-sonnet-4-5-20250929",
    "max_tokens": 8192,
    "betas": ["context-1m-2025-08-07"],  # 1M konteksti
    "stream": True,
}

# Vaihtoehtoinen Opus-konfiguraatio
OPUS_CONFIG = {
    "model": "claude-opus-4-5-20251101",
    "max_tokens": 16000,
    "thinking": {
        "type": "enabled",
        "budget_tokens": 10000
    },
    "stream": True,
}
```

---

## Muutoshistoria

| Versio | PÃ¤ivÃ¤mÃ¤Ã¤rÃ¤ | Muutokset |
|--------|------------|-----------|
| 1.0 | 2025-11-25 | EnsimmÃ¤inen versio - API-tiedot kerÃ¤tty |

---

## LÃ¤hteet

- [Claude API Documentation](https://docs.anthropic.com/)
- [Claude Models Overview](https://docs.anthropic.com/en/docs/about-claude/models)
- [Context Windows](https://docs.claude.com/en/docs/build-with-claude/context-windows)
- [Messages API Reference](https://docs.anthropic.com/en/api/messages)
- [Tool Use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
- [Anthropic Blog: Claude Opus 4.5](https://www.anthropic.com/news/claude-opus-4-5)
- [Anthropic Blog: 1M Context](https://www.claude.com/blog/1m-context)

---

*Dokumentti on osa Claude API -suunnittelutyÃ¶kalun dokumentaatiota.*
