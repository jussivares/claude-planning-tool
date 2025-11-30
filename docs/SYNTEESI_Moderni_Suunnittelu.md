# SYNTEESI: Moderni ohjelmistosuunnittelu AI-agenttipohjaiseen jÃƒÂ¤rjestelmÃƒÂ¤ÃƒÂ¤n

> **Versio:** 1.2  
> **PÃƒÂ¤ivitetty:** 2025-11-23  
> **Tiedosto:** v1.2_SYNTEESI_Moderni_Suunnittelu.md  
> **Tarkoitus:** YhdistÃƒÂ¤ÃƒÂ¤ modernit kÃƒÂ¤ytÃƒÂ¤nnÃƒÂ¶t (API-first, AI agents, systems architecture) AI-taloushallintojÃƒÂ¤rjestelmÃƒÂ¤n suunnitteluun  
> **Peer Review:** Google Gemini (AI Software Architect)  
> - v1.1: Enterprise-grade -tarkennukset  
> - v1.2: KÃƒÂ¤ytÃƒÂ¤nnÃƒÂ¶n linjaukset ja workflow-tarkennukset

---

## Johdanto

Tutkimme kysymystÃƒÂ¤: **MitÃƒÂ¤ tarvitaan SPEC:in ja koodauksen vÃƒÂ¤lissÃƒÂ¤?**

Analysoimme kolme nÃƒÂ¤kÃƒÂ¶kulmaa:
1. **API-First Design** (2024-2025 trendit)
2. **AI Agent Architecture** (agentic systems)
3. **Systems Architecture** (Eskil Steenberg, black box -periaatteet)

TÃƒÂ¤mÃƒÂ¤ dokumentti syntetisoi nÃƒÂ¤mÃƒÂ¤ nÃƒÂ¤kÃƒÂ¶kulmat **kÃƒÂ¤ytÃƒÂ¤nnÃƒÂ¶n malliksi** meidÃƒÂ¤n projektiin.

---

## Ã¢Å¡Â Ã¯Â¸Â KRIITTISET ENTERPRISE-LISÃƒâ€žYKSET (Peer Review)

**LÃƒÂ¤hde:** Google Gemini (AI Software Architect, 2025-11-23)

AlkuperÃƒÂ¤inen synteesi (v1.0) oli **erinomainen pohja**, mutta tarvitsee seuraavat lisÃƒÂ¤ykset tullakseen **tuotantovalmiiksi** (Enterprise-grade):

### 1. LLM-Optimized OpenAPI (Semanttinen kerros) Ã°Å¸Å½Â¯

**Ongelma:** Perinteinen OpenAPI on kirjoitettu **ihmiselle**. AI tarvitsee **semanttisen kontekstin**.

**Ratkaisu:** "Verbose Descriptions" - selitÃƒÂ¤ AI:lle MILLOIN ja MIKSI endpoint kutsutaan.

```yaml
# Ã¢ÂÅ’ HUONO (ihmiselle)
description: "Creates a purchase invoice"

# Ã¢Å“â€¦ HYVÃƒâ€ž (AI:lle)
description: |
  Use this endpoint ONLY when:
  1. Vendor has been validated (search_vendor returned match)
  2. Duplicate check passed (check_duplicate_invoice = false)
  3. All invoice lines have valid account_id and vat_code
  
  Ã¢Å¡Â Ã¯Â¸Â WARNING: This action commits FINANCIAL DATA to database.
  If ai_confidence < 0.85, you MUST set needs_human_review = true.

# LisÃƒÂ¤ksi: Vendor extension merkitsemÃƒÂ¤ÃƒÂ¤n vaarallisia operaatioita
x-openai-isConsequential: true  # POST/PUT/DELETE taloushallinnossa
```

**HyÃƒÂ¶ty:** AI ymmÃƒÂ¤rtÃƒÂ¤ÃƒÂ¤ **kontekstin** ja toimii varovaisemmin.

---

### 2. Guardrails & Validation Layer (Deterministisyys) Ã°Å¸â€ºÂ¡Ã¯Â¸Â

**Ongelma:** AI:n luovuus on **riski** taloushallinnossa. Validointi ei riitÃƒÂ¤ - tarvitaan **Guardrails**.

**Ratkaisu:** LisÃƒÂ¤ÃƒÂ¤ TECH_SPEC:iin osio **"Output Guardrails"** - deterministiset sÃƒÂ¤ÃƒÂ¤nnÃƒÂ¶t jotka ajetaan **ennen** kuin AI:n output hyvÃƒÂ¤ksytÃƒÂ¤ÃƒÂ¤n.

```python
# Esimerkki: Guardrails ostolaskulle

class InvoiceGuardrails:
    """Deterministiset sÃƒÂ¤ÃƒÂ¤nnÃƒÂ¶t ennen API-kutsua"""
    
    @staticmethod
    def validate_before_commit(invoice_data: dict) -> tuple[bool, str]:
        """Tarkista ENNEN kuin AI commitoi datan"""
        
        # Rule 1: Loppusumma tÃƒÂ¤smÃƒÂ¤ÃƒÂ¤ riveihin
        lines_total = sum(line['amount'] for line in invoice_data['lines'])
        invoice_total = invoice_data['total_amount']
        
        if abs(lines_total - invoice_total) > 0.01:
            return False, f"Total mismatch: {invoice_total} != {lines_total}"
        
        # Rule 2: PÃƒÂ¤ivÃƒÂ¤mÃƒÂ¤ÃƒÂ¤rÃƒÂ¤t loogisia
        if invoice_data['due_date'] < invoice_data['invoice_date']:
            return False, "Due date cannot be before invoice date"
        
        # Rule 3: Vendor exists
        if not invoice_data.get('vendor_id'):
            return False, "Vendor ID missing"
        
        # Rule 4: ALV-koodit valideja
        valid_vat_codes = ['KOMY24', 'KOMY14', 'KOMY10', 'KOOS']
        for line in invoice_data['lines']:
            if line.get('vat_code') not in valid_vat_codes:
                return False, f"Invalid VAT code: {line['vat_code']}"
        
        return True, "OK"
```

**MissÃƒÂ¤:** Ajetaan ENNEN kuin AI:n ehdotus menee APIin tai ihmiselle.

**HyÃƒÂ¶ty:** EstÃƒÂ¤ÃƒÂ¤ AI:n tekemÃƒÂ¤stÃƒÂ¤ loogisesti virheellisiÃƒÂ¤ pÃƒÂ¤ÃƒÂ¤tÃƒÂ¶ksiÃƒÂ¤.

---

### 3. Observability & Evals (AI-testaus) Ã°Å¸â€œÅ 

**Ongelma:** "Miten testaamme AI:n pÃƒÂ¤ÃƒÂ¤tÃƒÂ¶ksentekoa?" - Dokumentissa kysyttiin, mutta ei vastattu.

**Ratkaisu:** **Golden Dataset** + CI/CD-testaus.

#### 3.1 Golden Dataset
```markdown
# TECH_SPEC_02: Ostolaskut
## 8. Golden Dataset (AI Testing)

MÃƒÂ¤ÃƒÂ¤ritellÃƒÂ¤ÃƒÂ¤n 20 esimerkkitapausta:

| ID | Input (PDF) | Expected Output | Notes |
|----|-------------|-----------------|-------|
| 01 | invoice_normal.pdf | vendor_id=123, confidence>0.9, auto-approve | Peruscase |
| 02 | invoice_new_vendor.pdf | create_vendor=true, needs_review=true | Uusi toimittaja |
| 03 | invoice_duplicate.pdf | duplicate_detected=true, ask_human | Duplikaatti |
| 04 | invoice_bad_ocr.pdf | confidence<0.7, ask_human | Huono OCR |
| ... | ... | ... | ... |

SÃƒÂ¤ilytÃƒÂ¤: /tests/golden_datasets/purchase_invoices/
```

#### 3.2 CI/CD-testaus
```python
# tests/test_ai_agent_purchase_invoices.py

def test_golden_dataset():
    """Unit test for AI intelligence"""
    
    dataset = load_golden_dataset('purchase_invoices')
    agent = PurchaseInvoiceAgent()
    
    results = []
    for case in dataset:
        prediction = agent.process(case['input'])
        expected = case['expected_output']
        
        match = compare_outputs(prediction, expected)
        results.append({
            'case_id': case['id'],
            'match': match,
            'confidence': prediction.confidence
        })
    
    accuracy = sum(r['match'] for r in results) / len(results)
    
    # Regression check: jos tarkkuus laskee, deployment estetÃƒÂ¤ÃƒÂ¤n
    assert accuracy >= 0.85, f"AI accuracy dropped: {accuracy}"
```

#### 3.3 Chain of Thought -lokitus
```python
# Vaatimus: Jokainen AI:n pÃƒÂ¤ÃƒÂ¤tÃƒÂ¶s logitetaan

logger.info({
    'event': 'ai_decision',
    'agent': 'purchase_invoice',
    'invoice_id': invoice.id,
    'chain_of_thought': [
        '1. OCR extracted vendor name: Verkkokauppa.com',
        '2. search_vendor("Verkkokauppa.com") Ã¢â€ â€™ found: id=456',
        '3. check_duplicate(456, "INV-2024-001") Ã¢â€ â€™ false',
        '4. suggest_account("Tietokoneet") Ã¢â€ â€™ 4000 (confidence 0.92)',
        '5. Decision: auto-approve (confidence > 0.90)'
    ],
    'final_decision': 'approved',
    'confidence': 0.92
})
```

