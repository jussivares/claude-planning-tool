# Zapier MCP GitHub Integration Guide

> **Versio:** 1.0  
> **Luotu:** 2025-12-02  
> **Status:** ✅ Toimiva

## Yleiskatsaus

Zapier MCP mahdollistaa tiedostojen kirjoittamisen suoraan claude.ai:sta GitHubiin ilman manuaalista lataus/kopiointiprosessia.

```
claude.ai → Zapier MCP → GitHub API → Repository
                                          ↓
                                     git pull
                                          ↓
                                   Lokaali kopio (C:\)
```

## Edellytykset

1. **Claude Pro/Max/Team/Enterprise** - Integrations-ominaisuus
2. **Zapier-tili** (ilmainen riittää, 300 kutsua/kk)
3. **GitHub-tili** ja repository

## Käyttöönotto

### 1. Luo Zapier MCP Server

1. Mene: https://mcp.zapier.com
2. Klikkaa: "+ New MCP Server"
3. Valitse: "Claude" (client)
4. Anna nimi, esim. "GitHub-Commit"
5. Klikkaa: "Create MCP Server"

### 2. Lisää GitHub-toiminto

1. Configure-välilehti
2. "+ Add tool"
3. Etsi: "GitHub"
4. Valitse: "Create or Update File in Repository"
5. Yhdistä GitHub-tilisi (OAuth)

### 3. KRIITTINEN: Repository-konfiguraatio

**HUOM:** "Have AI select a value" EI TOIMI Repository-kentässä!

```
Repository-kenttä:
  ○ Have AI select/generate a value  ← EI TOIMI
  ● Set a specific value             ← KÄYTÄ TÄTÄ
  
  Pudotusvalikko:
  → Valitse: owner/repository-name
```

**Muut kentät** (path, content, message) voivat olla "Have AI generate".

### 4. Yhdistä claude.ai:hin

1. Claude.ai → Settings → Integrations
2. "+ Add integration"
3. Liitä Zapierin antama URL (Connect-välilehdeltä)
4. Hyväksy yhteys

## Käyttö claude.ai:ssa

### Uuden tiedoston luonti

```
"Luo tiedosto docs/SPEC_Example.md GitHubiin seuraavalla sisällöllä:
[sisältö tähän]"
```

### Tiedoston päivitys

Päivitys vaatii SHA-arvon (tiedoston nykyinen versio):

```
"Päivitä tiedosto docs/README.md, SHA: [sha-arvo]"
```

## Rajoitukset

| Rajoitus | Arvo |
|----------|------|
| Kutsuja/tunti | 80 |
| Kutsuja/päivä | 160 |
| Kutsuja/kuukausi | 300 (ilmainen) |
| Taskeja per kutsu | 2 |

## Tunnetut ongelmat

### ISSUE-001: Repository-kenttä ei hyväksy AI-valintaa

- **Status:** Tutkitaan  
- **Kuvaus:** "Have AI select a value" aiheuttaa virheen  
- **Kiertotapa:** Käytä "Set a specific value"  

### ISSUE-002: SHA vaaditaan päivityksiin

- **Status:** Normaali käyttäytyminen  
- **Kuvaus:** Olemassa olevan tiedoston päivitys vaatii SHA:n  
- **Ratkaisu:** Hae SHA ensin tai käytä uniikkia tiedostonimeä  

## Työnkulku

```
1. CLAUDE.AI → Kirjoita dokumentti → Vie GitHubiin
2. ZAPIER MCP → GitHub API -kutsu → Commit
3. GITHUB → Tiedosto repossa
4. LOKAALI → git pull → C:\projekti\
```

---

*Dokumentti luotu Zapier MCP:llä claude.ai:sta 2025-12-02*
