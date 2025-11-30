# Prosessiohje: Dokumenttien päivitys

> **Versio:** 1.7  
> **Päivitetty:** 2025-11-25  
> **Tiedosto:** v1_7_PROCESS_Document_Updates.md  
> **Tarkoitus:** Varmistaa dokumenttien yhtenäisyys ja estää versio-ongelmat  
> **Edellinen:** v1.6 (2025-11-25)

---

## Versionumerointi (Hybridimalli)

### Periaate: Tuplasuojaus

Versionumero merkitään **SEKÄ** tiedostonimeen **ETTÄ** dokumentin sisälle:

```
Tiedostonimi:
v1_2_DATABASE_MODEL_CORE.md
│
└─► Versio näkyy heti tiedostoa valitessa

Dokumentin otsikko:
┌─────────────────────────────────────────┐
│ # DATABASE_MODEL_CORE                   │
│                                         │
│ > **Versio:** 1.2                       │
│ > **Päivitetty:** 2025-05-22            │
│ > **Tiedosto:** v1_2_DATABASE_MODEL_... │
└─────────────────────────────────────────┘
│
└─► Versio näkyy dokumenttia lukiessa
```

### Miksi hybridimalli?

| Tilanne | Suojaus |
|---------|---------|
| Käyttäjä valitsee tiedostoa | Näkee version tiedostonimestä |
| Claude lukee dokumenttia | Näkee version otsikosta |
| Ristiriita (nimi ≠ sisältö) | Molemmat huomaavat virheen |

### Tiedostojen nimeämiskäytäntö

```
v[MAJOR]_[MINOR]_DOKUMENTIN_NIMI.md
```

**Esimerkit:**
```
v1_0_MASTER_FUNCTIONAL.md
v1_0_ARCHITECTURE_OVERVIEW.md
v1_2_DATABASE_MODEL_CORE.md
v1_4_PROCESS_Document_Updates.md
```

### Dokumentin otsikkoformaatti

Jokaisen dokumentin alussa AINA:

```markdown
# DOKUMENTIN_NIMI

> **Versio:** X.Y  
> **Päivitetty:** YYYY-MM-DD  
> **Tiedosto:** vX_Y_DOKUMENTIN_NIMI.md  
> **Edellinen:** vX.Z (YYYY-MM-DD) [jos päivitys]

---
```

### Versionumeroinnin säännöt

| Muutoksen tyyppi | Versiomuutos | Esimerkki |
|------------------|--------------|-----------|
| Uusi dokumentti | v1.0 | v1_0_MASTER_FUNCTIONAL.md |
| Pieni päivitys (tarkennus, korjaus, lisäys) | +0.1 | v1.0 → v1.1 |
| Merkittävä päivitys (uusi osio, rakenne muuttuu) | +1.0 | v1.5 → v2.0 |

### Version päivitysprosessi

```
1. Claude luo päivitetyn dokumentin UUDELLA versionumerolla
   → v1_1_DOKUMENTTI.md (uusi tiedosto)
   → Otsikossa: Versio: 1.1, Edellinen: v1.0

2. Käyttäjä lataa uuden version

3. Käyttäjä POISTAA vanhan version projektista
   → Poista v1_0_DOKUMENTTI.md

4. Käyttäjä LISÄÄ uuden version projektiin
   → Lisää v1_1_DOKUMENTTI.md

⚠️ TÄRKEÄÄ: Älä jätä vanhoja versioita projektiin!
```

### 🚀 Tehokas päivitysprosessi (str_replace -malli)

**Periaate:** Käytä `str_replace`-työkalua pieniin päivityksiin, kirjoita uudelleen vain kun välttämätöntä.

#### Milloin käyttää str_replace?

| Tilanne | Päivitystapa |
|---------|--------------|
| **1-5 kohdennettua muutosta** | str_replace + nimeä uudelleen |
| **>5 muutosta TAI rakennemuutos** | Kirjoita koko dokumentti uudelleen |
| **Epävarma/monimutkainen** | Kirjoita uudelleen (turvallisempi) |

#### str_replace -prosessi