**TyÃƒÂ¶kalu:** LangSmith, Weights & Biases, tai custom tracing.

---

### 4. Asynkronisuus ja Human-in-the-Loop Ã°Å¸â€â€ž

**Ongelma:** AI-prosessointi voi kestÃƒÂ¤ÃƒÂ¤ (OCR + pÃƒÂ¤ÃƒÂ¤ttely). Synkroninen HTTP aikakatkaistaa.

**Ratkaisu:** **Webhook tai Polling** -malli raskaissa operaatioissa.

```
Nykyinen (synkroninen):
Client Ã¢â€ â€™ POST /api/v1/purchase-invoices Ã¢â€ â€™ 201 Created (tai 400 Bad Request)
         [AI prosessoi 5-30 sekuntia]
         Ã¢ÂÅ’ Riski: Timeout!

Parannettu (asynkroninen):
Client Ã¢â€ â€™ POST /api/v1/purchase-invoices Ã¢â€ â€™ 202 Accepted + job_id
         [AI prosessoi taustalla]
         
Client Ã¢â€ â€™ GET /api/v1/jobs/{job_id} Ã¢â€ â€™ {status: "processing"}
         [Pollaa 2 sekunnin vÃƒÂ¤lein]
         
Client Ã¢â€ â€™ GET /api/v1/jobs/{job_id} Ã¢â€ â€™ {status: "needs_review", question_id: ...}
         [Ihminen vastaa kysymykseen]
         
Client Ã¢â€ â€™ POST /api/v1/jobs/{job_id}/answers Ã¢â€ â€™ OK
         
Client Ã¢â€ â€™ GET /api/v1/jobs/{job_id} Ã¢â€ â€™ {status: "completed", invoice_id: ...}
```

**Vaihtoehto:** Webhook
```yaml
# Client rekisterÃƒÂ¶i webhook:n
POST /api/v1/webhooks
{
  "url": "https://client.com/invoice-updates",
  "events": ["invoice.needs_review", "invoice.completed"]
}

# AI lÃƒÂ¤hettÃƒÂ¤ÃƒÂ¤ notification kun valmis
POST https://client.com/invoice-updates
{
  "event": "invoice.needs_review",
  "job_id": "abc-123",
  "question": {...}
}
```

---

### 5. System Prompt Architecture Ã°Å¸Â§Â 

**Ongelma:** AI ei toimi tyhjiÃƒÂ¶ssÃƒÂ¤. TECH_SPEC ei mÃƒÂ¤ÃƒÂ¤rittele **agentin personaa** ja **kontekstia**.

**Ratkaisu:** LisÃƒÂ¤ÃƒÂ¤ TECH_SPEC:iin osio **"System Prompt Architecture"**.

```markdown
# TECH_SPEC_02: Ostolaskut
## 2.5 System Prompt Architecture

### Persona
Olet tarkka ja varovainen kirjanpitoavustaja. TehtÃƒÂ¤vÃƒÂ¤si on analysoida 
ostolaskuja ja ehdottaa kirjauksia. Sinun TÃƒâ€žYTYY:
- Olla konservatiivinen: jos epÃƒÂ¤varma, kysy ihmiseltÃƒÂ¤
- Noudattaa kirjanpitolakia: jokainen kirjaus on jÃƒÂ¤ljitettÃƒÂ¤vÃƒÂ¤
- Suosia turvallisempia vaihtoehtoja (esim. ALV-kannassa)

### Context Window Strategy
Jokaiseen promptiin sisÃƒÂ¤llytetÃƒÂ¤ÃƒÂ¤n:
1. **Tilikartta** (accounts): kaikki tilit + ai_keywords
2. **Toimittajan historia**: viimeisimmÃƒÂ¤t 5 laskua tÃƒÂ¤ltÃƒÂ¤ toimittajalta
   - MitÃƒÂ¤ tilejÃƒÂ¤ kÃƒÂ¤ytetty aiemmin?
   - KeskimÃƒÂ¤ÃƒÂ¤rÃƒÂ¤inen laskusumma?
   - Tyypilliset tuotekuvaukset?
3. **Accounting rules**: Kaikki sÃƒÂ¤ÃƒÂ¤nnÃƒÂ¶t jotka koskevat tÃƒÂ¤tÃƒÂ¤ toimittajaa
4. **Current invoice data**: OCR-tulos, metadata

### Few-Shot Examples
SisÃƒÂ¤llytÃƒÂ¤ 2-3 esimerkkiÃƒÂ¤ onnistuneista kirjauksista:
---
Esimerkki 1:
Input: "Verkkokauppa.com, LÃƒÂ¤ppÃƒÂ¤ri 1200Ã¢â€šÂ¬, ALV 24%"
Output: {account: 4000, vat: KOMY24, confidence: 0.95}
Reasoning: "Tietokoneet Ã¢â€ â€™ tili 4000, tavallinen ALV 24%"
---
```

**HyÃƒÂ¶ty:** AI ymmÃƒÂ¤rtÃƒÂ¤ÃƒÂ¤ roolinsa ja tekee johdonmukaisempia pÃƒÂ¤ÃƒÂ¤tÃƒÂ¶ksiÃƒÂ¤.

---

### 6. Self-Correction Loop (Virheiden korjaus) Ã°Å¸â€Â§

**Ongelma:** Jos API palauttaa virheen (400 Bad Request), AI ei saa kaatua.

**Ratkaisu:** **Self-Correction Loop** - AI yrittÃƒÂ¤ÃƒÂ¤ korjata virheen automaattisesti.

```python
# Esimerkki: AI yrittÃƒÂ¤ÃƒÂ¤ korjata "Vendor not found" -virheen

class SelfCorrectionLoop:
    def __init__(self, max_retries=3):
        self.max_retries = max_retries
    
    def create_invoice_with_retry(self, invoice_data: dict):
        """AI yrittÃƒÂ¤ÃƒÂ¤ korjata virheet automaattisesti"""
        
        for attempt in range(self.max_retries):
            try:
                # YritÃƒÂ¤ luoda lasku
                response = api.post('/purchase-invoices', invoice_data)
                return response  # Onnistui!
                
            except VendorNotFoundError as e:
                # AI:n ajatus: "Vendor ID ei lÃƒÂ¶ydy. Kokeilen etsiÃƒÂ¤ uudestaan."
                
                if attempt < self.max_retries - 1:
                    # Korjausyritys 1: Etsi vendor Y-tunnuksella
                    if 'business_id' in invoice_data:
                        vendor = search_vendor_by_business_id(
                            invoice_data['business_id']
                        )
                        if vendor:
                            invoice_data['vendor_id'] = vendor.id
                            continue  # YritÃƒÂ¤ uudestaan
                    
                    # Korjausyritys 2: Luo uusi vendor
                    vendor = create_vendor_from_invoice(invoice_data)
                    invoice_data['vendor_id'] = vendor.id
                    continue
                
                else:
                    # Max retries saavutettu - kysy ihmiseltÃƒÂ¤
                    return ask_human({
                        'question_type': 'vendor_not_found',
                        'context': invoice_data,
                        'error': str(e)
                    })
            
            except ValidationError as e:
                # Validointivirhe - korjaa mitÃƒÂ¤ voi
                if 'due_date' in e.fields:
                    # Korjaa: Jos due_date puuttuu, laske 14 pv
                    invoice_data['due_date'] = (
                        invoice_data['invoice_date'] + timedelta(days=14)
                    )
                    continue
                
                # Jos ei voi korjata, kysy ihmiseltÃƒÂ¤
                return ask_human({
                    'question_type': 'validation_error',
                    'error': e
                })
```

**HyÃƒÂ¶ty:** AI ei jÃƒÂ¤ÃƒÂ¤ jumiin pienistÃƒÂ¤ virheistÃƒÂ¤, vaan yrittÃƒÂ¤ÃƒÂ¤ ratkaista ongelmat itsenÃƒÂ¤isesti.

---

### 7. TyÃƒÂ¶kalusuositus: TypeSpec yli OpenAPI YAML Ã°Å¸â€ºÂ Ã¯Â¸Â

**Perustelu:**
- Ã¢Å“â€¦ **Modulaarisempi** - komponentit uudelleenkÃƒÂ¤ytettÃƒÂ¤viÃƒÂ¤
- Ã¢Å“â€¦ **Tiiviimpi** - vÃƒÂ¤hemmÃƒÂ¤n boilerplate-koodia
- Ã¢Å“â€¦ **Code-first tyyli** - tuttu kehittÃƒÂ¤jille
- Ã¢Å“â€¦ **Generoi** sekÃƒÂ¤ OpenAPI YAML:in (AI:lle) ettÃƒÂ¤ JSON Scheman (koodille)

**Esimerkki: TypeSpec vs. YAML**

