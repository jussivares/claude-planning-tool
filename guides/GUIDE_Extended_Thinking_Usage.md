# Extended Thinking -käyttöopas

> **Versio:** 1.0  
> **Päivitetty:** 2025-11-25  
> **Tarkoitus:** Milloin käyttää Extended Thinking -tilaa ja milloin ei  
> **Opittu:** Kantapään kautta - timeout katkaisi pitkän tutkimussession

---

## Mitä Extended Thinking tekee?

Extended Thinking antaa Claudelle "ajatteluaikaa" ennen vastaamista. Claude käy läpi ongelman vaihe vaiheelta näkymättömässä thinking-lohkossa.

```
NORMAALI TILA:                    EXTENDED THINKING:
─────────────────                 ─────────────────────

Kysymys                           Kysymys
    │                                 │
    ▼                                 ▼
┌─────────┐                      ┌─────────────────┐
│ Vastaus │                      │ Thinking...     │
│ (heti)  │                      │ (näkymätön)     │
└─────────┘                      │ - Analysoi...   │
                                 │ - Vertailee...  │
                                 │ - Päättelee...  │
                                 └────────┬────────┘
                                          │
                                          ▼
                                    ┌─────────┐
                                    │ Vastaus │
                                    │ (syvempi)│
                                    └─────────┘
```

---

## ⚠️ Timeout-riski

Anthropicin dokumentaation mukaan:

> "Requests pushing the model to think above 32K tokens causes long running 
> requests that might run up against system timeouts and open connection limits."

**Riskitekijät:**
- Extended thinking ON
- Pitkä keskusteluhistoria (>30 viestiä)
- Monta web-hakua samassa pyynnössä
- Monimutkainen, pitkä vastaus

**Tulos:** Yhteys katkeaa, työ häviää.

---

## 🎯 Milloin käyttää Extended Thinking?

### ✅ KÄYTÄ kun:

| Tilanne | Esimerkki | Miksi auttaa |
|---------|-----------|--------------|
| **Monimutkainen päätös** | "Kumpi arkkitehtuuri on parempi: A vai B?" | Punnitsee systemaattisesti |
| **Matemaattinen ongelma** | "Laske optimaalinen token-budjetti" | Vaiheittainen laskenta |
| **Monivaiheinen logiikka** | "Jos X, niin Y, mutta jos Z..." | Seuraa ehtohaaroja |
| **Koodin debuggaus** | "Miksi tämä koodi ei toimi?" | Käy läpi rivi riviltä |
| **Strateginen analyysi** | "Mitä riskejä tässä lähestymistavassa on?" | Kattava arviointi |
| **Vertailu** | "Vertaa 5 eri kirjastoa näillä kriteereillä" | Järjestelmällinen taulukko |

**Tunnusmerkit:**
- Kysymys alkaa "miksi", "miten", "kumpi", "vertaa"
- Vastaus vaatii monta ajatteluvaihetta
- Oikea vastaus ei ole ilmeinen
- Virhe maksaa paljon (arkkitehtuuri, turvallisuus)

---

### ❌ ÄLÄ KÄYTÄ kun:

| Tilanne | Esimerkki | Miksi ei tarvita |
|---------|-----------|------------------|
| **Tiedonhaku** | "Etsi best practices API wrappereille" | Web search hoitaa |
| **Dokumentin kirjoitus** | "Kirjoita SPEC-dokumentti" | Luova työ, ei päättelyä |
| **Yksinkertainen kysymys** | "Mikä on Sonnet 4.5:n hinta?" | Fakta, ei analyysiä |
| **Pitkä tutkimussessio** | "Tutki ja dokumentoi löydökset" | Timeout-riski! |
| **Monta peräkkäistä hakua** | "Hae 5 eri lähteestä" | Kumuloituva riski |
| **Keskustelu on jo pitkä** | >30 viestiä historiassa | Konteksti + thinking = timeout |
| **Dokumentin lukeminen** | "Lue tämä SPEC ja tiivistä" | Ei tarvitse "ajatella" |

**Tunnusmerkit:**
- Tehtävä on pääosin hakemista tai kirjoittamista
- Vastaus on suoraviivainen
- Sessio on kestänyt jo pitkään
- Tehtävä vaatii monta työkalujen käyttöä

---

## 📋 Käytännön esimerkit projektiimme

### Esimerkki 1: RESEARCH-vaihe

```
TEHTÄVÄ: "Tutki Claude API:n retry-patternit ja rate limiting"

Extended Thinking: ❌ OFF

MIKSI?
├── Pääosin web-hakuja (5-10 hakua)
├── Dokumenttien lukemista
├── Tiedon kokoamista
├── Pitkäkestoinen sessio
└── Timeout-riski korkea

PAREMPI TAPA:
├── Tee haut ilman extended thinkingiä
├── Tallenna RESEARCH-dokumentti välillä
└── Laita thinking ON vasta analyysiä varten
```

### Esimerkki 2: Arkkitehtuuripäätös

```
TEHTÄVÄ: "Kumpi on parempi MemoryServicelle: 
          Markdown-tiedostot vai SQLite?"

Extended Thinking: ✅ ON

MIKSI?
├── Monimutkainen vertailu
├── Monta kriteeriä (nopeus, luettavuus, Git, haku...)
├── Pitkäaikaiset seuraukset
├── Ei web-hakuja (tieto on jo kontekstissa)
└── Yksittäinen, rajattu kysymys

TULOS:
├── Syvällisempi analyysi
├── Paremmin perusteltu suositus
└── Vähemmän "unohduksia"
```

