# VEYRIN - Hostile Diligence and Product Direction
Date: 22 September 2026
Status: Working product thesis after deep public-source validation

## Executive decision

VEYRIN should continue to validation, but its wedge is not retailer connectivity itself.

The strongest supported position is:

> VEYRIN is the cross-retailer commercial settlement and deduction intelligence layer between retailer data and the supplier's finance environment.

Existing EDI, API, portal and integration-provider connections should be reused wherever possible. VEYRIN should focus on normalising retailer-specific information, linking it to supplier-side evidence, classifying exceptions, preparing action, and tracking commercial outcomes.

Do not build a full EDI gateway, retailer portal replacement, SAP replacement, bank reconciliation product, or autonomous dispute bot as V1.

---

## What is now verified

### Shoprite

Shoprite has a live documented supplier REST API and public implementation guide.

Publicly exposed services include:
- VendorOrder
- VendorClaim
- VendorASN
- VendorRebate
- VendorInvoice
- B2BInvoice

Shoprite describes Debit/Credit Advice claims as monetary adjustments for cases including damaged goods, returns, incorrect quantity received and incorrect pricing.

Critically, Shoprite's implementation guide explicitly supports an agent model. The supplier/trading partner master user creates and maintains accounts for agents downloading on the supplier's behalf.

Implication for VEYRIN:
- Third-party technical access is not merely inferred.
- A live Shoprite connector is feasible where the supplier authorises it.
- Credential governance and commercial/legal permission still need to be contractually confirmed during onboarding.
- Shoprite does not publicly expose a dedicated payment/remittance endpoint in the documented REST service list, so settlement data may still need another source.

### Pick n Pay

Pick n Pay's supplier portal is materially richer than the original hypothesis assumed.

Its published supplier documentation covers:
- Purchase orders
- Invoices
- Goods receipts / return notes
- Credit advices
- Statements
- Remittances
- Master data
- Cost prices
- Order / invoice reconciliation
- Sales and inventory reporting

Its EDI implementation material lists:
- Orders from Pick n Pay
- Invoices to Pick n Pay
- Goods receipts
- Goods returns
- Credit advice
- Statements
- Remittance advice

Pick n Pay supports:
- Portal use
- Direct supplier web-service integration
- EDI via a Value Added Network / service provider acting on behalf of the supplier

The Pick n Pay portal is powered by Endoxa.

Implication for VEYRIN:
- Pick n Pay may be the strongest first retailer for historical reconciliation because the required evidence chain is unusually complete.
- VEYRIN should not rebuild Pick n Pay EDI.
- It should ingest authorised exports or feeds from the existing portal/integration path and provide the interpretation/exception layer.

### SPAR South Africa

Public evidence shows an established electronic supplier environment.

Signals include:
- SPAR supplier portal registration
- historic XML/web-service supplier integration
- current roles referencing electronic and manual claims, Gateway claims, supplier training on the electronic claims system, GRVs, PODs, statements and remittances
- a South African SPAR Supplier Recon Manager product that uploads warehouse/drop-shipment statements, auto-allocates credits, reconciles transactions and tracks unreconciled items as claims

Implication:
- SPAR should be treated as a real later integration target, not as an API unknown.
- The likely value is cross-retailer normalisation and supplier-side claims visibility, not basic statement matching alone.

### Makro / Massmart

Evidence is mixed between traditional retail supply and marketplace workflows, but several useful signals exist:
- South African suppliers have implemented EDI with Makro.
- A 2026 DGB case study reports Pick n Pay, Shoprite and Makro live on EDI with purchase orders, invoices and claims automated.
- Makro Marketplace provides remittance detail, returns, claims workflows and seller reports.
- Third-party EDI providers advertise Makro document flows including purchase order, invoice, ASN and remittance advice, though partner-specific validation is still required.

Implication:
- Makro should remain in the roadmap.
- Do not assume marketplace fields equal traditional vendor account fields.
- Validate traditional Massmart/Makro supplier document scope directly with a real supplier before coding.

### Woolworths South Africa

Verified:
- A live Woolworths supplier portal exists.
- Woolworths finance roles reference EDI and invoice-entry systems.
- Public supplier material supports electronic supplier processes.

Still unverified publicly:
- exact supplier-side claim/remittance/GRV message set available to third-party integrations
- dispute workflow details and APIs

Implication:
- Keep Woolworths as a discovery target, but do not market a live Woolworths connector yet.

### Clicks / UPD

Clicks public job evidence confirms:
- reconciliation and payment staging
- invoice matching
- claims processing and clearing
- goods receipt verification
- claims disputes
- vendor queries
- high-volume FMCG/wholesale finance processes

Implication:
- The commercial problem exists in the Clicks ecosystem.
- Public integration evidence is insufficient to promise direct connectivity.
- Historical-file pilot first.

### Dis-Chem

Dis-Chem is important mainly as a warning against an over-broad thesis.

BEST SAP's case study shows Dis-Chem already automated large-scale vendor statement reconciliation inside SAP/BEST, while finance staff still focus on disputes and exceptions.