```typescript
// TypeSpec (tiivis, luettava)

import "@typespec/http";
import "@typespec/openapi";

namespace AIAccounting.PurchaseInvoices;

@doc("AI-optimized: Use ONLY after vendor validation and duplicate check")
@tag("AI-Consequential")  // Merkki vaarallisesta operaatiosta
op createPurchaseInvoice(
  @doc("Strict business ID from invoice document. DO NOT hallucinate.")
  @body invoice: InvoiceCreationRequest
): {
  @statusCode statusCode: 202;  // Asynkroninen!
  @body job: JobResponse;
} | {
  @statusCode statusCode: 400;
  @body error: ValidationError;
};

model InvoiceCreationRequest {
  @doc("Vendor UUID. Must exist in database. Use search_vendor first.")
  vendor_id: string;
  
  @doc("Total including VAT. Must match sum of lines (tolerance 0.01Ã¢â€šÂ¬)")
  total_amount: decimal;
  
  @doc("AI confidence [0-1]. If <0.85, MUST set needs_human_review=true")
  ai_confidence: float;
  
  @doc("Force human review. Set true if OCR quality poor or logic fuzzy.")
  needs_human_review: boolean;
  
  lines: InvoiceLine[];
}
```

**vs. OpenAPI YAML (pitkÃƒÂ¤, monisanainen):**
```yaml
# 3x pidempi, monimutkaisempi...
```

**PÃƒÂ¤ÃƒÂ¤tÃƒÂ¶s:** KÃƒÂ¤ytÃƒÂ¤ **TypeSpec** teknisessÃƒÂ¤ mÃƒÂ¤ÃƒÂ¤rittelyssÃƒÂ¤. Generoi siitÃƒÂ¤ OpenAPI YAML AI:ta varten.

---

## Ã°Å¸Å½Â¯ Vastaus kysymykseen: MitÃƒÂ¤ puuttuu?

### Nykyinen tilanne
```
SPEC_XX.md                           main.py, api.py, ...
(Toiminnallinen)                     (Toteutus)
     Ã¢â€â€š                                    Ã¢â€â€š
     Ã¢â€â€š         Ã¢ÂÅ’ PUUTTUU                 Ã¢â€â€š
     Ã¢â€â€š                                    Ã¢â€â€š
     Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ
```

### MitÃƒÂ¤ puuttuu (kolme tasoa)

#### 1. API-tason mÃƒÂ¤ÃƒÂ¤rittely
- **OpenAPI/Swagger-speksit** (REST endpoints)
- **PyyntÃƒÂ¶/vastaus-skemat** (JSON structures)
- **ValidointisÃƒÂ¤ÃƒÂ¤nnÃƒÂ¶t** (mitÃƒÂ¤ data pitÃƒÂ¤ÃƒÂ¤ sisÃƒÂ¤ltÃƒÂ¤ÃƒÂ¤)
- **Error handling** (mitÃƒÂ¤ tapahtuu kun menee pieleen)

#### 2. AI-agentin logiikka
- **Decision trees** (miten AI pÃƒÂ¤ÃƒÂ¤ttÃƒÂ¤ÃƒÂ¤)
- **Tool definitions** (mitÃƒÂ¤ tyÃƒÂ¶kaluja AI kÃƒÂ¤yttÃƒÂ¤ÃƒÂ¤)
- **Confidence thresholds** (milloin kysyy ihmiseltÃƒÂ¤)
- **Learning patterns** (miten AI oppii)

#### 3. Moduulirajapinnat
- **Black box contracts** (mitÃƒÂ¤ moduuli ottaa sisÃƒÂ¤ÃƒÂ¤n, antaa ulos)
- **Integration patterns** (miten moduulit kommunikoivat)
- **Wrapper interfaces** (miten ulkoiset palvelut integroidaan)

---

## Ã°Å¸â€œÂ Moderni malli: Kolme kerrosta

```
Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â
Ã¢â€â€š                    MODERNI SUUNNITTELUMALLI                     Ã¢â€â€š
Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â¤
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  Kerros 1: SPEC (Toiminnallinen mÃƒÂ¤ÃƒÂ¤rittely)                    Ã¢â€â€š
Ã¢â€â€š  Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ MitÃƒÂ¤ jÃƒÂ¤rjestelmÃƒÂ¤ tekee                                   Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ AI:n rooli (yleisellÃƒÂ¤ tasolla)                          Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Liiketoimintaprosessit                                  Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ KÃƒÂ¤yttÃƒÂ¤jÃƒÂ¤profiilit ja -tarinat                           Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Tietorakenteet (taulut, kentÃƒÂ¤t)                         Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ Ã¢â€â€š
Ã¢â€â€š                           Ã¢â€â€š                                     Ã¢â€â€š
Ã¢â€â€š                           Ã¢â€“Â¼                                     Ã¢â€â€š
Ã¢â€â€š  Kerros 2: API & AGENT SPEC (Tekninen mÃƒÂ¤ÃƒÂ¤rittely) Ã¢Â¬â€¦Ã¯Â¸Â UUSI     Ã¢â€â€š
Ã¢â€â€š  Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š A. API-rajapinnat                                          Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š    Ã¢â‚¬Â¢ OpenAPI-speksit (REST endpoints)                      Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š    Ã¢â‚¬Â¢ Request/Response schemas                              Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š    Ã¢â‚¬Â¢ Validoinnit ja error handling                         Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š                                                            Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š B. AI-agentin mÃƒÂ¤ÃƒÂ¤rittely                                   Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š    Ã¢â‚¬Â¢ Tool definitions (mitÃƒÂ¤ AI voi kutsua)                 Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š    Ã¢â‚¬Â¢ Decision logic (pseudokoodi/flowchart)                Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š    Ã¢â‚¬Â¢ Confidence thresholds                                 Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š    Ã¢â‚¬Â¢ Learning patterns                                     Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š                                                            Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š C. Moduulirajapinnat                                       Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š    Ã¢â‚¬Â¢ Black box contracts                                   Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š    Ã¢â‚¬Â¢ Integration patterns                                  Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š    Ã¢â‚¬Â¢ Wrapper interfaces (pankit, Finvoice)                 Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ Ã¢â€â€š
Ã¢â€â€š                           Ã¢â€â€š                                     Ã¢â€â€š
Ã¢â€â€š                           Ã¢â€“Â¼                                     Ã¢â€â€š
Ã¢â€â€š  Kerros 3: KOODI (Toteutus)                                    Ã¢â€â€š
Ã¢â€â€š  Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Varsinainen ohjelmakoodi                                 Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Unit testit                                              Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Integration testit                                       Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Deployment                                               Ã¢â€â€š Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ
```

---

## Ã°Å¸â€Â§ Kerros 2 tarkemmin: API & AGENT SPEC

### A. API-rajapintojen mÃƒÂ¤ÃƒÂ¤rittely

**TyÃƒÂ¶kalu: OpenAPI 3.1 / TypeSpec**

Moderneissa projekteissa (2024-2025):
- Ã¢Å“â€¦ **API-first**: API spesifioidaan ENNEN koodausta
- Ã¢Å“â€¦ **Contract-driven**: OpenAPI on "sopimus" frontend/backend vÃƒÂ¤lillÃƒÂ¤
- Ã¢Å“â€¦ **Parallel development**: Frontend ja backend voivat edetÃƒÂ¤ samanaikaisesti
- Ã¢Å“â€¦ **Auto-generation**: Koodi generoidaan speksistÃƒÂ¤ (tai validoidaan vasten sitÃƒÂ¤)

**Esimerkki: Ostolaskun luonti**

```yaml
# openapi_spec.yaml

/api/v1/purchase-invoices:
  post:
    summary: Luo uusi ostolasku (AI-toiminto)
    description: |
      AI-agentti kutsuu tÃƒÂ¤tÃƒÂ¤ kun se on tunnistanut ostolaskun
      ja haluaa luoda sen jÃƒÂ¤rjestelmÃƒÂ¤ÃƒÂ¤n.
    requestBody:
      required: true
      content:
        application/json:
          schema:
            type: object
            required: [vendor_id, invoice_number, invoice_date, total_amount]
            properties:
              vendor_id:
                type: string
                format: uuid
                description: Toimittajan tunniste
              invoice_number:
                type: string
                maxLength: 50
                description: Laskun numero
              invoice_date:
                type: string
                format: date
                description: Laskun pÃƒÂ¤ivÃƒÂ¤mÃƒÂ¤ÃƒÂ¤rÃƒÂ¤
              due_date:
                type: string
                format: date
              total_amount:
                type: number
                format: decimal
                minimum: 0.01
              lines:
                type: array
                items:
                  type: object
                  properties:
                    description: string
                    amount: number
                    account_id: uuid
                    vat_code: string
              ai_confidence:
                type: number
                format: float
                minimum: 0
                maximum: 1
                description: AI:n luottamustaso (0-1)
    responses:
      201:
        description: Ostolasku luotu onnistuneesti
        content:
          application/json:
            schema:
              type: object
              properties:
                invoice_id: 
                  type: string
                  format: uuid
                status:
                  type: string
                  enum: [draft, pending_review, approved]
                needs_human_review:
                  type: boolean
                  description: PitÃƒÂ¤ÃƒÂ¤kÃƒÂ¶ ihmisen tarkistaa
      400:
        description: Validointivirhe
        content:
          application/json:
            schema:
              type: object
              properties:
                error_code: string
                error_message: string
                validation_errors: array
```

