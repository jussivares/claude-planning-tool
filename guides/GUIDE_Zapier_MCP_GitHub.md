# Zapier MCP GitHub Integration Guide

> **Versio:** 1.0  
> **Luotu:** 2025-12-02  
> **Status:** ✅ Toimiva

## Yleiskuvaus

Zapier MCP mahdollistaa tiedostojen kirjoittamisen suoraan claude.ai:sta GitHubiin ilman manuaalista lataus/kopiointiprosessia.

```
claude.ai → Zapier MCP → GitHub API → Repository
                                          ↓
                                     git pull
                                          ↓
                                   Lokaali kopio
```

## Edellytykset

1. **Zapier-tili** (ilmainen riittää, 300 kutsua/kk)
2. **Claude Pro/Max** (Integrations-ominaisuus)
3. **GitHub-tili** yhdistettynä Zapieriin

## Käyttöönotto

### 1. Luo MCP Server

1. Mene: https://mcp.zapier.com
2. Klikkaa: "+ New MCP Server"
3. Valitse: "Claude" pudotusvalikosta
4. Nimeä palvelin (esim. "GitHub-Commit")
5. Klikkaa: "Create MCP Server"

### 2. Lisää GitHub-toiminto

1. Configure-välilehti
2. Klikkaa: "+ Add tool"
3. Etsi: "GitHub"
4. Valitse: "Create or Update File in Repository"
5. Yhdistä GitHub-tilisi (OAuth)

### 3. ⚠️ KRIITTINEN: Repository-kentän konfigurointi

**Tunnettu ongelma:** "Have AI select a value" EI TOIMI Repository-kentässä.

**Ratkaisu:**
```
Repository-kenttä:
├── ❌ Have AI select/generate a value  ← ÄLÄ KÄYTÄ
├── ✅ Set a specific value             ← KÄYTÄ TÄTÄ
│   └── Valitse: [owner/repo-name]
└── Muut kentät: "Have AI generate" OK
```

### 4. Yhdistä claude.ai:hin

1. Claude.ai → Settings → Integrations
2. "+ Add integration"
3. Liitä Zapierin Connect-välilehden URL
4. Hyväksy yhteys

## Käyttö claude.ai:ssa

### Peruskomento
```
"Luo tiedosto docs/example.md sisällöllä [X] ja commit-viestillä [Y]"
```

### Esimerkki
```
"Vie tämä SPEC GitHubiin polkuun docs/specs/SPEC_Example.md 
commit-viestillä 'docs: Add Example SPEC v1.0'"
```

### Tiedoston päivitys
```
"Päivitä tiedosto docs/INDEX.md uudella sisällöllä [X]"
```

**Huom:** Päivitys vaatii tiedoston SHA:n. Zapier hakee sen automaattisesti.

## Rajoitukset

| Rajoitus | Arvo |
|----------|------|
| Kutsuja/tunti | 80 |
| Kutsuja/päivä | 160 |
| Kutsuja/kuukausi | 300 (ilmainen) |
| Taskeja per kutsu | 2 |

## Vianmääritys

### "Required field 'repo' is missing"

**Syy:** Repository-kenttä on "Have AI select" -tilassa.

**Ratkaisu:**
1. mcp.zapier.com → Configure
2. GitHub-toiminto → Repository
3. Vaihda: "Set a specific value"
4. Valitse repo listasta
5. Save

### "Resolved fields: (tyhjä)"

**Syy:** Zapier ei osaa mappata parametreja.

**Ratkaisu:** Sama kuin yllä - kiinnitä repo.

## Työnkulku projektissa

```
1. Claude.ai: Kirjoita/muokkaa dokumentti
2. Claude.ai: "Vie GitHubiin polkuun X commit-viestillä Y"
3. Zapier: Tekee commitin automaattisesti
4. Lokaali: git pull (manuaalinen tai automatisoitu)
5. Claude Desktop: Voi lukea päivitetyn tiedoston
```

## TODO: Tutkittavat asiat

- [ ] **Multi-repo tuki:** Voiko samaa MCP-palvelinta käyttää useille repoille?
- [ ] **Repo-bugi:** Miksi "Have AI select" ei toimi? Onko virallinen bugi?
- [ ] **Branch-tuki:** Toimiiko muille brancheille kuin main?

---

*Dokumentti luotu Zapier MCP -integraatiolla 2025-12-02*