Implication:
- Generic reconciliation is not enough.
- VEYRIN must be stronger on retailer claims, supplier-side evidence, commercial-term interpretation and cross-retailer exception intelligence.
- "SAP cannot reconcile" is false positioning and must never appear in sales material.

---

## Competitive conclusion

### Direct global analogue

HighRadius is the closest full-function analogue:
- portal claim retrieval
- claim backup linking
- POD aggregation
- trade-promotion matching
- pricing variance analysis
- shortage analysis
- validity prediction
- dispute workflow
- ERP/TPM updates

SPS Revenue Recovery / SupplyPike also targets retailer deductions, supporting document gathering, dispute automation and root-cause analysis.

### ERP / local workflow competitors

SYSPRO Trade Promotions already:
- tracks promotion accruals
- processes deductions
- matches deductions to accrued promotions
- supports acceptance, rejection, write-off and reconciliation

Dynamics 365 provides a Deduction Workbench and trade-allowance matching.

BEST SAP proves highly automated SAP-native reconciliation is already possible.

Endoxa and Flowgear already solve retailer connectivity and B2B document movement.

### Therefore

VEYRIN must not claim:
- unique ability to integrate retailers
- unique ability to reconcile accounts
- replacement of SAP/SYSPRO/Dynamics
- unique use of AI for deduction work

VEYRIN can credibly target:
- South African retailer-specific commercial normalisation
- one supplier view across multiple retailer ecosystems
- combining retailer evidence with internal supplier evidence
- exception prioritisation
- dispute-pack preparation
- outcome tracking and root-cause learning
- mid-market implementation and pricing

---

## Product architecture decision

### Layer 1 - Ingestion

V1 supported inputs:
- CSV
- XLSX
- XML
- JSON
- PDF
- manually uploaded supporting documents

Later:
- authorised retailer API
- EDI/web service
- SFTP
- existing integration provider feed
- email ingestion
- retailer portal automation only where there is no cleaner authorised route

Rule:
Never make scraping the default integration strategy.

### Layer 2 - Canonical retail settlement model

Core entities:

1. SupplierAccount
- supplier id
- legal entity
- retailer
- retailer vendor id
- GLN
- currency

2. RetailerLocation
- retailer
- store/DC id
- GLN
- description

3. PurchaseOrder
- PO number
- order date
- retailer location
- SKU lines
- ordered quantity
- unit price
- promotion reference

4. Invoice
- invoice number
- invoice date
- PO reference
- SKU lines
- invoiced quantity
- net amount
- VAT
- gross amount

5. ReceiptEvent
- GRV / goods receipt / return note id
- PO/invoice reference
- location
- received quantity
- rejected/returned quantity
- reason
- event date

6. Settlement
- payment/remittance reference
- settlement date
- gross payable
- amount paid
- total deductions
- unallocated amount

7. DeductionClaim
- retailer claim id
- claim date
- reason code
- reason description
- invoice reference
- PO reference
- SKU
- location
- claimed quantity
- claimed amount
- VAT
- claim category

8. CommercialTerm
- agreement id
- retailer
- effective dates
- term type
- rate/value
- product/customer scope
- promotion id
- approval/source document

9. DeliveryEvidence
- POD
- ASN
- bill of lading
- delivery note
- carrier proof
- date/time
- quantity
- signatory/reference

10. CreditAdjustment
- credit note/debit note
- claim reference
- invoice reference
- amount
- date
- accounting status

11. ExceptionCase
- exception id
- classification
- confidence
- amount at risk
- evidence status
- owner
- priority
- next action
- due/dispute deadline

12. DisputeOutcome
- submitted date
- amount disputed
- outcome
- recovered amount
- rejection reason
- resolution date
- retailer response reference

13. SourceDocument
- source system
- file/message id
- original format
- checksum
- received date
- extraction status

14. AuditEvent
- user/system
- action
- old value
- new value
- timestamp

### Matching hierarchy

Use deterministic logic before AI.

1. exact identifiers
- invoice
- PO
- claim
- remittance
- credit note

2. line identifiers
- GTIN
- retailer item id
- supplier item id
- line number

3. commercial attributes
- quantity
- price
- VAT
- dates
- location
- promotion id

4. fuzzy/AI-assisted matching
Only where source references are incomplete or inconsistent.

Every automatic match must retain provenance and explain why the records were linked.

---

## V1 exception taxonomy

Use a small practical taxonomy:

- Supported deduction
- Pricing variance
- Quantity / shortage variance
- Return / damage
- Promotional allowance
- Rebate / trading term
- Settlement discount
- Duplicate deduction
- Duplicate credit
- Missing evidence
- Evidence conflict
- Out-of-period claim
- Unallocated payment
- Timing difference
- Unknown / human review

Do not start with dozens of retailer-specific categories. Map retailer codes into this canonical taxonomy while retaining the original retailer reason code.

---

## V1 user workflow