**HyÃƒÂ¶dyt:**
- Frontend-tiimi voi mock-data avulla kehittÃƒÂ¤ÃƒÂ¤ UI:ta ennen backend-toteutusta
- Backend-tiimi tietÃƒÂ¤ÃƒÂ¤ tarkalleen mitÃƒÂ¤ pitÃƒÂ¤ÃƒÂ¤ toteuttaa
- Testaus on helpompaa (contract testing)
- Dokumentaatio syntyy automaattisesti

---

### B. AI-agentin mÃƒÂ¤ÃƒÂ¤rittely

**Perustuu Anthropic:in "Building Effective Agents" -malliin:**

```
Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â
Ã¢â€â€š                    AI-AGENTTI: Ostolaskujen kÃƒÂ¤sittely           Ã¢â€â€š
Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â¤
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  1. TOOLS (MitÃƒÂ¤ AI voi tehdÃƒÂ¤)                                   Ã¢â€â€š
Ã¢â€â€š     Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Ã¢â‚¬Â¢ search_vendor(business_id) Ã¢â€ â€™ Vendor | null           Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Ã¢â‚¬Â¢ create_vendor(data) Ã¢â€ â€™ Vendor                         Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Ã¢â‚¬Â¢ check_duplicate_invoice(vendor_id, invoice_number)   Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š   Ã¢â€ â€™ boolean                                            Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Ã¢â‚¬Â¢ suggest_account(description, vendor) Ã¢â€ â€™ Account       Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Ã¢â‚¬Â¢ create_purchase_invoice(data) Ã¢â€ â€™ Invoice              Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Ã¢â‚¬Â¢ ask_human(question_type, context) Ã¢â€ â€™ Answer           Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ  Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  2. DECISION LOGIC (PÃƒÂ¤ÃƒÂ¤tÃƒÂ¶ksenteko)                              Ã¢â€â€š
Ã¢â€â€š     Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Flowchart / Pseudokoodi:                               Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š                                                         Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š 1. OCR-analyysi Ã¢â€ â€™ extrahoi: Y-tunnus, summa, pÃƒÂ¤ivÃƒÂ¤...  Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š 2. Etsi toimittaja (search_vendor)                     Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š    IF NOT FOUND:                                       Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š       IF confidence > 0.7:                             Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š          ask_human("vendor_confirmation")              Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š       ELSE:                                            Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š          create_vendor() + LOW confidence flag         Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š                                                         Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š 3. Tarkista duplikaatti (check_duplicate_invoice)      Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š    IF DUPLICATE:                                       Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š       ask_human("duplicate_handling")                  Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š                                                         Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š 4. Ehdota tilejÃƒÂ¤ (suggest_account per line)            Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š    IF confidence < 0.85:                               Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š       status = "pending_review"                        Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š    ELSE:                                               Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š       status = "draft" (automaattinen)                 Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š                                                         Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š 5. Luo ostolasku (create_purchase_invoice)             Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ  Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  3. CONFIDENCE THRESHOLDS                                       Ã¢â€â€š
Ã¢â€â€š     Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Ã¢â‚¬Â¢ >0.90: Automaattinen kÃƒÂ¤sittely                       Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Ã¢â‚¬Â¢ 0.70-0.90: Luo draft, pyydÃƒÂ¤ vahvistus                Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Ã¢â‚¬Â¢ <0.70: Kysy ihmiseltÃƒÂ¤                                Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ  Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  4. LEARNING (Miten AI oppii)                                   Ã¢â€â€š
Ã¢â€â€š     Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Kun ihminen korjaa/vahvistaa:                          Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Ã¢â‚¬Â¢ Tallenna: original_suggestion, human_correction      Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Ã¢â‚¬Â¢ PÃƒÂ¤ivitÃƒÂ¤: accounting_rules (match_criteria)           Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š Ã¢â‚¬Â¢ Jos sama toimittaja:                                 Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€š     confidence++ seuraavalla kerralla                  Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š     Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ  Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ
```

**Dokumentointi:**
- **Sequence diagram** (miten AI ja ihminen interaktoivat)
- **Decision tree** (pÃƒÂ¤ÃƒÂ¤tÃƒÂ¶spuu eri skenaarioille)
- **State machine** (tilan muutokset)

---

### C. Moduulirajapinnat (Black Box Contracts)

**Perustuu Eskil Steenbergin systems-architecture -malliin:**

```
Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â
Ã¢â€â€š              MODUULI: Ostolaskujen kÃƒÂ¤sittely                    Ã¢â€â€š
Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â¤
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  INPUT (MitÃƒÂ¤ moduuli ottaa vastaan)                             Ã¢â€â€š
Ã¢â€â€š  Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Raw file (PDF, XML, EDI)                                Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Metadata (source, timestamp)                            Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ  Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  OUTPUT (MitÃƒÂ¤ moduuli palauttaa)                                Ã¢â€â€š
Ã¢â€â€š  Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Invoice object (structured data)                        Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Status (success/needs_review/error)                     Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Questions (if needs human input)                        Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ  Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  DEPENDENCIES (MitÃƒÂ¤ moduuli kÃƒÂ¤yttÃƒÂ¤ÃƒÂ¤)                            Ã¢â€â€š
Ã¢â€â€š  Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Voucher module (creates vouchers)                       Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Vendor module (manages vendors)                         Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ AI module (decision-making)                             Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ  Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  CONTRACT (Lupaus)                                              Ã¢â€â€š
Ã¢â€â€š  Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š "Annat minulle PDF-tiedoston ja metadataa.                Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š  Saat takaisin joko:                                      Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š  1. Validoidun Invoice-objektin (onnistunut)              Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š  2. Listan kysymyksiÃƒÂ¤ (tarvitaan ihmisen input)           Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š  3. Error-objektin (epÃƒÂ¤onnistunut)                        Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š                                                            Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š  Takaan ettÃƒÂ¤:                                             Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š  Ã¢â‚¬Â¢ En muuta tietokantaa ilman vahvistusta                 Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š  Ã¢â‚¬Â¢ Logitan kaikki pÃƒÂ¤ÃƒÂ¤tÃƒÂ¶kset                               Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š  Ã¢â‚¬Â¢ KÃƒÂ¤sittelen virheet hallitusti"                         Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ  Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  INTERNAL (Piilotettu, voi muuttua)                             Ã¢â€â€š
Ã¢â€â€š  Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ OCR-engine (voi vaihtaa)                                Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ ML-model (voi pÃƒÂ¤ivittÃƒÂ¤ÃƒÂ¤)                                Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Caching strategy (optimointi)                           Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ  Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ
```

**Esimerkki: Integration Wrapper (Finvoice)**

```
Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â
Ã¢â€â€š              WRAPPER: Finvoice Integration                      Ã¢â€â€š
Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â¤
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  MIKSI: EristÃƒÂ¤ÃƒÂ¤ ulkoinen riippuvuus (Finvoice)                  Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  PUBLIC API (Black Box Interface)                               Ã¢â€â€š
Ã¢â€â€š  Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š fetch_new_invoices(since: DateTime) -> List[RawInvoice]  Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š send_invoice(invoice: Invoice) -> SendResult             Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š get_status(invoice_id: str) -> InvoiceStatus             Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ  Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  INTERNAL (piilossa, voi vaihtaa)                               Ã¢â€â€š
Ã¢â€â€š  Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Finvoice 3.0 XML parsing                                Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ SOAP/REST adapter                                       Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Retry logic                                             Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€š Ã¢â‚¬Â¢ Error mapping                                           Ã¢â€â€š  Ã¢â€â€š
Ã¢â€â€š  Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ  Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€š  HYÃƒâ€“TY: Jos vaihdetaan Peppol:iin, muutetaan vain wrapper,     Ã¢â€â€š
Ã¢â€â€š         ei kosketa muuhun koodiin                               Ã¢â€â€š
Ã¢â€â€š                                                                 Ã¢â€â€š
Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ
```

---

## Ã°Å¸â€œÂ Dokumenttimalli: TECH_SPEC_XX

Ehdotan ettÃƒÂ¤ luomme uuden dokumenttityypin SPECien rinnalle:

### Rakenne

