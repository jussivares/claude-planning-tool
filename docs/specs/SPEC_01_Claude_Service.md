# SPEC_01: ClaudeService

> **Versio:** 1.2  
> **Päivitetty:** 2025-11-25  
> **Status:** Final  
> **Projekti:** Claude API -suunnittelutyökalu  
> **Tyyppi:** Toiminnallinen määrittely (SPEC)  
> **Edellinen:** v1.1 (2025-11-25)  
> **Tutkimus:** RESEARCH_01_Claude_API_Patterns.md  
> **Review:** Gemini (2025-11-25) - Hyväksytty korjauksin

---

## Muutokset v1.1 → v1.2 (Gemini-review)

| Muutos | Kuvaus | Gemini-viite |
|--------|--------|--------------|
| ✅ ClaudeServiceConfig | Konfiguraation injektointi | Kriittinen puute #1 |
| ✅ MetricsTracker | Kustannusseurannan rajapinta | Kriittinen puute #2 |
| ✅ Tietoturvahuomio | Lokituksen sanitointi | Parannusehdotus |
| ✅ TODO-lista | Tulevat parannukset dokumentoitu | Huomiot |

---

## Yleiskatsaus

### Tarkoitus

ClaudeService on **platform layer wrapper** joka käärii kaikki Claude API -kutsut. Muu sovellus ei koskaan kutsu Claude API:a suoraan - kaikki kulkee ClaudeServicen kautta.

### Black Box -periaate

```
┌─────────────────────────────────────────────────────────────────┐
│                     SOVELLUS                                    │
│  (Chat UI, ContextManager, MemoryService)                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    ClaudeService                                │
│                    ┌───────────┐                                │
│   send_message() ──►│           │──► Claude API                 │
│   stream_message()─►│  WRAPPER  │◄── Response                   │
│   count_tokens() ──►│           │                               │
│                    └───────────┘                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Konfiguraatio (v1.2 - uusi)

### ClaudeServiceConfig

```python
@dataclass
class ClaudeServiceConfig:
    """
    Konfiguraatio ClaudeServicelle.
    
    API-avain on PAKOLLINEN. Muut asetukset ovat valinnaisia
    ja käyttävät järkeviä oletusarvoja.
    """
    
    # Pakollinen
    api_key: str
    
    # Mallin asetukset
    default_model: ModelName = ModelName.SONNET_4_5
    default_max_tokens: int = 8192
    default_temperature: float = 1.0
    
    # Verkkoasetukset
    timeout: float = 600.0  # 10 min (pitkät 1M kontekstit)
    max_retries: int = 5
    
    # Valinnainen: ulkoinen hinnoittelu (ohittaa PRICING-vakiot)
    pricing_config: dict[str, dict[str, float]] | None = None
    
    # Valinnainen: ulkoiset beta-headerit (ohittaa BETA_HEADERS)
    beta_headers: dict[str, str] | None = None
    
    # Circuit breaker
    circuit_failure_threshold: int = 5
    circuit_recovery_timeout: float = 60.0


def load_config_from_env() -> ClaudeServiceConfig:
    """
    Lataa konfiguraatio ympäristömuuttujista.
    
    Vaatii: ANTHROPIC_API_KEY
    Valinnaiset: CLAUDE_DEFAULT_MODEL, CLAUDE_TIMEOUT, ...
    
    Raises:
        ValueError: Jos ANTHROPIC_API_KEY puuttuu
    """
    api_key = os.environ.get("ANTHROPIC_API_KEY")
    if not api_key:
        raise ValueError("ANTHROPIC_API_KEY environment variable is required")
    
    return ClaudeServiceConfig(
        api_key=api_key,
        default_model=ModelName(os.environ.get("CLAUDE_DEFAULT_MODEL", "claude-sonnet-4-5-20250929")),
        timeout=float(os.environ.get("CLAUDE_TIMEOUT", "600.0")),
    )
