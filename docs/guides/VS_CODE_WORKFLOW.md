# VS Code + Claude Code + GitHub - Työnkulkuohje

> **Versio:** 2.2  
> **Päivitetty:** 2025-11-25  
> **Projekti:** Claude API -suunnittelutyökalu  
> **Työnkulku:** Hybridimalli (Claude.ai + Claude Code)

---

## 📍 Projektikohtaiset tiedot

```
GitHub-käyttäjä:     jussivares
GitHub-repo:         https://github.com/jussivares/claude-planning-tool
Paikallinen polku:   C:\Users\Jussi\ClaudeProjektit\claude-planning-tool\
Repo-kansio:         claude-planning-tool/
```

---

## 🎯 Hybridityönkulku (SUOSITELTU)

```
Claude.ai (kirjoittaa) + Claude Code (committaa) = Tehokkain tapa
```

**Periaate:**
1. ✅ **Claude.ai** kirjoittaa dokumentit (rikas konteksti)
2. ✅ **Claude.ai** kirjoittaa valmiin Git-promptin (kopioi sellaisenaan)
3. ✅ **VS Code** tallennat tiedostot
4. ✅ **Claude Code** hoitaa Git-operaatiot (kopioi Clauden prompt)
5. ✅ **GitHub** säilyttää historian automaattisesti

---

## 📋 Vaihe 1: Perustaminen (kerran)

### 1.1 Kloonaa repo VS Codeen

**Vaihtoehto A: VS Coden GUI (helpoin)**

1. Avaa VS Code
2. Klikkaa **"Clone Repository"**
3. Liitä repo URL:
   ```
   https://github.com/jussivares/claude-planning-tool.git
   ```
4. Valitse tallennuskansio: `C:\Users\Jussi\ClaudeProjektit\`
5. Klikkaa **"Open"** kun valmis

**Vaihtoehto B: Terminaali**

```bash
cd C:\Users\Jussi\ClaudeProjektit
git clone https://github.com/jussivares/claude-planning-tool.git
cd claude-planning-tool
code .
```

### 1.2 Avaa Claude Code chat

**Näppäin:** `Ctrl+L`

**Tai:** Klikkaa oikealla olevaa 💬 **CHAT** -ikonia

✅ **Valmis!** Olet nyt valmis käyttämään hybridityönkulkua.

---

## 🚀 Vaihe 2: Session aloitus (AINA ENSIN)

> **TÄRKEÄ:** Aloita JOKAINEN VS Code -sessio tällä tarkistuksella!

### Avausprompt Claude Codelle

Kun avaat VS Coden ja projektin, kopioi tämä prompt Claude Code -chattiin:

```
Näytä projektin tila:
1. git status (onko uncommitted muutoksia?)
2. git log --oneline -3 (viimeisimmät commitit)
3. git fetch && git status (onko GitHubissa uudempia?)

