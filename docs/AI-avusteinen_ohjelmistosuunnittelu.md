# AI-avusteinen ohjelmistosuunnittelu

> **Versio:** 2.1  
> **Päivitetty:** 2025-11-30  
> **Tiedosto:** v2_1_AI-avusteinen_ohjelmistosuunnittelu.md  
> **Tarkoitus:** Filosofia ja periaatteet AI-avusteiseen ohjelmistosuunnitteluun  
> **Edellinen:** v2.0 (2025-11-25)  
> **Liittyy:** PROCESS_SPEC_Writing.md (käytännön prosessi), META_Development_Model.md (geneerisyys)

---

## Dokumentin tarkoitus

Tämä dokumentti kuvaa **MIKSI** teemme asioita tietyllä tavalla.

Käytännön **MITEN** löytyy dokumentista PROCESS_SPEC_Writing.md.

```
┌─────────────────────────────────────────────────────────────────┐
│  AI-avusteinen_ohjelmistosuunnittelu    PROCESS_SPEC_Writing   │
│  ─────────────────────────────────────  ─────────────────────  │
│                                                                 │
│  MIKSI?                                 MITEN?                  │
│  ├── Filosofia                          ├── 11-vaiheinen prosessi│
│  ├── Periaatteet                        ├── Tarkistuslistat     │
│  ├── Ajattelutapa                       ├── Dokumenttipohjat    │
│  └── Arvot                              └── Käytännön ohjeet    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⭐ Mallin geneerisyys (Fundamentti)

**Tämä kehitysmalli on suunniteltu geneerisenä viitekehyksenä.**

```
┌─────────────────────────────────────────────────────────────────┐
│  MALLI EI OLE PROJEKTIKOHTAINEN - SE ON METODOLOGIA            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Nykyinen projekti:                                            │
│  └── Claude API -suunnittelutyökalu                            │
│                                                                 │
│  Tulevat projektit:                                            │
│  └── Enterprise-scale ERP-järjestelmä                          │
│  └── Muut ohjelmistoprojektit                                  │
│                                                                 │
│  Malli säilyy ja kehittyy:                                     │
│  └── Jokainen projekti parantaa mallia                         │
│  └── Opit siirtyvät seuraaviin projekteihin                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Täysi dokumentaatio:** Katso META_Development_Model.md

---

## ⭐ Kielikäytäntö (v2.1)

| Konteksti | Kieli | Perustelu |
|-----------|-------|-----------|
| **Keskustelu** | Suomi | Käyttäjän mukavuus |
| **Dokumentaatio** | Englanti | Geneerisyys, AI-yhteensopivuus |
| **Koodi** | Englanti | Toimialastandardi |
| **Kommentit** | Englanti | Yhtenäisyys koodin kanssa |

**Käytännössä:**
- Käyttäjä kysyy suomeksi: "Miten testaus pitäisi hoitaa?"
- Claude vastaa suomeksi keskustelussa
- Dokumentti kirjoitetaan englanniksi: `PROCESS_Testing.md`

---

## 1. Ydinfilosofia: Iteratiivinen spiraali

### Perinteinen vs. AI-avusteinen

```
PERINTEINEN (Vesiputous)              AI-AVUSTEINEN (Spiraali)
─────────────────────────             ─────────────────────────

Vaatimukset (kiinteät)                Visio + Tutkimus
      │                                       │
      ▼                                       ▼
Suunnittelu (erikseen)                ┌───────────────────┐
      │                               │                   │
      ▼                               │  SPEC ◄────────►  │
Toteutussuunnitelma                   │    │      │       │
      │                               │    ▼      ▼       │
      ▼                               │  DATABASE ◄──►    │
Koodaus                               │    │      │       │
                                      │    ▼      ▼       │
                                      │  ARKKITEHTUURI    │
                                      │                   │
                                      │  (kehittyvät      │
                                      │   yhdessä)        │
                                      └─────────┬─────────┘
                                                │
                                                ▼
                                          Toteutus
                                       (kun kypsä)
```

### "Neo-Waterfall" AI:n aikakaudella (v2.1)

Perinteinen vesiputous epäonnistui koska ihmiset:
- Unohtavat yksityiskohdat
- Väsyvät ja tekevät virheitä
- Vaihtuvat kesken projektin
- Eivät huomaa ristiriitoja

**AI muuttaa tämän:**

| Ongelma | Miten AI ratkaisee |
|---------|-------------------|
| Ihmiset unohtavat | Claude ylläpitää 1M token kontekstia |
| Ihmiset väsyvät | Claude on johdonmukainen |
| Ihmiset vaihtuvat | Dokumentaatio + AI = jatkuvuus |
| Spec vanhenee | Claude vertaa spec vs. koodi |