### Esimerkki 3: SPEC-dokumentin kirjoitus

```
TEHTÄVÄ: "Kirjoita SPEC_02_Memory_Service.md"

Extended Thinking: ❌ OFF

MIKSI?
├── Pääosin kirjoitustyötä
├── Rakenne on jo määritelty (SYNTEESI)
├── Tutkimus on jo tehty (RESEARCH)
├── Pitkä dokumentti = pitkä prosessointiaika
└── Timeout-riski

PAREMPI TAPA:
├── Kirjoita ilman extended thinkingiä
├── Tallenna välillä
└── Käytä thinkingiä vain jos jumitat
```

### Esimerkki 4: Koodin debuggaus

```
TEHTÄVÄ: "Miksi tämä retry-logiikka ei toimi oikein?"

Extended Thinking: ✅ ON

MIKSI?
├── Vaatii koodin läpikäyntiä rivi riviltä
├── Logiikkavirheen etsintä
├── Rajattu, yksittäinen ongelma
├── Ei web-hakuja
└── Vastaus on lyhyt (korjausehdotus)
```

### Esimerkki 5: Pitkä keskustelu

```
TILANNE: Olemme keskustelleet 45 minuuttia,
         30+ viestiä historiassa

Extended Thinking: ❌ OFF (tai uusi ikkuna!)

MIKSI?
├── Konteksti on jo suuri
├── + Thinking tokenit = valtava pyyntö
├── Timeout-riski erittäin korkea
└── Aiempi työ voi hävitä

PAREMPI TAPA:
├── Avaa uusi keskusteluikkuna
├── Kirjoita "avaussanat" jatkoa varten
└── Aloita puhtaalta pöydältä
```

---

## 🔄 Dynaaminen käyttö session aikana

### Suositeltu malli:

```
SESSION ALKU
│
├── 1. Kontekstin lukeminen
│       Extended Thinking: OFF
│       (dokumenttien lukeminen)
│
├── 2. Tutkimus (RESEARCH)
│       Extended Thinking: OFF
│       (web-haut, tiedon keruu)
│       ⚠️ TALLENNA VÄLILLÄ!
│
├── 3. Analyysi ja päätökset
│       Extended Thinking: ON ← Laita päälle!
│       (vertailut, suositukset)
│
├── 4. Dokumentin kirjoitus
│       Extended Thinking: OFF
│       (SPEC, RESEARCH kirjoitus)
│       ⚠️ TALLENNA VÄLILLÄ!
│
├── 5. Review ja korjaukset
│       Extended Thinking: OFF tai ON
│       (riippuu monimutkaisuudesta)
│
└── SESSION LOPPU
        (tarkista tallennukset)
```

---

## 📊 Päätöspuu

```
                    ┌─────────────────────┐
                    │ Onko kysymys        │
                    │ monimutkainen       │
                    │ PÄÄTÖS/ANALYYSI?    │
                    └──────────┬──────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
             KYLLÄ                              EI
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐               ┌─────────────────┐
    │ Onko keskustelu │               │ Extended        │
    │ lyhyt (<20      │               │ Thinking: OFF   │
    │ viestiä)?       │               │                 │
    └────────┬────────┘               └─────────────────┘
             │
    ┌────────┴────────┐
    │                 │
   KYLLÄ              EI
    │                 │
    ▼                 ▼
┌─────────┐    ┌─────────────────┐
│ Extended│    │ Avaa UUSI       │
│ Thinking│    │ ikkuna, sitten  │
│ ON  ✅  │    │ thinking ON     │
└─────────┘    └─────────────────┘
```

---

## 🛡️ Turvaverkko: Välitallennus

**Riippumatta Extended Thinking -asetuksesta:**

> **"Tallenna välitulokset HETI outputs-kansioon."**
> Yhteys voi katketa milloin tahansa.

```
Tallennusväli:
├── RESEARCH: ~15 min välein
├── SPEC: jokaisen osion jälkeen
├── Pitkä analyysi: ennen ja jälkeen
└── Aina kun tuntuu "nyt on hyvä kohta"
```

---

## Yhteenveto

| Tilanne | Extended Thinking | Perustelu |
|---------|-------------------|-----------|
| Web-haku ja tutkimus | ❌ OFF | Timeout-riski |
| Dokumentin kirjoitus | ❌ OFF | Ei tarvita |
| Pitkä keskustelu (>30) | ❌ OFF + uusi ikkuna | Kumuloituva riski |
| Arkkitehtuuripäätös | ✅ ON | Hyötyy analyysistä |
| Vertailu (A vs B) | ✅ ON | Systemaattinen |
| Koodin debuggaus | ✅ ON | Vaiheittainen |
| Yksinkertainen kysymys | ❌ OFF | Ei hyötyä |

**Peukalosääntö:** 
> Jos tehtävä on "hae ja kirjoita" → OFF  
> Jos tehtävä on "analysoi ja päätä" → ON  
> Jos epävarma → OFF (turvallisempi)

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.0 | 2025-11-25 | Ensimmäinen versio (opittu timeout-ongelmasta) |

---

*Tämä opas syntyi käytännön kokemuksesta: pitkä tutkimussessio 
Extended Thinking päällä johti yhteyden katkeamiseen ja työn menetykseen.*