ÄLÄ tee muutoksia, vain raportoi tilanne.
```

### Mitä tarkistaa vastauksesta

| Tilanne | Toimenpide |
|---------|------------|
| ✅ "Working tree clean" | Kaikki OK, voit aloittaa |
| ⚠️ "Uncommitted changes" | Commitoi tai hylkää ennen uutta työtä |
| ⚠️ "Your branch is behind" | Vedä muutokset: `git pull` |
| ⚠️ "Your branch is ahead" | Pushaa muutokset: `git push` |

### Jos uncommitted muutoksia

**Vaihtoehto A: Commitoi ne**
```
Commitoi kaikki uncommitted muutokset.
Message: "WIP: Uncommitted changes from previous session"
```

**Vaihtoehto B: Hylkää ne**
```
Hylkää kaikki uncommitted muutokset (git checkout .)
VAROITUS: Tämä poistaa muutokset pysyvästi!
```

---

## 🔄 Vaihe 3: Päivittäinen työnkulku

### Perusprosessi (toista tätä)

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  1. Claude.ai → Kirjoita dokumentti                         │
│  2. Lataa tiedosto                                          │
│  3. VS Code → Tallenna oikeaan paikkaan                     │
│  4. Claude Code → Kopioi valmis prompt (alta)               │
│  5. GitHub → Historia päivittyy automaattisesti             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔥 VALMIIT PROMPTIT - Miten tämä toimii

> **KRIITTINEN MUUTOS v2.1:**  
> **Claude.ai (minä) kirjoitan AINA valmiin promptin sinulle.**  
> **Sinä vain KOPIOIT sen sellaisenaan Claude Code -chattiin.**

### Työnkulku

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  1. Claude.ai (minä):                                       │
│     "Tässä on valmis prompt Claude Codelle:"                │
│                                                             │
│     ┌─────────────────────────────────────────────────────┐ │
│     │ Commitoi ja pushaa:                                 │ │
│     │ docs/specs/SPEC_02_Memory_Service.md                │ │
│     │ Message: "Add SPEC_02 v1.0: ..."                    │ │
│     └─────────────────────────────────────────────────────┘ │
│                                                             │
│  2. Sinä:                                                   │
│     - Kopioi KOKO promptin (Ctrl+C)                         │
│     - Avaa Claude Code chat (Ctrl+L)                        │
│     - Liitä prompt (Ctrl+V)                                 │
│     - Enter                                                 │
│                                                             │
│  3. Claude Code:                                            │
│     ✅ Committaa tiedoston                                  │
│     ✅ Pushaa GitHubiin                                     │
│     ✅ Näyttää vahvistuksen                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Miksi tämä on parempi?

| Ennen | Nyt |
|-------|-----|
| ❌ Sinun täytyy muokata [TIEDOSTONIMI] | ✅ Minä tiedän oikean tiedostonimen |
| ❌ Sinun täytyy muokata [VIESTI] | ✅ Minä tiedän commit messagen |
| ❌ Mahdollisuus kirjoitusvirheisiin | ✅ Ei virheitä - kopioi vain |
| ❌ Täytyy muistaa oikea polku | ✅ Minä tiedän projektin rakenteen |

### Esimerkki (todellinen tapaus)

**Sinä sanot:**
```
"Olen tallentanut SPEC_02:n VS Codeen. Tee commit ja push."
```

**Minä vastaan:**
```
Tässä on valmis prompt Claude Codelle:

┌─────────────────────────────────────────────────────────────┐
│ Commitoi ja pushaa:                                         │
│ docs/specs/SPEC_02_Memory_Service.md                        │
│                                                             │
│ Message: "Add SPEC_02 v1.0: MemoryService specification"    │
└─────────────────────────────────────────────────────────────┘