**Tulos:** Perusteellinen etukäteissuunnittelu on taas mahdollista kun AI avustaa.

### Spiraalimallin ydin

1. **Dokumentit kehittyvät samanaikaisesti** - SPEC, DATABASE ja ARKKITEHTUURI vaikuttavat toisiinsa
2. **Ei "kiveen hakattuja" vaatimuksia** - Ymmärrys syvenee matkan varrella
3. **Iteraatio on normaalia** - Paluu aiempiin dokumentteihin on osa prosessia
4. **Toteutus alkaa kun suunnittelu on kypsä** - Ei kiire koodaamaan

---

## 2. AI-kumppanuusmalli

### Roolijako

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI-KUMPPANUUSMALLI                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  IHMINEN (Projektipäällikkö)                                   │
│  ├── Visio ja suunta                                           │
│  ├── Liiketoimintaymmärrys                                     │
│  ├── Päätökset ja priorisointi                                 │
│  ├── Laadunvarmistus (review)                                  │
│  └── Tallentaa dokumentit                                      │
│                                                                 │
│  AI (Claude) - KAKSOISROOLI                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  ROOLI 1: SUUNNITTELIJA     ROOLI 2: TUTKIJA           │   │
│  │  ─────────────────────      ─────────────────           │   │
│  │  Kysyy oikeat kysymykset    Hakee vastaukset            │   │
│  │  Tunnistaa vaihtoehdot      Vertailee ratkaisuja        │   │
│  │  Tekee suositukset          Dokumentoi lähteet          │   │
│  │  Kirjoittaa SPECit          Kirjoittaa RESEARCHit       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  AI (Gemini) - PEER REVIEW                                     │
│  ├── Tekninen validointi                                       │
│  ├── Vaihtoehtoiset näkökulmat                                 │
│  └── Laatutarkastuspiste                                       │
│                                                                 │
│  YHDESSÄ                                                        │
│  ├── Keskustelu ja iterointi                                   │
│  ├── Päätösten dokumentointi                                   │
│  └── Dokumenttien kehittäminen spiraalimaisesti                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Kaksoisroolin merkitys

Claude toimii **molemmissa rooleissa ENNEN** kuin kysyy käyttäjältä:

```
Käyttäjä: "Suunnittele muistimoduuli"
                │
                ▼
┌─────────────────────────────────────────┐
│  Claude (Suunnittelija):                │
│  "Mitä kysymyksiä pitää ratkaista?"     │
│  - Miten muisti tallennetaan?           │
│  - Miten muisti haetaan?                │
│  - Miten muisti indeksoidaan?           │
└─────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│  Claude (Tutkija):                      │
│  "Mitä best practices on olemassa?"     │
│  - Hakee: RAG patterns, vector DBs      │
│  - Vertailee: Markdown vs JSON vs SQL   │
│  - Dokumentoi: RESEARCH_XX.md           │
└─────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│  Claude (Suunnittelija):                │
│  "Mikä on suositukseni?"                │
│  - Tekee päätökset joissa best practice │
│  - Listaa avoimet kysymykset käyttäjälle│
│  - Kirjoittaa SPEC-luonnoksen           │
└─────────────────────────────────────────┘
                │
                ▼
         Käyttäjä (review)
```

---

## 3. Ydinperiaatteet

### Periaate 1: Tutki ensin, kysy sitten

```
❌ VÄÄRIN:
Claude: "Haluatko tallentaa muistin Markdownina vai JSON:ina?"
(Kysyy ilman tutkimusta)

✅ OIKEIN:
Claude: "Tutkin muistitallennusvaihtoehtoja..."
        [Tekee tiedonhaun]
        "Vertailun perusteella suosittelen Markdownia koska [syyt].
         JSON olisi parempi jos [tilanne]. Kumpi sopii meille?"
(Tutkii, suosittelee, sitten kysyy)
```

**Miksi?** 
- Säästää käyttäjän aikaa
- Parempi päätöksentekoaineisto
- AI hyödyntää vahvuuksiaan (nopea tiedonhaku)

---

### Periaate 2: Vaihtoehdot + Ehdotus AINA

Kun kysytään käyttäjältä, anna **AINA**:

```
┌─────────────────────────────────────────────────────────────────┐
│  KYSYMYKSEN RAKENNE                                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Selkeä kysymys                                              │
│     "Miten tallennetaan muisti?"                                │
│                                                                 │
│  2. Vaihtoehdot taulukkona                                      │
│     | Vaihtoehto | Plussat      | Miinukset     |               │
│     |------------|--------------|---------------|               │
│     | A) Markdown| Luettava     | Ei hakua      |               │
│     | B) JSON    | Rakenteinen  | Ei luettava   |               │
│     | C) SQLite  | Haku toimii  | Monimutkaisempi|              │
│                                                                 │
│  3. Ehdotus                                                     │
│     "Ehdotukseni: A) Markdown"                                  │
│                                                                 │
│  4. Perustelu                                                   │
│     "Koska MVP:ssä luettavuus > hakutehokkuus, ja              │
│      Git-versionhallinta toimii suoraan."                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Miksi?**
- Käyttäjä näkee kokonaiskuvan
- Päätös on informoitu
- Perustelu dokumentoituu automaattisesti

---

### Periaate 3: Primitiivi-ajattelu

Jokaisen moduulin suunnittelu alkaa kysymyksellä:

> **"Mikä on tämän moduulin primitiivi - perusyksikkö jonka ympärille kaikki rakentuu?"**

```
Esimerkkejä primitiiveistä:

ClaudeService    → Message (yksittäinen viesti)
MemoryService    → MemoryItem (muistipala)
SessionService   → Session (keskustelusessio)
ContextManager   → Token (kontekstin yksikkö)
```

**Miksi?**
- Selkeyttää ajattelua
- Pakottaa yksinkertaistamaan
- Helpottaa API-suunnittelua

---

### Periaate 4: Black Box -rajapinnat

Jokainen moduuli on **musta laatikko**:

```
┌─────────────────────────────────────────────────────────────────┐
│                         BLACK BOX                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ULKOPUOLELLE NÄKYY (Public API):                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • Metodit: create(), get(), update(), delete()         │   │
│  │  • Input-tyypit: MemoryItem, SearchQuery                │   │
│  │  • Output-tyypit: MemoryItem, List[MemoryItem]          │   │
│  │  • Virhetilanteet: NotFound, ValidationError            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  SISÄLLÄ PIILOSSA (Internal):                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  • Tallennusmuoto (Markdown/JSON/SQL)                   │   │
│  │  • Tiedostorakenne                                       │   │
│  │  • Caching-strategia                                     │   │
│  │  • Optimoinnit                                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  HYÖTY: Sisäisen toteutuksen voi kirjoittaa kokonaan           │
│         uudelleen muuttamatta mitään muuta järjestelmässä.     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Miksi?**
- Moduulit ovat itsenäisiä
- Voidaan refaktoroida turvallisesti
- Selkeät vastuualueet

---

### Periaate 5: Ulkoiset riippuvuudet wrapattaan

**ÄLÄ KOSKAAN** kutsu ulkoista palvelua suoraan:

```
❌ VÄÄRIN:
response = anthropic.messages.create(...)  # Suoraan koodissa

✅ OIKEIN:
response = claude_service.send_message(...)  # Wrapperin kautta
```

**Miksi?**
- Ulkoinen API voi muuttua
- Testaus helpottuu (mock)
- Yksi paikka konfiguraatiolle

---

### Periaate 6: Dokumentoi oppiminen (RESEARCH)

Jokainen merkittävä tutkimus tallennetaan:

```
Dokumenttihierarkia:

RESEARCH_02_Memory_Patterns.md   ← Mitä opittiin tutkimuksessa
         │
         ▼
SPEC_02_Memory_Service.md        ← Mitä päätettiin
         │
         ▼
TECH_SPEC_02_Memory_Service.md   ← Miten toteutetaan
```

**Miksi?**
- Oppiminen ei katoa
- Päätösten perusteet löytyvät
- Seuraava projekti hyötyy

---

## 4. Progressive Disclosure

### Kolme tasoa

```
TASO 1: Kokonaiskuva (aina kontekstissa)
├── INDEX.md
├── MASTER_FUNCTIONAL.md
├── ARCHITECTURE_OVERVIEW.md
└── KEHITYSLOKI.md

TASO 2: Prosessit (aina kontekstissa)
├── PROCESS_SPEC_Writing.md
├── PROCESS_Document_Updates.md
└── VS_CODE_WORKFLOW.md

TASO 3: Yksityiskohdat (GitHub - ON/OFF)
├── RESEARCH_XX.md
├── SPEC_XX.md
└── TECH_SPEC_XX.md
```

**Miksi?**
- Token-budjetti pysyy hallinnassa
- Relevantit dokumentit aina saatavilla
- Yksityiskohdat haetaan tarvittaessa

---

## 5. Laadunvarmistus: Kaksi reviewia