```

### Käyttöesimerkit

```python
# Vaihtoehto 1: Suora konfiguraatio
config = ClaudeServiceConfig(
    api_key="sk-ant-...",
    default_model=ModelName.SONNET_4_5,
)
service = ClaudeService(config)

# Vaihtoehto 2: Ympäristömuuttujista
config = load_config_from_env()
service = ClaudeService(config)

# Vaihtoehto 3: Testeissä
test_config = ClaudeServiceConfig(api_key="test-key")
service = ClaudeService(test_config)
```

---

## MetricsTracker (v1.2 - uusi)

### Rajapinta

```python
@dataclass
class MetricsTracker:
    """
    Seuraa API-käyttöä ja kustannuksia.
    
    ClaudeService päivittää tätä automaattisesti jokaisen
    API-kutsun jälkeen.
    """
    
    # Tokenit
    total_input_tokens: int = 0
    total_output_tokens: int = 0
    
    # Kustannukset (USD)
    total_cost: float = 0.0
    
    # Pyynnöt
    request_count: int = 0
    error_count: int = 0
    
    # Mallikohtainen erittely
    usage_by_model: dict[ModelName, dict] = field(default_factory=dict)
    
    def record_request(
        self,
        usage: TokenUsage,
        model: ModelName,
        cached_input_tokens: int = 0,
    ) -> None:
        """Kirjaa yhden API-kutsun käyttötiedot."""
        self.total_input_tokens += usage.input_tokens
        self.total_output_tokens += usage.output_tokens
        self.request_count += 1
        
        cost = self._calculate_cost(usage, model, cached_input_tokens)
        self.total_cost += cost
    
    def record_error(self) -> None:
        """Kirjaa epäonnistunut pyyntö."""
        self.error_count += 1
    
    def get_summary(self) -> dict:
        """Palauttaa yhteenvedon käytöstä."""
        return {
            "total_tokens": self.total_input_tokens + self.total_output_tokens,
            "total_cost_usd": round(self.total_cost, 4),
            "requests": self.request_count,
            "errors": self.error_count,
            "error_rate": self.error_count / max(self.request_count, 1),
        }
```

### Käyttö ClaudeServicessä

```python
class ClaudeService:
    def __init__(self, config: ClaudeServiceConfig):
        self.config = config
        self.metrics = MetricsTracker()  # Automaattinen seuranta
        self._client = anthropic.Anthropic(api_key=config.api_key)
```

---

## Tietoturva (v1.2 - uusi)

### ⚠️ KRIITTINEN: Lokituksen sanitointi

```python
# ❌ VÄÄRIN - Älä koskaan tee näin!
logger.info(f"Request: {messages}")  # Vuotaa käyttäjän datan
logger.debug(f"API key: {config.api_key}")  # Vuotaa avaimen

# ✅ OIKEIN - Lokita vain metatietoja
logger.info(f"Request: model={model}, tokens={token_count}")
logger.debug(f"API key configured: {bool(config.api_key)}")
```

### Lokitussäännöt

| Sallittu lokitettavaksi | EI SAA lokittaa |
|-------------------------|-----------------|
| Mallin nimi | Prompt-sisältö |
| Token-määrät | Vastauksen sisältö |
| Kustannukset | API-avain |
| Virhetyypit | System prompt |
| Latenssi | Käyttäjän tunnistetiedot |

---

## Primitiivi: Message

```python
@dataclass
class Message:
    role: Literal["user", "assistant"]
    content: str | list[ContentBlock]
    timestamp: datetime
    token_count: int | None = None
    model: str | None = None
```

---

## Julkinen API

### Konstruktori

```python
class ClaudeService:
    def __init__(self, config: ClaudeServiceConfig):
        """
        Args:
            config: ClaudeServiceConfig-objekti (pakollinen)
        
        Attributes:
            config: Konfiguraatio (read-only)
            metrics: MetricsTracker käytön seurantaan
        """