```markdown
# TECH_SPEC_XX: [Moduulin nimi]

> Perustuu: SPEC_XX_[Moduulin_nimi].md  
> TypeSpec lÃƒÂ¤hde: /specs/typespec/module_xx.tsp

---

## 1. API-rajapinnat

### 1.1 REST Endpoints (TypeSpec)

[TypeSpec-mÃƒÂ¤ÃƒÂ¤rittely tai linkki tiedostoon]

### 1.2 LLM-Optimized Descriptions

**Kriittinen:** Jokainen endpoint sisÃƒÂ¤ltÃƒÂ¤ÃƒÂ¤ "Verbose Description" AI:lle:
- MILLOIN endpoint kutsutaan
- MITÃƒâ€ž edellytyksiÃƒÂ¤
- MITKÃƒâ€ž sivuvaikutukset (x-openai-isConsequential)

### 1.3 Validoinnit

[MitÃƒÂ¤ tarkistetaan, virheviestit]

---

## 2. AI-agentin mÃƒÂ¤ÃƒÂ¤rittely

### 2.1 Tools (TyÃƒÂ¶kalut)

| Tool | Input | Output | Purpose |
|------|-------|--------|---------|
| search_vendor | business_id | Vendor or null | Etsi toimittaja |
| ... | ... | ... | ... |

### 2.2 Decision Logic

[Flowchart tai pseudokoodi]

### 2.3 Confidence Thresholds

| Toiminto | Threshold | Seuraava vaihe |
|----------|-----------|----------------|
| Automaattinen | >0.90 | Kirjaa suoraan |
| Vahvistus | 0.70-0.90 | Luo draft + pyydÃƒÂ¤ vahvistus |
| Kysymys | <0.70 | Kysy ihmiseltÃƒÂ¤ |

### 2.4 Learning Patterns

[Miten AI oppii kÃƒÂ¤yttÃƒÂ¤jÃƒÂ¤n valinnoista]

### 2.5 System Prompt Architecture Ã¢Â¬â€¦Ã¯Â¸Â UUSI

**Persona:**
[AI:n rooli, arvot, kÃƒÂ¤yttÃƒÂ¤ytyminen]

**Context Window Strategy:**
[MitÃƒÂ¤ tietoa syÃƒÂ¶tetÃƒÂ¤ÃƒÂ¤n promptiin]

**Few-Shot Examples:**
[2-3 esimerkkiÃƒÂ¤ onnistuneista pÃƒÂ¤ÃƒÂ¤tÃƒÂ¶ksistÃƒÂ¤]

### 2.6 Output Guardrails Ã¢Â¬â€¦Ã¯Â¸Â UUSI

**Deterministiset sÃƒÂ¤ÃƒÂ¤nnÃƒÂ¶t ennen API-kutsua:**
```python
# Esimerkki: Laskun summan tÃƒÂ¤smÃƒÂ¤ytys
def validate_invoice_total(invoice):
    lines_sum = sum(line.amount for line in invoice.lines)
    if abs(invoice.total - lines_sum) > 0.01:
        raise GuardrailViolation("Total mismatch")
```

[Lista kaikista guardrail-sÃƒÂ¤ÃƒÂ¤nnÃƒÂ¶istÃƒÂ¤]

---

## 3. Moduulirajapinnat

### 3.1 Black Box Contract

**Input:**
- [MitÃƒÂ¤ moduuli ottaa vastaan]

**Output:**
- [MitÃƒÂ¤ moduuli palauttaa]

**Dependencies:**
- [MitÃƒÂ¤ muita moduuleja kÃƒÂ¤yttÃƒÂ¤ÃƒÂ¤]

**Guarantees:**
- [MitÃƒÂ¤ moduuli lupaa]

### 3.2 Integration Wrappers

[Ulkoiset palvelut ja niiden wrapperit]

---

## 4. Data Flow

[Sequence diagram: Miten data liikkuu]

### 4.1 Asynkroninen prosessointi Ã¢Â¬â€¦Ã¯Â¸Â UUSI

**Webhook/Polling-malli:**
```
1. Client Ã¢â€ â€™ POST /jobs Ã¢â€ â€™ 202 Accepted + job_id
2. AI prosessoi taustalla
3. Client pollaa GET /jobs/{id} tai saa webhook:n
4. Status: processing Ã¢â€ â€™ needs_review Ã¢â€ â€™ completed
```

---

## 5. Error Handling

### 5.1 Self-Correction Loop Ã¢Â¬â€¦Ã¯Â¸Â UUSI

**Max retries:** 3

**Korjausstrategiat:**
1. Vendor not found Ã¢â€ â€™ Etsi Y-tunnuksella Ã¢â€ â€™ Luo uusi
2. Validation error Ã¢â€ â€™ Korjaa mitÃƒÂ¤ voi Ã¢â€ â€™ Kysy ihmiseltÃƒÂ¤
3. Timeout Ã¢â€ â€™ Retry exponential backoff

[MitÃƒÂ¤ tehdÃƒÂ¤ÃƒÂ¤n virhetilanteissa]

---

## 6. Testing Strategy

### 6.1 Unit Tests
[Perinteiset yksikkÃƒÂ¶testit]

### 6.2 Integration Tests
[API-testit]

### 6.3 AI Agent Tests Ã¢Â¬â€¦Ã¯Â¸Â UUSI

**Golden Dataset:** 20 esimerkkitapausta
- SÃƒÂ¤ilytyspaikka: /tests/golden_datasets/module_xx/
- CI/CD: Accuracy threshold >= 85%
- Regression check: Jos accuracy laskee, deploy estyy

**Chain of Thought -lokitus:**
- Jokainen AI:n pÃƒÂ¤ÃƒÂ¤tÃƒÂ¶s logitetaan jÃƒÂ¤ljitettÃƒÂ¤vyyden vuoksi
- TyÃƒÂ¶kalu: LangSmith / W&B / custom

---

## 7. Observability Ã¢Â¬â€¦Ã¯Â¸Â UUSI

### 7.1 Metrics
- AI confidence distribution (histogrammi)
- Human review rate (tavoite <15%)
- Processing time (median, p95, p99)
- Error rate by type

### 7.2 Alerts
- Confidence drop (keskiarvo <0.80)
- Human review spike (>30%)
- Processing timeout (>60s)

### 7.3 Dashboards
[Grafana/CloudWatch/DataDog]

---

## 8. Implementation Notes

[Muistiinpanoja toteutukselle]
```

---

## Ã°Å¸â€â€ž Prosessi: SPEC Ã¢â€ â€™ TECH_SPEC Ã¢â€ â€™ CODE

### Vaihe 1: Toiminnallinen mÃƒÂ¤ÃƒÂ¤rittely (SPEC)
**Kuka:** Projekti-tiimi + AI (Claude)
**Tulos:** SPEC_XX.md
- Liiketoimintaprosessit
- AI:n rooli (yleisellÃƒÂ¤ tasolla)
- KÃƒÂ¤yttÃƒÂ¤jÃƒÂ¤tarinat
- Tietorakenteet (taulut)

### Vaihe 2: Tekninen mÃƒÂ¤ÃƒÂ¤rittely (TECH_SPEC) Ã¢Â¬â€¦Ã¯Â¸Â UUSI
**Kuka:** Tekninen arkkitehti + AI (Claude)
**Tulos:** TECH_SPEC_XX.md + openapi_spec.yaml
- API-rajapinnat (OpenAPI)
- AI-agentin logiikka (pseudokoodi)
- Moduulirajapinnat (black box contracts)
- Sequence diagramit

**Milloin:** 
- Kun SPEC on ~80% valmis
- Ennen koodauksen aloittamista
- PÃƒÂ¤ivitetÃƒÂ¤ÃƒÂ¤n kun SPEC muuttuu

### Vaihe 3: Toteutus (CODE)
**Kuka:** KehittÃƒÂ¤jÃƒÂ¤t + Claude Code
**Tulos:** Python/JavaScript/etc. koodi
- Toteutetaan TECH_SPEC:in mukaan
- Validoidaan OpenAPI:a vasten
- Testataan

---

## Ã°Å¸Å½Â¨ TyÃƒÂ¶kalut

### API-mÃƒÂ¤ÃƒÂ¤rittelyyn
- **OpenAPI 3.1** (YAML/JSON)
- **TypeSpec** (Microsoft, koodi Ã¢â€ â€™ OpenAPI)
- **Swagger Editor** (visuaalinen editor)
- **Postman** (testaus)

### AI-agentin mÃƒÂ¤ÃƒÂ¤rittelyyn
- **Mermaid** (diagrammit)
- **Pseudokoodi** (selkeÃƒÂ¤, yksinkertainen)
- **LangChain** (jos kÃƒÂ¤ytetÃƒÂ¤ÃƒÂ¤n toteutuksessa)

### ModuulimÃƒÂ¤ÃƒÂ¤rittelyyn
- **C4 Model** (Context, Container, Component, Code)
- **Mermaid flowcharts**
- **Markdown tables** (yksinkertainen, selvÃƒÂ¤)

---

## Ã°Å¸Ââ€”Ã¯Â¸Â Sovellus meidÃƒÂ¤n projektiin

### Prioriteetti: SPEC_04 (Myyntilaskut)

Kun SPEC_04 on valmis (~80%), luodaan:

**1. TECH_SPEC_04_Myyntilaskut.md**
- API-rajapinnat myyntilaskuille
- AI-agentin logiikka (laskutuksen automaatio)
- Moduulirajapinnat

**2. openapi_sales_invoices.yaml**
- REST endpoints
- Request/response skemat
- Validoinnit

**3. AI_Agent_Sales.mermaid**
- Decision flowchart
- Tool definitions
- Learning patterns

### TyÃƒÂ¶nkulku

```
Session 1: SPEC_04 (~80% valmis)
    Ã¢â€â€š
    Ã¢â€“Â¼
Session 2: TECH_SPEC_04 luonti
    Ã¢â€â€š 1. API-rajapinnat (OpenAPI)
    Ã¢â€â€š 2. AI-agentin logiikka
    Ã¢â€â€š 3. Moduulirajapinnat
    Ã¢â€“Â¼
Review: KÃƒÂ¤yttÃƒÂ¤jÃƒÂ¤ tarkistaa TECH_SPEC
    Ã¢â€â€š
    Ã¢â€“Â¼
Session 3: SPEC_04 viimeistely + DATABASE pÃƒÂ¤ivitys
    Ã¢â€â€š
    Ã¢â€“Â¼
VALMIS koodaukseen (Claude Code)
```