```
1. Kopioi vanha tiedosto outputs-kansioon
   → cp v1_9_INDEX.md → outputs/v1_9_INDEX.md

2. ENNEN muutoksia: Etsi kaikki päivitettävät kohdat
   → grep -n "v1_9_INDEX" tiedosto.md
   → Kirjaa ylös montako kohtaa (esim. 4 kpl)

3. Käytä str_replace päivittämään vain muuttuneet kohdat
   → Versio otsikossa: 1.9 → 1.10
   → Tiedostonimi viitteissä: v1_9 → v1_10
   → Muutoshistoria: lisää uusi rivi
   → Sisältömuutokset

4. JÄLKEEN muutoksia: Varmista että kaikki päivittyi
   → grep -n "v1_9_INDEX" tiedosto.md  (pitäisi olla 0 tulosta)
   → grep -n "v1_10_INDEX" tiedosto.md (pitäisi olla 4 tulosta)

5. Kun valmis, nimeä uudelleen
   → mv v1_9_INDEX.md → v1_10_INDEX.md
   → TAI cp + rm

6. Käyttäjä lataa uuden version
```

#### ⚠️ Fail Safe -tarkistukset

**ENNEN str_replace:**
```bash
# Listaa KAIKKI kohdat jotka pitää päivittää
grep -n "vanha_versio" tiedosto.md

# Tulostaa esim:
# 3:> **Tiedosto:** v1_9_INDEX.md
# 27:│  │  v1_9_INDEX.md              📍 Olet tässä
# 128:| INDEX | `v1_9_INDEX.md` | 📍 Navigointikartta |
# → 3 kohtaa päivitettävä
```

**JÄLKEEN str_replace:**
```bash
# Varmista ettei vanhoja jäänyt
grep -n "vanha_versio" tiedosto.md
# → Pitäisi palauttaa 0 tulosta!

# Varmista että uudet löytyvät
grep -n "uusi_versio" tiedosto.md
# → Pitäisi palauttaa sama määrä kuin yllä (3 kpl)
```

**Jos grep löytää vanhoja viittauksia → str_replace unohtui!**

#### 📝 Oikoluku (dokumentin valmistuttua)

Kun kaikki muutokset on tehty, Claude tekee **koko dokumentin** oikoluvun:

```
1. LUE KOKO DOKUMENTTI LÄPI (view-komennolla osissa)
   → view tiedosto.md [1, 100]
   → view tiedosto.md [100, 200]
   → ... kunnes koko dokumentti luettu
   
   Etsi samalla:
   □ Vanhentuneet versionumerot (esim. "v1.4" kun pitäisi olla "v1.6")
   □ Vanhentuneet päivämäärät
   □ Vanhentuneet tiedostonimiviittaukset
   □ Ristiriitaiset tiedot eri osioissa
   □ Keskeneräiset lauseet tai TODO-merkinnät

2. Tarkista otsikko
   □ Versio oikein?
   □ Päivämäärä oikein?
   □ Tiedostonimi vastaa otsikkoa?

3. Tarkista sisältö
   □ Kaikki viittaukset päivitetty? (grep-tarkistus)
   □ Muutoshistoriassa uusi rivi?
   □ UTF-8 encoding OK? (ä/ö/å näkyvät)

4. Tarkista rakenne
   □ Markdown renderöityy oikein?
   □ Taulukot ehjät? (sarakkeet täsmäävät)
   □ Koodilohkot suljettu? (``` -parit)
   □ Otsikkotasot johdonmukaiset? (##, ###, ####)

5. Raportoi käyttäjälle
   ┌─────────────────────────────────────────────────┐
   │ OIKOLUKU: [tiedostonimi]                        │
   ├─────────────────────────────────────────────────┤
   │ Rivejä luettu:        XXX                       │
   │ Taulukoita:           X (kaikki ehjät / virhe)  │
   │ Koodilohkoja:         X (kaikki suljettu / virhe)│
   │ UTF-8 merkit:         ✅ OK / ❌ Ongelma        │
   │ Versioviittaukset:    ✅ OK / ❌ Vanhoja jäljellä│
   │                                                 │
   │ TULOS: ✅ OK / ❌ Korjattavaa kohdassa X        │
   └─────────────────────────────────────────────────┘
```

**TÄRKEÄÄ:** Älä ohita oikolukua! Pelkkä grep ei löydä:
- Esimerkeissä olevia vanhentuneita versioita
- Tekstin seassa olevia ristiriitoja
- Rakenteellisia ongelmia (rikkinäiset taulukot)

#### Hyödyt

| Vanha tapa | str_replace -tapa |
|------------|-------------------|
| 258 riviä uudelleen | 4 kohdennettua muutosta |
| Riski: sisältö muuttuu vahingossa | Vain halutut kohdat muuttuvat |
| Hidas | Nopea |
| Virhealttius | Tarkka |

#### Esimerkki: INDEX v1.9 → v1.10

```python
# Muutos 1: Versio otsikossa
str_replace(
    old="> **Versio:** 1.9",
    new="> **Versio:** 1.10"
)

# Muutos 2: Tiedostonimi
str_replace(
    old="v1_9_INDEX.md",
    new="v1_10_INDEX.md"
)

# Muutos 3: Muutoshistoria
str_replace(
    old="| 1.9 | 2025-11-25 |",
    new="| 1.10 | 2025-11-25 | ... |\n| 1.9 | 2025-11-25 |"
)

# Lopuksi: Nimeä uudelleen
mv v1_9_INDEX.md → v1_10_INDEX.md
```

### Viittaukset dokumenttien välillä

Koska tiedostonimet muuttuvat, ÄLÄ käytä suoria linkkejä:

```markdown
❌ VÄÄRIN (rikkoutuu):
Katso: [DATABASE_MODEL](v1.2_DATABASE_MODEL_CORE.md)

✅ OIKEIN:
Katso: DATABASE_MODEL_CORE (projektin tiedostot)

✅ OIKEIN (GitHub):
Täysi dokumentti: GitHub → docs/specs/SPEC_01_Claude_Service.md
```

---

## 🔤 Merkistökoodaus (UTF-8) - KRIITTINEN!

### Sääntö: Kaikki dokumentit UTF-8

**Kaikki .md-tiedostot luodaan AINA UTF-8 -merkistökoodauksella.**

### Miksi tämä on tärkeää?

| Ongelma | Seuraus |
|---------|---------|
| Väärä encoding | ä → Ã¤, ö → Ã¶, å → Ã¥ |
| Dokumentti lukukelvoton | Käyttäjä ei ymmärrä sisältöä |
| Vaikea korjata | Täytyy kirjoittaa uudelleen |

### Tarkistuslista ennen tallennusta

```
✅ Tarkista että nämä merkit näkyvät oikein:
   - ä, ö, å (skandinaaviset)
   - Ä, Ö, Å (isot skandinaaviset)
   - € (euro-merkki)
   - → ← ↑ ↓ (nuolet)
   - ✅ ❌ ⚠️ 🔴 🟡 🟢 (emojit)
```

### Claude: Miten varmistat UTF-8:n?

1. **Käytä suomenkielisiä merkkejä suoraan** - älä korvaa esim. "ä" → "a"
2. **Tarkista dokumentti silmämääräisesti** ennen tallennusta
3. **Jos epäilet ongelmaa**, luo tiedosto uudelleen

### Käyttäjä: Miten tunnistat encoding-ongelman?

```
Normaali teksti:        Rikkinäinen teksti:
─────────────────       ─────────────────
Päivitetty              PÃ¤ivitetty
Työnkulku               TyÃ¶nkulku
Käyttäjä                KÃ¤yttÃ¤jÃ¤
```

**Jos näet "Ã¤" tai "Ã¶"** → Käytä alla olevaa workaroundia.

### ⚠️ Workaround: Jos encoding on rikki projektiin lisäyksen jälkeen

**Ongelma:** Lataamasi .md-tiedosto näyttää oikein paikallisesti, mutta projektiin lisäyksen jälkeen merkit ovat rikki (ä → Ã¤).

**Tunnettu bugi:** Tämä on Windows + Claude -ympäristön tunnettu ongelma (GitHub issues #1716, #10709, #7134).

**Korjaus - Vaihtoehto A (suositeltu):**
1. Avaa Clauden luoma tiedosto tekstieditorissa (esim. Notepad++)
2. Kopioi KOKO sisältö (Ctrl+A, Ctrl+C)
3. Luo uusi tekstitiedosto projektissa (Add content → Text)
4. Liitä sisältö (Ctrl+V)
5. Tallenna oikealla tiedostonimellä (esim. v1_4_PROCESS_Document_Updates.md)

**Korjaus - Vaihtoehto B:**
1. Pyydä Claudea tulostamaan dokumentin sisältö suoraan chattiin
2. Kopioi teksti chatista
3. Luo tekstitiedosto manuaalisesti projektiin

**Tarkistus tallennuksen jälkeen:**
1. Pyydä Claudea: "Tarkista projektin tiedosto [NIMI] - onko encoding kunnossa?"
2. Claude hakee tiedoston ja raportoi onko ä/ö/å oikein

---

## Ydindokumentit ja päivitystriggerit

### Taso 1: Kokonaiskuva (projektissa aina)

| Dokumentti | Päivitä kun... | Kriittisyys |
|------------|----------------|-------------|
| **MASTER_FUNCTIONAL** | Uusi moduuli, status muuttuu, AI-rooli muuttuu | 🔴 Korkea |
| **ARCHITECTURE_OVERVIEW** | Uusi integraatio, tekninen päätös, moduulirakenne | 🔴 Korkea |
| **DATABASE_MODEL_CORE** | Uusi taulu, tärkeä kenttämuutos | 🔴 Korkea |
| **KEHITYSLOKI** | Jokainen työjakso, päätös, backlog-muutos | 🟡 Keskitaso |

### Taso 2: Yksityiskohdat (GitHub)

| Dokumentti | Päivitä kun... | Kriittisyys |
|------------|----------------|-------------|
| **SPEC_XX_[nimi]** | Uusi toiminnallisuus, validointi, AI-logiikka | 🔴 Korkea |
| **DATABASE_MODEL_FULL** | Mikä tahansa tietokantamuutos | 🔴 Korkea |
| **DATABASE_CHANGELOG** | Jokainen DB-muutos (ennen MODEL-päivitystä) | 🔴 Korkea |

---

## Päivitysprosessi

### Vaihe 1: Tunnista muutoksen tyyppi

```
┌─────────────────────────────────────────────────────────────┐
│  MUUTOKSEN TYYPPI           → PÄIVITETTÄVÄT DOKUMENTIT     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Uusi moduuli aloitetaan    → MASTER_FUNCTIONAL            │
│                             → KEHITYSLOKI                   │
│                                                             │
│  SPEC valmistuu             → MASTER_FUNCTIONAL (status)    │
│                             → KEHITYSLOKI                   │
│                             → GitHub (täysi SPEC)           │
│                                                             │
│  Tietokantamuutos           → DATABASE_CHANGELOG (ensin!)   │
│                             → DATABASE_MODEL_CORE           │
│                             → DATABASE_MODEL_FULL (GitHub)  │
│                             → SPEC (jos vaikuttaa)          │
│                                                             │
│  Arkkitehtuuripäätös        → ARCHITECTURE_OVERVIEW         │
│                             → KEHITYSLOKI                   │
│                             → MASTER_FUNCTIONAL (jos vaik.) │
│                                                             │
│  AI-logiikka muuttuu        → SPEC (yksityiskohdat)         │
│                             → MASTER_FUNCTIONAL (yleiskuva) │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Vaihe 2: Päivitysjärjestys

**KRIITTINEN:** Noudata aina tätä järjestystä!

```
1. DATABASE_CHANGELOG      ← Kirjaa muutos ensin lokiin
2. DATABASE_MODEL          ← Päivitä rakenne
3. SPEC_XX                 ← Päivitä yksityiskohdat
4. MASTER_FUNCTIONAL       ← Päivitä yleiskuva
5. KEHITYSLOKI             ← Kirjaa mitä tehtiin
```

### Vaihe 3: Päivityksen suoritus

**Claude tekee:**
1. Lukee nykyisen version
2. Tekee muutokset
3. Päivittää versionumeron (nimi JA otsikko)
4. **Varmistaa UTF-8 encodingin**
5. Luo tiedoston → `/mnt/user-data/outputs/`
6. Antaa latauslinkin

**Käyttäjä tekee:**
1. Lataa päivitetyn tiedoston
2. **Tarkistaa että ä/ö/å näkyvät oikein**
3. POISTAA vanhan version projektista
4. LISÄÄ uuden version projektiin
5. **Pyytää Claudea tarkistamaan encodingin**
6. TAI lataa GitHubiin (täydet dokumentit)

---

## Tarkistuslista: Ennen jokaista muutosta

### Kysymykset

- [ ] Mihin dokumentteihin tämä muutos vaikuttaa?
- [ ] Onko DATABASE_CHANGELOG päivitetty (jos DB-muutos)?
- [ ] Onko MASTER_FUNCTIONAL ajan tasalla?
- [ ] Onko KEHITYSLOKI päivitetty?
- [ ] Onko GitHubin dokumentti päivitettävä?
- [ ] Mikä on uusi versionumero?
- [ ] Onko versio SEKÄ nimessä ETTÄ otsikossa?
- [ ] **Onko encoding UTF-8 (ä/ö/å toimivat)?**

### Yhtenäisyystarkistus

- [ ] Käytetäänkö samoja termejä kaikissa dokumenteissa?
- [ ] Ovatko taulujen/kenttien nimet yhtenäisiä?
- [ ] Ovatko statukset ajan tasalla?
- [ ] Ovatko riippuvuudet merkitty oikein?
- [ ] Täsmääkö tiedostonimi ja otsikon versio?

---

## Konfliktien välttäminen

### Sääntö 1: Yksi totuuden lähde

| Tieto | Totuuden lähde | Muut viittaavat |
|-------|----------------|-----------------|
| Taulurakenne | DATABASE_MODEL | SPECit viittaavat |
| Toiminnallisuus (yleiskuva) | MASTER_FUNCTIONAL | - |
| Toiminnallisuus (yksityiskohdat) | SPEC_XX | MASTER viittaa |
| Päätökset | KEHITYSLOKI | - |
| AI-logiikka | SPEC_XX | MASTER tiivistää |

### Sääntö 2: Älä duplikoi yksityiskohtia

```
❌ VÄÄRIN:
MASTER_FUNCTIONAL:
"Ostolaskun kentät: invoice_number VARCHAR(50), invoice_date DATE..."

✅ OIKEIN:
MASTER_FUNCTIONAL:
"Kentät: Ks. SPEC_02 ja DATABASE_MODEL"
```

### Sääntö 3: Päivitä ylhäältä alas

```
Tietokantamuutos havaittu
        │
        ▼
DATABASE_CHANGELOG (kirjaa löydös)
        │
        ▼
DATABASE_MODEL (päivitä rakenne)
        │
        ▼
SPEC_XX (päivitä viittaukset)
        │
        ▼
MASTER_FUNCTIONAL (päivitä status/yleiskuva)
        │
        ▼
KEHITYSLOKI (kirjaa mitä tehtiin)
```

---

## Muistisäännöt

> **Versio AINA sekä tiedostonimeen ETTÄ dokumentin otsikkoon**
> ```
> Tiedosto: v1_2_DOKUMENTTI.md
> Otsikko:  > **Versio:** 1.2
> ```

> **Encoding AINA UTF-8 - tarkista ä/ö/å ennen JA jälkeen tallennuksen**

> **Jos encoding rikki → kopioi teksti ja luo uusi tiedosto**

> **Muutos → Changelog → Model → Spec → Master → Loki**

> **Yksi totuuden lähde per tieto. Muut viittaavat.**

> **Poista vanha versio AINA kun lisäät uuden!**

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 1.7 | 2025-11-25 | Oikoluku laajennettu koko dokumentin läpikäynniksi (view osissa) |
| 1.6 | 2025-11-25 | Lisätty fail safe -tarkistukset (grep) ja oikoluku str_replace-prosessiin |
| 1.5 | 2025-11-25 | Lisätty tehokas päivitysprosessi (str_replace -malli) |
| 1.4 | 2025-11-25 | Lisätty workaround encoding-ongelmaan, tarkistusprosessi |
| 1.3 | 2025-11-25 | Lisätty UTF-8 encoding -osio ja tarkistukset |
| 1.2 | 2025-05-22 | Hybridimalli: versio sekä nimeen että otsikkoon |
| 1.1 | 2025-05-22 | Lisätty versionumerointiohjeistus |
| 1.0 | 2025-05-22 | Ensimmäinen versio |

---

*Dokumentti on osa projektin prosessiohjeistusta.*