Kopioi tämä Claude Code -chattiin.
```

**Sinä:**
1. Kopioi koko laatikon sisältö
2. Liitä Claude Code -chattiin
3. Enter
4. Valmis!

---

## 📝 ESIMERKKEJÄ VALMIISTA PROMPTEISTA

> **Huom:** Nämä ovat esimerkkejä mitä MINÄ (Claude.ai) kirjoitan sinulle.  
> Älä kopioi näitä suoraan - pyydä minua kirjoittamaan oikea prompt kulloiseenkin tilanteeseen.

---

### Esimerkki 1: Uusi dokumentti

**Tilanne:** Olet kirjoittanut SPEC_02:n v1.0 ja tallentanut sen.

**Minä kirjoitan:**
```
┌─────────────────────────────────────────────────────────────┐
│ Commitoi ja pushaa:                                         │
│ docs/specs/SPEC_02_Memory_Service.md                        │
│                                                             │
│ Message: "Add SPEC_02 v1.0: MemoryService specification"    │
└─────────────────────────────────────────────────────────────┘
```

---

### Esimerkki 2: Dokumentin päivitys

**Tilanne:** Olet päivittänyt SPEC_02:n versioon 1.1.

**Minä kirjoitan:**
```
┌─────────────────────────────────────────────────────────────┐
│ Commitoi ja pushaa:                                         │
│ docs/specs/SPEC_02_Memory_Service.md                        │
│                                                             │
│ Message: "Update SPEC_02 to v1.1: Add retry strategy"       │
└─────────────────────────────────────────────────────────────┘
```

---

### Esimerkki 3: Usean tiedoston päivitys

**Tilanne:** Olet päivittänyt INDEX.md, KEHITYSLOKI.md ja SPEC_02.md.

**Minä kirjoitan:**
```
┌───────────────────────────────────────────────────────────────────┐
│ Commitoi ja pushaa KAIKKI muuttuneet tiedostot seuraavista:       │
│                                                                   │
│ - v1.4_INDEX.md                                                   │
│ - v1.1_KEHITYSLOKI.md                                             │
│ - docs/specs/SPEC_02_Memory_Service.md                            │
│                                                                   │
│ Message: "Update documentation: INDEX v1.4, KEHITYSLOKI v1.1,     │
│          SPEC_02 v1.1"                                            │
└───────────────────────────────────────────────────────────────────┘
```

---

### Esimerkki 4: Pieni korjaus

**Tilanne:** Korjasit typon SPEC_02:ssa.

**Minä kirjoitan:**
```
┌─────────────────────────────────────────────────────────────────────┐
│ Commitoi ja pushaa:                                                 │
│ docs/specs/SPEC_02_Memory_Service.md                                │
│                                                                     │
│ Message: "Fix typo in SPEC_02: 'persistenss' → 'persistence'"       │
└─────────────────────────────────────────────────────────────────────┘
```

---

### Esimerkki 5: Tarkista ensin (turvallisuus)

**Tilanne:** Haluat nähdä mitä muutoksia on ennen committia.

**Minä kirjoitan:**
```
┌─────────────────────────────────────────────────────────────┐
│ Näytä git status ja git diff                                │
│                                                             │
│ ÄLÄ tee committia vielä, vain näytä muutokset.              │
└─────────────────────────────────────────────────────────────┘
```

**Kun olet tarkistanut:**

```
┌─────────────────────────────────────────────────────────────┐
│ OK, commitoi nyt:                                           │
│ [TIEDOSTOPOLKU]                                             │
│                                                             │
│ Message: "[VIESTI]"                                         │
└─────────────────────────────────────────────────────────────┘
```

---

### Esimerkki 6: Peruuta viimeisin commit

**Tilanne:** Teit commitin väärällä messagella.

**Minä kirjoitan:**
```
┌─────────────────────────────────────────────────────────────┐
│ Näytä ensin viimeisin commit:                               │
│ git log --oneline -1                                        │
│                                                             │
│ Sitten peruuta se (soft reset):                             │
│ git reset --soft HEAD~1                                     │
│                                                             │
│ Tämän jälkeen muutokset ovat stagessa, voit commitoida      │
│ uudelleen oikealla messagella.                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 Hyvät promptit vs huonot promptit

### ✅ Hyvät (käytä näitä malleja)

```
✅ "Commitoi ja pushaa: docs/specs/SPEC_02.md
    Message: 'Add SPEC_02 v1.0: MemoryService'"
   → Tarkka tiedosto + selkeä message

✅ "Näytä git status ja git diff (ÄLÄ commitoi)"
   → Turvallinen tarkistus ensin

✅ "Commitoi KAIKKI muuttuneet tiedostot:
    Message: 'Update docs: INDEX, KEHITYSLOKI, SPEC_02'"
   → Batch-commit kun useita tiedostoja
```

### ❌ Huonot (vältä näitä)

```
❌ "Commitoi tiedosto"
   → Claude Code ei tiedä mikä tiedosto

❌ "Pushaa muutokset"
   → Claude Code ei tiedä commit messagea

❌ "Päivitä dokumentti"
   → Ei riittävän tarkka
```

---

## 🛡️ Turvallisuustarkistukset (AINA)

### Ennen committia, tarkista:

1. **Tiedosto oikeassa paikassa?**
   ```
   ✅ C:\Users\Jussi\ClaudeProjektit\claude-planning-tool\docs\specs\SPEC_02.md
   ❌ C:\Users\Jussi\Downloads\SPEC_02.md
   ```

2. **Oikea tiedostonimi?**
   ```
   ✅ SPEC_02_Memory_Service.md (GitHub)
   ✅ v1.3_INDEX.md (projekti)
   ```

3. **Commit message selkeä?**
   ```
   ✅ "Add SPEC_02 v1.0: MemoryService specification"
   ❌ "Update file"
   ```

4. **Versionumero dokumentissa?**
   ```markdown
   > **Versio:** 1.0
   > **Päivitetty:** 2025-11-25
   ```

---

## 🎯 Claude Code vastausten tulkinta

### ✅ Onnistunut commit

```
Claude Code sanoo:

✔ Staged: docs/specs/SPEC_02_Memory_Service.md
✔ Committed: "Add SPEC_02 v1.0: MemoryService specification"
✔ Pushed to origin/main

→ VALMIS! Tarkista GitHub.
```

### ⚠️ Merge conflict

```
Claude Code sanoo:

✗ Error: Merge conflict in docs/specs/SPEC_02_Memory_Service.md

→ TOIMENPIDE:
1. Vedä muutokset: git pull
2. Ratkaise konfliktit manuaalisesti
3. Commitoi ratkaisut
```

### ❌ Virhe: Ei muutoksia

```
Claude Code sanoo:

✗ Nothing to commit, working tree clean

→ TOIMENPIDE:
Tarkista että olet tallentanut tiedoston VS Codessa (Ctrl+S)
```

---

## 🔄 Yleisimmät virhetilanteet ja korjaukset

### Virhe 1: "Push rejected"

**Syy:** GitHub on edellä paikallista repoasi.

**Korjaa:**
```
Claude Code:lle: "Vedä uusimmat muutokset GitHubista ja yritä uudelleen"

Tai:
git pull
git push
```

### Virhe 2: "Authentication failed"

**Syy:** Git-tunnukset puuttuvat.

**Korjaa:**
```bash
git config --global user.name "Jussi Vares"
git config --global user.email "sähköposti@example.com"
```

### Virhe 3: "File not found"

**Syy:** Väärä tiedostopolku.

**Korjaa:**
```
Claude Code:lle: "Näytä projektin tiedostorakenne"

Tarkista oikea polku ja yritä uudelleen.
```

### Virhe 4: "Commit message tyhjä"

**Syy:** Unohdit commit messagen.

**Korjaa:**
```
Claude Code:lle: "Peruuta edellinen commit ja tee uudelleen
messagella: '[OIKEA MESSAGE]'"
```

---

## 📊 Commit message -käytännöt

### Hyvät commit messaget (käytä näitä malleja)

```
✅ "Add [DOKUMENTTI] v[VERSIO]: [LYHYT KUVAUS]"
   Esim: "Add SPEC_02 v1.0: MemoryService specification"

✅ "Update [DOKUMENTTI] to v[VERSIO]: [MUUTOS]"
   Esim: "Update SPEC_02 to v1.1: Add retry strategy"

✅ "Fix [MIKÄ VIRHE] in [DOKUMENTTI]"
   Esim: "Fix typo in SPEC_02: correct 'persistenss' to 'persistence'"

✅ "Rename: [VANHA] → [UUSI] - [SYY]"
   Esim: "Rename: Remove version prefix from SPEC_01 (use Git history)"

✅ "Remove [DOKUMENTTI] - [SYY]"
   Esim: "Remove old INDEX v1.0 - replaced by v1.2"
```

### Huonot commit messaget (VÄLTÄ)

```
❌ "Update"
❌ "Fix"
❌ "Changes"
❌ "asdf"
❌ "WIP"
```

---

## 🎓 Edistyneet käyttötapaukset

### Multi-step workflow