---

## Ã°Å¸â€™Â¡ HyÃƒÂ¶dyt

### SPEC:in nÃƒÂ¤kÃƒÂ¶kulmasta
- Ã¢Å“â€¦ SelkeÃƒÂ¤mpi fokus (ei teknisiÃƒÂ¤ yksityiskohtia)
- Ã¢Å“â€¦ Helpompi lukea ei-teknisille sidosryhmille
- Ã¢Å“â€¦ Toiminnallinen ymmÃƒÂ¤rrys sÃƒÂ¤ilyy puhtaana

### TECH_SPEC:in nÃƒÂ¤kÃƒÂ¶kulmasta
- Ã¢Å“â€¦ SelkeÃƒÂ¤ sopimus (contract) koodaukselle
- Ã¢Å“â€¦ AI-logiikka dokumentoitu (ei "musta laatikko")
- Ã¢Å“â€¦ Testattavissa ENNEN koodausta

### Koodauksen nÃƒÂ¤kÃƒÂ¶kulmasta
- Ã¢Å“â€¦ Yksiselitteinen ohje (ei tulkintaa)
- Ã¢Å“â€¦ OpenAPI Ã¢â€ â€™ automaattinen validointi
- Ã¢Å“â€¦ Black box contracts Ã¢â€ â€™ modulaarinen

---

## Ã°Å¸Â¤â€ Avoimet kysymykset Ã¢â€ â€™ Ratkaistut Ã¢Å“â€¦

### 1. TyÃƒÂ¶kalut: Ã¢Å“â€¦ RATKAISTU + TARKENNETTU v1.2

**Kysymys:** KÃƒÂ¤ytÃƒÂ¤mmekÃƒÂ¶ TypeSpec vai pelkkÃƒÂ¤ÃƒÂ¤ OpenAPI YAML:ia?

**Vastaus (Gemini):** **TypeSpec**
- Modulaarisempi ja tiiviimpi
- Code-first -tyyli tuttu kehittÃƒÂ¤jille
- Generoi sekÃƒÂ¤ OpenAPI YAML:in (AI:lle) ettÃƒÂ¤ JSON Scheman (koodille)
- Tukee vendor extensions (x-openai-isConsequential)

**PÃƒÂ¤ÃƒÂ¤tÃƒÂ¶s:** TypeSpec tekniseen mÃƒÂ¤ÃƒÂ¤rittelyyn, generoitu OpenAPI YAML AI:lle.

**SÃƒÂ¤ilytyspaikka (TARKENNETTU v1.2):**
```
Git (Versionhallinta - TEKNINEN):
/api/
  Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬ typespec/
  Ã¢â€â€š   Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬ common/
  Ã¢â€â€š   Ã¢â€â€š   Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬ models.tsp          (Yhteiset mallit)
  Ã¢â€â€š   Ã¢â€â€š   Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬ errors.tsp          (Virhemallit)
  Ã¢â€â€š   Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬ modules/
  Ã¢â€â€š   Ã¢â€â€š   Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬ vouchers.tsp        (TECH_SPEC_01)
  Ã¢â€â€š   Ã¢â€â€š   Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬ purchase_invoices.tsp (TECH_SPEC_02)
  Ã¢â€â€š   Ã¢â€â€š   Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬ sales_invoices.tsp  (TECH_SPEC_04)
  Ã¢â€â€š   Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬ main.tsp                (Kokoaa kaikki)
  Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬ generated/
      Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬ openapi.yaml            (AI:lle, generoidaan automaattisesti)
      Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬ schemas/                (JSON Schema koodille)
          Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬ Invoice.json
          Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬ Vendor.json

Google Drive (TOIMINNALLINEN):
/SPEC_XX dokumentit (korkean tason kuvaus)
```

**Perustelu (Gemini v1.2):**
- **Git:** API-speksi on koodia ("Infrastructure as Code") - kuuluu samaan paikkaan kuin lÃƒÂ¤hdekoodi
- **Drive:** SPEC_XX on ihmisille (ei-tekniset sidosryhmÃƒÂ¤t) - sÃƒÂ¤ilytetÃƒÂ¤ÃƒÂ¤n Drive:ssa
- **TECH_SPEC_XX:** Voi olla kummassakin, mutta suositus Git (lÃƒÂ¤hellÃƒÂ¤ TypeSpec-tiedostoja)

**Konkreettinen TypeSpec-esimerkki (Gemini v1.2):**
```typescript
// api/typespec/modules/purchase_invoices.tsp

import "@typespec/http";
using TypeSpec.Http;

namespace AIAccounting.PurchaseInvoices;

/**
 * Invoice creation model with AI-specific fields.
 * 
 * @doc AI Agent: Use this model ONLY after:
 *      1. Vendor validation (search_vendor)
 *      2. Duplicate check (check_duplicate_invoice)
 *      3. Account suggestions validated
 */
model InvoiceCreateRequest {
  /** 
   * Strict official business ID found on invoice document.
   * Ã¢Å¡Â Ã¯Â¸Â DO NOT hallucinate. Must exist in vendors table.
   */
  @doc("Vendor UUID - use search_vendor first")
  vendor_id: string;

  /** 
   * Invoice number as shown on document.
   * Used for duplicate detection.
   */
  @maxLength(50)
  invoice_number: string;

  /** 
   * Total amount including VAT.
   * Ã¢Å¡Â Ã¯Â¸Â MUST match sum of lines (tolerance Ã‚Â±0.01Ã¢â€šÂ¬)
   * This will be validated by Guardrails.
   */
  @doc("Total incl. VAT - must match lines")
  total_amount: decimal;
  
  /**
   * AI confidence score [0.0 - 1.0].
   * If < 0.85, you MUST set needs_human_review = true.
   */
  @minValue(0.0)
  @maxValue(1.0)
  ai_confidence: float32;
  
  /**
   * Force human review flag.
   * Set true if: OCR quality poor, new vendor, or logic uncertain.
   */
  needs_human_review: boolean;
  
  /** Invoice line items */
  @minItems(1)
  lines: InvoiceLine[];
}

model InvoiceLine {
  @doc("Line description from invoice")
  description: string;
  
  @doc("Line amount excl. VAT")
  amount: decimal;
  
  @doc("Account ID from chart of accounts")
  account_id: string;
  
  @doc("VAT code (KOMY24, KOOS, etc.)")
  vat_code: string;
}

/**
 * Invoice Service - AI-optimized endpoints
 */
@route("/api/v1/purchase-invoices")
interface PurchaseInvoiceService {
  
  /**
   * Create a new purchase invoice (AI Agent endpoint).
   * 
   * Ã¢Å¡Â Ã¯Â¸Â CONSEQUENTIAL: This commits financial data to database.
   * 
   * @returns 202 Accepted - Processing started (async)
   * @returns 400 Bad Request - Validation failed
   */
  @post
  @tag("AI-Consequential")  // Vendor extension for AI
  @doc("AI: Use ONLY after vendor validation and duplicate check")
  create(
    @body invoice: InvoiceCreateRequest
  ): {
    @statusCode statusCode: 202;
    @body job: {
      job_id: string;
      status: "processing";
    };
  } | {
    @statusCode statusCode: 400;
    @body error: {
      error_code: string;
      error_message: string;
      validation_errors: string[];
    };
  };
  
  /**
   * Check job status (for async processing).
   */
  @get
  @route("/{jobId}/status")
  getJobStatus(
    @path jobId: string
  ): {
    @statusCode statusCode: 200;
    @body status: {
      job_id: string;
      status: "processing" | "needs_review" | "completed" | "failed";
      invoice_id?: string;  // When completed
      question?: {          // When needs_review
        question_type: string;
        question_text: string;
      };
    };
  };
}
```

**Generoidaan OpenAPI:**
```bash
# CI/CD pipeline
tsp compile api/typespec/main.tsp --emit @typespec/openapi3
# Ã¢â€ â€™ api/generated/openapi.yaml (AI:n kÃƒÂ¤yttÃƒÂ¶ÃƒÂ¶n)
```

---

### 2. Prosessi: Ã¢Å“â€¦ RATKAISTU + TARKENNETTU v1.2

**Kysymys:** TeemmekÃƒÂ¶ TECH_SPEC:in ennen vai jÃƒÂ¤lkeen DATABASE-pÃƒÂ¤ivityksen?

**Vastaus v1.1 (Gemini):** ~~Rinnakkain~~

**TARKENNUS v1.2 (Gemini):** **API ensin Ã¢â€ â€™ sitten Database**

**Perustelu:**
- API on **"sopimus"** (Contract) - mitÃƒÂ¤ data pitÃƒÂ¤ÃƒÂ¤ sisÃƒÂ¤ltÃƒÂ¤ÃƒÂ¤ ja miten se kÃƒÂ¤yttÃƒÂ¤ytyy
- Database on **toteutusdetaili** - vain tapa tallentaa data sopimuksen tÃƒÂ¤yttÃƒÂ¤miseksi
- Jos suunnittelet DB ensin, **vuodat** usein tietokannan sisÃƒÂ¤iset rakenteet API:in Ã¢â€ â€™ huono arkkitehtuuri