1. Upload/select settlement period
2. VEYRIN imports and normalises available records
3. VEYRIN links settlement lines to invoices, claims, receipts and terms
4. User receives an exception queue, not a giant reconciliation table
5. Open an exception to see:
   - what happened
   - amount at risk
   - linked evidence
   - missing evidence
   - why VEYRIN flagged it
6. User chooses:
   - accept
   - investigate
   - request evidence
   - dispute
   - write off
7. VEYRIN generates a structured action/dispute pack
8. User records/submits the outcome
9. Dashboard shows:
   - total deducted
   - explained
   - accepted
   - questionable
   - disputed
   - recovered
   - unresolved
   - recurring root causes

---

## What NOT to build in V1

- full EDI VAN
- bank integrations
- retailer portal replacement
- autonomous dispute submission
- universal AI validity prediction
- full trade-promotion management
- AP automation
- broad ERP replacement
- RPA bots for every retailer
- management dashboards before the underlying exception workflow works

---

## Recommended pilot

### Pilot objective

Prove that VEYRIN can reduce investigation effort and identify economically meaningful exceptions using existing data before live integrations.

### Best pilot shape

One supplier.
Two retailers if data allows:
- Pick n Pay
- Shoprite

Historical window:
- 3 months

Inputs:
- invoice register
- remittances / payment allocation files
- claims / credit advices
- goods receipts / returns where available
- credit notes
- trading terms
- promotion schedules
- PODs / delivery evidence for selected shortage cases
- current manual claims tracker

### Baseline measures

- number of deduction lines
- total value deducted
- analyst hours spent
- number of systems/portals/files touched
- unresolved value
- value written off
- dispute success rate where known
- average days to resolution

### VEYRIN output measures

- % settlement value explained
- % deductions automatically linked to evidence
- exceptions requiring human review
- potentially invalid/questionable value
- duplicates detected
- missing evidence cases
- historical recoverable value found
- analyst time per case
- actual recovery after dispute

### Pilot success gate

Do not define a fake recovery-rate target before seeing customer data.

Proceed to live integration only if:
- the customer confirms recurring pain,
- the historical review creates measurable time or cash value,
- required data can be obtained legally and operationally,
- willingness to pay is demonstrated.

---

## Security and trust requirements

Minimum:
- read-only source access wherever possible
- supplier-owned service/agent accounts
- least privilege
- no shared personal credentials
- encrypted secret storage
- encryption in transit and at rest
- customer-level data isolation
- full audit trail
- source-document provenance
- configurable retention
- export/delete capability
- role-based access
- approval gates before external disputes or ERP postings

High trust matters more than flashy AI.

---

## Go-to-market position

Primary user:
- Claims Clerk
- Debtors Clerk
- Credit Controller
- Key Account Finance
- AR / Credit Manager
- Commercial Finance

Primary buyer:
- Finance Manager
- Head of Credit / AR
- Financial Controller
- Commercial Finance lead

Target company profile:
- South African FMCG / CPG supplier
- sells to multiple national retailers
- meaningful monthly deduction volume
- finance team still uses Excel, portal downloads and ERP screens to investigate claims
- too small or locally specific for a HighRadius-style enterprise deployment
- existing ERP/EDI environment should remain in place

Positioning:
> Keep the systems that move the data. Use VEYRIN to understand the exceptions.

---

## Moat strategy

Weak moats:
- API connectors
- generic matching
- dashboards
- "AI"

Potential real moats:
1. retailer-specific reason-code and evidence maps
2. local dispute rules and documentary requirements
3. historical outcome data by retailer/reason/evidence pattern
4. supplier-specific commercial-term models
5. cross-retailer canonical data model
6. embedded finance workflow and audit history
7. integration partnerships that make deployment easier

The objective is to accumulate proprietary operational knowledge without making the customer dependent on opaque algorithms.

---

## Research backlog

Before full build:
1. Obtain real Pick n Pay remittance, credit advice and goods receipt samples from a consenting supplier.
2. Obtain a real Shoprite claim payload/export from a consenting supplier.
3. Confirm Shoprite commercial terms for third-party agent/service-account use.
4. Confirm Pick n Pay service-provider access mechanics and available message endpoints for remittance/credit advice.
5. Interview at least 5 supplier-side claims/credit professionals.
6. Validate SPAR Gateway data access from the supplier side.
7. Validate traditional Makro/Massmart vendor claim/remittance flow separately from Marketplace.
8. Validate Woolworths claim/remittance workflow.
9. Validate Clicks and Dis-Chem supplier-side data availability.
10. Price the problem only after volume and labour data are known.

---

## Current decision

VALIDATE AND PROTOTYPE THE RECONCILIATION/EXCEPTION WORKFLOW.

Do not build retailer integrations first.

The first product proof should accept existing files, normalise them into the canonical model, produce an explainable exception queue, and create an auditable case file for each material deduction.

If this is valuable using static historical data, live integration becomes an accelerator.

If it is not valuable using static historical data, integration will not rescue the product.