```
SPEC valmis
     │
     ▼
┌─────────────────┐
│ Käyttäjä-review │  ← Liiketoiminta, prioriteetit
└─────────────────┘
     │
     ▼
┌─────────────────┐
│ Gemini-review   │  ← Tekninen laatu, aukot
└─────────────────┘
     │
     ▼
Viimeistely ja tallennus
```

**Miksi kaksi reviewia?**
- Käyttäjä tuntee liiketoiminnan
- Gemini antaa ulkopuolisen teknisen näkökulman
- Kaksi silmäparia löytää enemmän

---

## 6. Dokumenttien vaikutusketjut

```
SPEC_XX päivittyy
    │
    ├──► DATABASE_MODEL: Uusi taulu tai kenttä?
    │
    ├──► ARCHITECTURE_OVERVIEW: Uusi moduuli tai integraatio?
    │
    └──► MASTER_FUNCTIONAL: Päivitä status
             │
             └──► KEHITYSLOKI: Kirjaa mitä tehtiin
```

**Miksi?**
- Dokumentit pysyvät synkassa
- Muutokset eivät unohdu
- Kokonaiskuva säilyy eheänä

---

## 7. Muistisäännöt

> **"Tutki ensin, kysy sitten"**
> Älä kysy käyttäjältä ennen kuin olet tehnyt tiedonhaun.

> **"Vaihtoehdot + Ehdotus AINA"**
> Kun kysyt, anna valinnat ja oma suositus perusteluineen.

> **"Tunnista primitiivi"**
> Jokaisen moduulin ydin on yksi perusyksikkö.

> **"Black box -rajapinnat"**
> Sisäinen toteutus pitää voida kirjoittaa uudelleen.

> **"Dokumentoi oppiminen"**
> RESEARCH_XX tallentaa tutkimuksen, SPEC_XX tallentaa päätöksen.

> **"Toteutus kun kypsä"**
> Ei kiirettä koodaamaan ennen kuin suunnitelma on hyvä.

> **"Malli on geneerinen"** ← UUSI v2.1
> Metodologia säilyy ja kehittyy projektien välillä.

---

## Yhteenveto

```
┌─────────────────────────────────────────────────────────────────┐
│              AI-AVUSTEINEN OHJELMISTOSUUNNITTELU                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  FILOSOFIA                                                      │
│  └─ Iteratiivinen spiraali: dokumentit kehittyvät yhdessä      │
│  └─ "Neo-waterfall": AI mahdollistaa perusteellisen suunnittelun│
│                                                                 │
│  KUMPPANUUS                                                     │
│  └─ Claude: kaksoisrooli (suunnittelija + tutkija)             │
│  └─ Ihminen: visio, päätökset, review                          │
│  └─ Gemini: peer review, tekninen validointi                   │
│                                                                 │
│  PERIAATTEET                                                    │
│  └─ Tutki ensin, kysy sitten                                   │
│  └─ Vaihtoehdot + Ehdotus AINA                                 │
│  └─ Primitiivi-ajattelu                                        │
│  └─ Black box -rajapinnat                                      │
│  └─ Ulkoiset riippuvuudet wrapattaan                           │
│  └─ Dokumentoi oppiminen (RESEARCH)                            │
│                                                                 │
│  GENEERISYYS                                                    │
│  └─ Malli on metodologia, ei projektikohtainen                 │
│  └─ Dokumentaatio englanniksi, keskustelu käyttäjän kielellä   │
│                                                                 │
│  LAATU                                                          │
│  └─ Käyttäjä-review + Gemini-review                            │
│                                                                 │
│  KÄYTÄNTÖ                                                       │
│  └─ Katso: PROCESS_SPEC_Writing.md                             │
│  └─ Katso: META_Development_Model.md                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 2.1 | 2025-11-30 | Lisätty: Mallin geneerisyys -fundamentti, kielikäytäntö (suomi/englanti), "Neo-waterfall" selitys, viittaus META_Development_Model.md, Gemini peer review rooliin |
| 2.0 | 2025-11-25 | Uudelleenkirjoitus: fokus filosofiaan ja periaatteisiin. Lisätty: kaksoisrooli, "tutki ensin", vaihtoehdot+ehdotus, primitiivi-ajattelu, black box, RESEARCH. Poistettu: käytännön prosessikuvaukset (→ PROCESS_SPEC_Writing.md) |
| 1.0 | 2025-05-22 | Ensimmäinen versio |

---

*Tämä dokumentti kuvaa suunnittelufilosofiaa. Käytännön prosessi: PROCESS_SPEC_Writing.md*