**TyÃƒÂ¶nkulku (korjattu):**
```
Week 1: SPEC_04 "happy path" (80%)
        Ã¢â€â€š
        Ã¢â€â€Ã¢â€â‚¬Ã¢â€“Âº TECH_SPEC_04: API-mÃƒÂ¤ÃƒÂ¤rittely (JSON-rakenteet)
            Ã¢â€â€š
            Ã¢â€â€Ã¢â€â‚¬Ã¢â€“Âº "MitÃƒÂ¤ UI/AI tarvitsee?"
                
Week 2: SPEC_04 viimeistely
        Ã¢â€â€š
        Ã¢â€Å“Ã¢â€â‚¬Ã¢â€“Âº TECH_SPEC_04: AI-logiikka + guardrails
        Ã¢â€â€š
        Ã¢â€â€Ã¢â€â‚¬Ã¢â€“Âº DATABASE_CHANGELOG: "MitkÃƒÂ¤ SQL-taulut tarvitaan API:n toteuttamiseen?"
            
Week 3: DATABASE_MODEL pÃƒÂ¤ivitys
        Ã¢â€â€š
        Ã¢â€â€Ã¢â€â‚¬Ã¢â€“Âº Suunnittele taulut API:n JSON-rakenteiden pohjalta
            
Week 4: Review + viimeistely
        Ã¢â€â€š
        Ã¢â€â€Ã¢â€â‚¬Ã¢â€“Âº Kaikki valmiina koodaukseen
```

**SÃƒÂ¤ÃƒÂ¤ntÃƒÂ¶:** Define JSON structure (what UI/AI needs) Ã¢â€ â€™ Design SQL tables based on that.

---

### 3. Dokumenttirakenne: Ã¢Å“â€¦ RATKAISTU + TARKENNETTU v1.2

**Kysymys:** Oma TECH_SPEC per moduuli vai yksi iso?

**Vastaus (Gemini):** **Moduulikohtainen**
- TECH_SPEC_02.md, TECH_SPEC_04.md jne.
- Yksi iso tiedosto kasvaa hallitsemattomaksi
- Helpompi yllÃƒÂ¤pitÃƒÂ¤ÃƒÂ¤ ja versioida

**Synkronointi SPEC Ã¢â€ â€ TECH_SPEC (TARKENNETTU v1.2):**

**Perinteinen tapa (manuaalinen):**
```markdown
# TECH_SPEC_04.md
> Perustuu: SPEC_04_Myyntilaskujen_kasittely.md (v1.0)
> Synkronoitu: 2025-11-25

Jos SPEC muuttuu:
1. Merkitse TECH_SPEC:iin "OUT OF SYNC"
2. Review + pÃƒÂ¤ivitÃƒÂ¤ TECH_SPEC
3. PÃƒÂ¤ivitÃƒÂ¤ "Synkronoitu" -pÃƒÂ¤ivÃƒÂ¤mÃƒÂ¤ÃƒÂ¤rÃƒÂ¤
```

**AI-avusteinen tapa (suositeltu v1.2):**

**"Gap Analysis" -tyÃƒÂ¶kalu:**

```markdown
# Prompt Claudelle projektissa:

"Tee Gap Analysis seuraavien dokumenttien vÃƒÂ¤lille:
- SPEC_04_Myyntilaskujen_kasittely.md (v1.1)
- TECH_SPEC_04_Myyntilaskujen_kasittely.md (v1.0)

Tunnista:
1. MitÃƒÂ¤ UUTTA SPEC:ssÃƒÂ¤ on, jota ei ole TECH_SPEC:ssÃƒÂ¤?
2. MitÃƒÂ¤ on MUUTTUNUT SPEC:ssÃƒÂ¤, mikÃƒÂ¤ vaatii TECH_SPEC:n pÃƒÂ¤ivitystÃƒÂ¤?
3. MitÃƒÂ¤ TECH_SPEC:ssÃƒÂ¤ on, joka on nyt VANHENTUNUTTA?

Anna suositukset pÃƒÂ¤ivityksiin."
```

**AI:n vastaus (esimerkki):**
```markdown
## Gap Analysis: SPEC_04 v1.1 vs TECH_SPEC_04 v1.0

### Ã°Å¸â€ â€¢ Uutta SPEC:ssÃƒÂ¤ (puuttuu TECH_SPEC:stÃƒÂ¤)

1. **Toistuvat laskut** (SPEC 4.5)
   - SPEC: Automaattinen lÃƒÂ¤hetys kuukausittain
   - TECH_SPEC: Ei mainintaa
   - **Toimenpide:** LisÃƒÂ¤ÃƒÂ¤ TECH_SPEC:iin API-endpoint 
     POST /recurring-invoices + cron job -logiikka

2. **Maksumuistutukset** (SPEC 4.6)
   - SPEC: 3-portainen muistutusprosessi
   - TECH_SPEC: Ei toteutusta
   - **Toimenpide:** LisÃƒÂ¤ÃƒÂ¤ osio "2.7 Reminder Logic"

### Ã°Å¸â€â€ž Muuttunut SPEC:ssÃƒÂ¤ (vaatii pÃƒÂ¤ivitystÃƒÂ¤)

1. **ALV-kÃƒÂ¤sittely** (SPEC 4.3)
   - SPEC v1.0: "AI ehdottaa ALV:n"
   - SPEC v1.1: "AI VALITSEE ALV:n automaattisesti (>90% luottamus)"
   - **Toimenpide:** PÃƒÂ¤ivitÃƒÂ¤ TECH_SPEC osio "2.3 Confidence Thresholds"
     - LisÃƒÂ¤ÃƒÂ¤ ALV-valinta automaattisiin (>0.90)

### Ã¢ÂÅ’ Vanhentunutta TECH_SPEC:ssÃƒÂ¤

1. **Tulostusominaisuus** (TECH_SPEC 3.2)
   - TECH_SPEC: API-endpoint /invoices/{id}/print
   - SPEC: Ei enÃƒÂ¤ÃƒÂ¤ mainintaa (poistettu)
   - **Toimenpide:** Poista TECH_SPEC:stÃƒÂ¤ tai siirrÃƒÂ¤ backlogiin

## Suositus

PÃƒÂ¤ivitÃƒÂ¤ TECH_SPEC_04 Ã¢â€ â€™ v1.1:
- LisÃƒÂ¤ÃƒÂ¤ 2 uutta API-endpointia
- Muuta confidence thresholds
- Poista vanhentuneet osat
```

**HyÃƒÂ¶ty:** AI tekee "diff:in" nopeasti ja ehdottaa tarvittavat muutokset.

---

### 4. AI-testaus: Ã¢Å“â€¦ RATKAISTU + TARKENNETTU v1.2

**Kysymys:** Miten testaamme AI-agentin pÃƒÂ¤ÃƒÂ¤tÃƒÂ¶ksentekoa?

**Vastaus (Gemini):** **Golden Dataset + CI/CD + Nightly Evals**

**TERMINOLOGINEN TARKENNUS v1.2:**

**"Evals" ei "Tests"**
- **Perinteinen koodi:** Deterministinen (1+1=2) Ã¢â€ â€™ **Tests**
- **AI-pÃƒÂ¤ÃƒÂ¤ttely:** Probabilistinen (todennÃƒÂ¤kÃƒÂ¶isyys 90%) Ã¢â€ â€™ **Evals** (Evaluations)

**Perustelu:** AI:n "oikea" vastaus voi vaihdella. Mittaamme **tarkkuutta kokonaisuutena**, ei yksittÃƒÂ¤isiÃƒÂ¤ oikein/vÃƒÂ¤ÃƒÂ¤rin.

---

**Golden Dataset (sama kuin v1.1):**
- 20 esimerkkitapausta per moduuli
- Input (PDF/JSON) + Expected Output
- SÃƒÂ¤ilytyspaikka: `/tests/golden_datasets/`

**CI/CD Evals (TARKENNETTU v1.2):**
```python
def evaluate_agent_accuracy():
    """
    Evals (ei tests!) - Probabilistinen mittaus.
    
    HUOM: TÃƒÂ¤mÃƒÂ¤ EI ole perinteinen unit test.
    AI voi antaa eri vastauksia eri ajoilla.
    Mittaamme kokonaistarkkuutta.
    """
    dataset = load_golden_dataset('purchase_invoices')
    agent = PurchaseInvoiceAgent()
    
    results = []
    for case in dataset:
        prediction = agent.process(case['input'])
        expected = case['expected_output']
        
        # Fuzzy matching (ei strict equality)
        match_score = calculate_similarity(prediction, expected)
        # 1.0 = tÃƒÂ¤ysi osuma, 0.9 = lÃƒÂ¤hes oikein, <0.8 = virhe
        
        results.append({
            'case_id': case['id'],
            'match_score': match_score,
            'confidence': prediction.confidence
        })
    
    # KeskimÃƒÂ¤ÃƒÂ¤rÃƒÂ¤inen tarkkuus
    avg_accuracy = sum(r['match_score'] for r in results) / len(results)
    
    # KRIITTINEN: Regression check
    # Jos tarkkuus laskee, deployment estetÃƒÂ¤ÃƒÂ¤n
    assert avg_accuracy >= 0.85, f"AI accuracy dropped: {avg_accuracy}"
    
    # Log tulokset
    logger.info({
        'eval_run': datetime.now(),
        'avg_accuracy': avg_accuracy,
        'cases_passed': sum(r['match_score'] > 0.9 for r in results),
        'cases_total': len(results)
    })
```