```
Claude Code:lle:

Tee seuraavat askeleet järjestyksessä:

1. Luo .gitignore jos ei ole
2. Commitoi: docs/specs/SPEC_02_Memory_Service.md
   Message: "Add SPEC_02 v1.0"
3. Commitoi: v1.3_INDEX.md
   Message: "Update INDEX to v1.3"
4. Push molemmat commitit
5. Näytä yhteenveto

Vahvista jokainen vaihe ennen seuraavaa.
```

### Conditional commit

```
Claude Code:lle:

1. Tarkista onko docs/specs/SPEC_03_Context_Manager.md muuttunut
2. JOS on muuttunut:
   - Commitoi message: "Update SPEC_03 to v1.1"
3. JOS EI ole muuttunut:
   - Älä tee mitään, ilmoita minulle
```

---

## 📚 Quick reference (pidä esillä)

| Toiminto | Prompt Claude Codelle |
|----------|-----------------------|
| **Session aloitus** | `Näytä: git status, git log -3, git fetch && git status` |
| **Uusi dokumentti** | `Commitoi ja pushaa: [POLKU]` <br> `Message: "Add [NIMI] v1.0: [KUVAUS]"` |
| **Päivitys** | `Commitoi ja pushaa: [POLKU]` <br> `Message: "Update [NIMI] to v[X]: [MUUTOS]"` |
| **Korjaus** | `Commitoi ja pushaa: [POLKU]` <br> `Message: "Fix [VIRHE] in [NIMI]"` |
| **Tarkista status** | `Näytä git status ja git diff` |
| **Peruuta commit** | `Näytä viimeisin commit, sitten revertoi` |
| **Usea tiedosto** | `Commitoi kaikki muuttuneet tiedostot` <br> `Message: "[KUVAUS]"` |

---

## ✅ Työnkulun tarkistussumma

### Session alussa (AINA):

- [ ] Ajettu avausprompt (git status, git log, git fetch)
- [ ] Ei uncommitted muutoksia edelliseltä sessiolta
- [ ] Paikallinen repo on ajan tasalla GitHubin kanssa

### Ennen committia (AINA):

- [ ] Tiedosto tallennettu VS Codessa (`Ctrl+S`)
- [ ] Tiedosto oikeassa paikassa (`docs/specs/` tai juuri)
- [ ] Dokumentissa oikea versionumero
- [ ] Commit message valmis
- [ ] Prompt kopioitu Claude Codelle

### Commitin jälkeen (AINA):

- [ ] Claude Code vahvisti onnistumisen
- [ ] Tarkista GitHub-historiasta että commit näkyy
- [ ] (Valinnainen) Claude.ai projektissa: Sync GitHub

---

## 🎯 Session lopetus

### Kun session päättyy

1. **Varmista että kaikki commitoitu:**
   ```
   Claude Code:lle: "Näytä git status - onko jotain uncommitted?"
   ```

2. **Tarkista GitHub:**
   - Avaa repo selaimessa
   - Katso että viimeisimmät commitit näkyvät

3. **Deaktivoi GitHub-dokumentit Claude.ai projektissa** (token-säästö)

---

## 🔗 Linkit

- [Claude Code Docs](https://docs.anthropic.com/en/docs/claude-code)
- [Git Basics](https://git-scm.com/book/en/v2/Getting-Started-Git-Basics)
- [GitHub Guides](https://guides.github.com/)

---

## Muutoshistoria

| Versio | Päivämäärä | Muutokset |
|--------|------------|-----------|
| 2.2 | 2025-11-25 | Korjattu placeholderit (jussivares), lisätty session aloitustarkistus, korjattu UTF-8 enkoodaus |
| 2.1 | 2025-11-25 | Claude.ai kirjoittaa valmiit promptit, projektikohtaiset tiedot |
| 2.0 | 2025-11-25 | Hybridimalli, valmiit promptit, turvallisuustarkistukset |
| 1.0 | 2025-11-25 | Ensimmäinen versio |

---

*Tämä ohje on osa Claude API -suunnittelutyökalun dokumentaatiota.*