```

### Ydinmetodit

```python
async def send_message(
    self,
    messages: list[Message],
    *,
    system_prompt: str | None = None,
    model: ModelName | None = None,
    max_tokens: int | None = None,
    temperature: float | None = None,
    cache_system_prompt: bool = False,
) -> Message:
    """Lähetä viesti ja odota vastaus."""

async def stream_message(
    self,
    messages: list[Message],
    **kwargs,
) -> AsyncGenerator[StreamEvent, None]:
    """
    Lähetä viesti ja vastaanota vastaus streamina.
    
    ⚠️ HUOM: Virhe voi tulla 200-vastauksen JÄLKEEN!
    Käsittele StreamEvent.type == "error" loopissa.
    """

def count_tokens(
    self,
    messages: list[Message],
    **kwargs,
) -> TokenCount:
    """
    Laske tokenit ENNEN lähetystä.
    
    ⚠️ ILMAINEN API-kutsu - ei laskutusta!
    """
```

### Proaktiivinen Token-seuranta

```python
class TokenCount:
    input_tokens: int
    warnings: list[TokenWarning]
    
    def exceeds_context(self, model: ModelName) -> bool: ...
    def in_premium_zone(self) -> bool: ...
    def usage_percentage(self, model: ModelName) -> float: ...

TOKEN_THRESHOLDS = {
    "premium_zone": 0.2,   # 200K/1M
    "warning": 0.8,
    "critical": 0.9,
    "danger": 0.95,
}
```

---

## Mallit ja hinnoittelu

```python
class ModelName(Enum):
    SONNET_4 = "claude-sonnet-4-20250514"
    SONNET_4_5 = "claude-sonnet-4-5-20250929"
    OPUS_4_5 = "claude-opus-4-5-20251101"
    HAIKU_4_5 = "claude-haiku-4-5-20251001"

PRICING = {
    ModelName.SONNET_4: {"input": 3.0, "output": 15.0},
    ModelName.SONNET_4_5: {"input": 3.0, "output": 15.0},
    ModelName.OPUS_4_5: {"input": 5.0, "output": 25.0},
    ModelName.HAIKU_4_5: {"input": 0.5, "output": 2.5},
}

PREMIUM_MULTIPLIER = {"input": 2.0, "output": 1.5}  # >200K tokenia
```

---

## Virheenkäsittely

```python
class ClaudeAPIError(Exception): ...
class RateLimitError(ClaudeAPIError): retry_after: float
class TokenLimitError(ClaudeAPIError): current_tokens: int; max_tokens: int
class OverloadedError(ClaudeAPIError): ...
class CircuitOpenError(ClaudeAPIError): retry_after: float
```

### Circuit Breaker

```
CLOSED ──► OPEN ──► HALF_OPEN ──► CLOSED
```

---

## Rajaukset (MVP)

### Sisältyy MVP:hen ✅

- send_message, stream_message, count_tokens
- 4 mallia, 1M konteksti (Sonnet)
- Retry + Circuit Breaker
- Prompt Caching
- Token-seuranta
- **ClaudeServiceConfig (v1.2)**
- **MetricsTracker (v1.2)**
- **Tietoturvalokitus (v1.2)**

### EI sisälly MVP:hen ❌

- Tool use, Multimodal, Batch API, Extended thinking

---

## TODO: Tulevat parannukset (MVP:n jälkeen)

| # | Parannus | Prioriteetti |
|---|----------|--------------|
| 1 | Ulkoinen hinnoittelu (JSON/YAML) | P2 |
| 2 | Beta-headerien automaattinen hallinta | P2 |
| 3 | Prompt caching -arvio count_tokens:iin | P3 |

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.2 | 2025-11-25 | Gemini-review: Config, MetricsTracker, tietoturva |
| 1.1 | 2025-11-25 | Circuit breaker, streaming-virheet, prompt caching |
| 1.0 | 2025-11-25 | Ensimmäinen versio |

---

*Status: FINAL - Valmis TECH_SPEC_01:een.*