**Nightly Runs (UUSI v1.2):**

```yaml
# .github/workflows/nightly-evals.yml

name: Nightly AI Evals

on:
  schedule:
    # Ajetaan joka yÃƒÂ¶ klo 03:00 UTC
    - cron: '0 3 * * *'
  workflow_dispatch:  # Voi ajaa myÃƒÂ¶s manuaalisesti

jobs:
  evaluate-agents:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Purchase Invoice Evals
        run: |
          python tests/evals/purchase_invoices.py
          
      - name: Run Sales Invoice Evals
        run: |
          python tests/evals/sales_invoices.py
      
      - name: Check Accuracy Thresholds
        run: |
          # Jos tarkkuus alle 85%, lÃƒÂ¤hetÃƒÂ¤ hÃƒÂ¤lytys
          python tests/evals/check_regression.py
      
      - name: Notify if Failed
        if: failure()
        uses: slack-notify
        with:
          message: "Ã¢Å¡Â Ã¯Â¸Â AI Accuracy dropped below 85%!"
```

**Miksi nightly?**
- LLM-mallit voivat muuttua (API-provider pÃƒÂ¤ivittÃƒÂ¤ÃƒÂ¤)
- Data drift (uusia laskutyyppejÃƒÂ¤)
- Agentin prompt muutokset
- **Proaktiivinen** ongelmien havaitseminen ennen kuin kÃƒÂ¤yttÃƒÂ¤jÃƒÂ¤t huomaavat

**Dashboard:**
```
Ã¢â€Å’Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â
Ã¢â€â€š       AI Agent Accuracy Tracker (Last 30 days)     Ã¢â€â€š
Ã¢â€Å“Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Â¤
Ã¢â€â€š                                                     Ã¢â€â€š
Ã¢â€â€š  Purchase Invoices:  Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“â€˜Ã¢â€“â€˜  92%       Ã¢â€â€š
Ã¢â€â€š  Sales Invoices:     Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“â€˜Ã¢â€“â€˜  91%       Ã¢â€â€š
Ã¢â€â€š  Bank Statements:    Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“Ë†Ã¢â€“â€˜  96%       Ã¢â€â€š
Ã¢â€â€š                                                     Ã¢â€â€š
Ã¢â€â€š  Trend: Ã¢â€ â€˜ +2% this week                            Ã¢â€â€š
Ã¢â€â€š                                                     Ã¢â€â€š
Ã¢â€â€š  Last Regression: 2025-11-20 (Purchase 83% Ã¢â€ â€™ Fixed)Ã¢â€â€š
Ã¢â€â€Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€â‚¬Ã¢â€Ëœ
```

---

**Chain of Thought -lokitus (sama kuin v1.1):**
- Jokainen AI:n pÃƒÂ¤ÃƒÂ¤tÃƒÂ¶s logitetaan
- TyÃƒÂ¶kalu: LangSmith, Weights & Biases tai custom
- JÃƒÂ¤ljitettÃƒÂ¤vyys: Pakollinen taloushallinnossa

---

### 5. Asynkronisuus: Ã¢Å“â€¦ RATKAISTU

**Kysymys:** Miten kÃƒÂ¤sitellÃƒÂ¤ÃƒÂ¤n pitkÃƒÂ¤t AI-prosessoinnit?

**Vastaus (Gemini):** **Webhook/Polling-malli**

```
Synkroninen (Ã¢ÂÅ’ riski: timeout):
Client Ã¢â€ â€™ POST /invoices Ã¢â€ â€™ [AI 5-30s] Ã¢â€ â€™ 201 Created

Asynkroninen (Ã¢Å“â€¦ skaalautuva):
Client Ã¢â€ â€™ POST /jobs Ã¢â€ â€™ 202 Accepted + job_id
         [AI taustalla]
Client Ã¢â€ â€™ GET /jobs/{id} Ã¢â€ â€™ {status: "processing"}
         [Pollaa 2s vÃƒÂ¤lein]
Client Ã¢â€ â€™ GET /jobs/{id} Ã¢â€ â€™ {status: "needs_review"}
         [Ihminen vastaa]
Client Ã¢â€ â€™ POST /jobs/{id}/answer Ã¢â€ â€™ OK
Client Ã¢â€ â€™ GET /jobs/{id} Ã¢â€ â€™ {status: "completed"}
```

---

### 6. Guardrails: Ã¢Å“â€¦ UUSI LÃƒâ€“YDÃƒâ€“S

**Kysymys:** Miten varmistetaan AI:n pÃƒÂ¤ÃƒÂ¤tÃƒÂ¶sten oikeellisuus?

**Vastaus (Gemini):** **Output Guardrails** - deterministiset sÃƒÂ¤ÃƒÂ¤nnÃƒÂ¶t

**Sijainti:** TECH_SPEC osiossa "2.6 Output Guardrails"

**Esimerkki:**
```python
# Ennen kuin AI:n output hyvÃƒÂ¤ksytÃƒÂ¤ÃƒÂ¤n:
1. Laskun loppusumma = rivien summa (Ã‚Â±0.01Ã¢â€šÂ¬)
2. PÃƒÂ¤ivÃƒÂ¤mÃƒÂ¤ÃƒÂ¤rÃƒÂ¤t loogisia (due_date >= invoice_date)
3. Vendor exists (ei hallusinoituja ID:itÃƒÂ¤)
4. ALV-koodit valideja (KOMY24, KOOS jne.)
```

**HyÃƒÂ¶ty:** EstÃƒÂ¤ÃƒÂ¤ loogisesti virheelliset pÃƒÂ¤ÃƒÂ¤tÃƒÂ¶kset **ennen** commitia.

---

### 7. System Prompt: Ã¢Å“â€¦ UUSI LÃƒâ€“YDÃƒâ€“S

**Kysymys:** Miten AI:n kÃƒÂ¤yttÃƒÂ¤ytyminen dokumentoidaan?

**Vastaus (Gemini):** **System Prompt Architecture** 

**Sijainti:** TECH_SPEC osiossa "2.5 System Prompt Architecture"

**SisÃƒÂ¤ltÃƒÂ¤ÃƒÂ¤:**
1. **Persona** - AI:n rooli ja arvot
2. **Context Window Strategy** - MitÃƒÂ¤ dataa syÃƒÂ¶tetÃƒÂ¤ÃƒÂ¤n
3. **Few-Shot Examples** - 2-3 esimerkkiÃƒÂ¤

**Esimerkki:**
```
Persona:
"Olet tarkka kirjanpitoavustaja. Jos epÃƒÂ¤varma ALV-kannasta,
 valitse turvallisempi vaihtoehto ja merkkaa 'review_needed'."

Context:
- Tilikartta (accounts) + ai_keywords
- Toimittajan historia (last 5 invoices)
- Accounting rules (vendor-specific)
```

---

## Ã°Å¸â€œÅ¡ LÃƒÂ¤hteet

### API-First Design
- Microsoft ISE: API Design-First with TypeSpec (2024)
- Moesif: API Design-First methodology (2024)
- InfoQ: Design-First approach (2022)

### AI Agent Architecture
- Anthropic: Building Effective Agents (2024)
- FME Safe: AI Agent Architecture Tutorial (2024)
- Microsoft Azure: AI Agent Orchestration Patterns (2024)
- IBM: Agentic Architecture (2024)

### Systems Architecture
- Eskil Steenberg: Black box principles
- Systems-architecture skill (projektin skill)

### Enterprise-Grade AI Systems Ã¢Â¬â€¦Ã¯Â¸Â UUSI
- **Google Gemini (AI Software Architect):** Peer review 2025-11-23
  - LLM-Optimized OpenAPI patterns
  - Guardrails & Validation Layer best practices
  - Golden Dataset methodology
  - System Prompt Architecture patterns

---

## Muutoshistoria

| Versio | PÃƒÂ¤ivÃƒÂ¤mÃƒÂ¤ÃƒÂ¤rÃƒÂ¤ | Muutokset |
|--------|------------|-----------|
| 1.2 | 2025-11-23 | KÃƒÂ¤ytÃƒÂ¤nnÃƒÂ¶n tarkennukset (Gemini v1.1): **API ensin Ã¢â€ â€™ sitten DB** (ei rinnakkain), TypeSpec konkreettinen esimerkki, Git vs Drive -jako, Gap Analysis -tyÃƒÂ¶kalu synkronointiin, "Evals" terminologia, Nightly runs |
| 1.1 | 2025-11-23 | Enterprise-grade lisÃƒÂ¤ykset (Gemini peer review): LLM-Optimized OpenAPI, Guardrails, Observability, Asynkronisuus, System Prompt, Self-Correction, TypeSpec-suositus |
| 1.0 | 2025-11-23 | EnsimmÃƒÂ¤inen versio - synteesi kolmesta nÃƒÂ¤kÃƒÂ¶kulmasta |

---

*Dokumentti on osa AI-taloushallintojÃƒÂ¤rjestelmÃƒÂ¤n suunnitteludokumentaatiota.*
