# Order Workflow State Machine - AFRILUX CCA Dashboard

## Complete Order Lifecycle

The order workflow consists of 7 business objects, each with its own state machine. These machines interact and trigger each other based on business rules.

## 1. PROFORMA State Machine

```
┌──────────────────────┐
│  PF-01               │
│  En attente          │  (Commercial receives requirement)
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  PF-02               │
│  En cours            │  (Commercial collects info)
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  PF-03               │
│  En signature        │  (Commercial + Approvers sign)
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  PF-04               │  (Signed proforma sent to CCA)
│  OK                  │  ──→ TRIGGERS: BC-01 (Bon en attente)
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  PF-FIN              │
│  ACHEVÉ              │  (Archived)
└──────────────────────┘

Responsible: COMMERCIAL (COMM role)
Alerts:
  - PF-01 without action > 7 days
  - PF-04 without BC transition > 5 days
```

## 2. PURCHASE ORDER (Client) State Machine

```
┌──────────────────────┐
│  BC-01               │
│  En attente          │  (Created from PF-04)
└──────────┬───────────┘
           │ (Facturation/Comptabilité validates)
           ▼
┌──────────────────────┐
│  BC-02               │  (Delivery order sent to warehouse)
│  En attente          │  ──→ TRIGGERS: LV-01 (Livraison en préparation)
│  de livraison        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  BC-FIN              │
│  ACHEVÉE             │  (Archived)
└──────────────────────┘

Responsible: FACTURATION (FACT role)
Alerts:
  - BC-01 without validation > 5 days
  - BC-02 without delivery start > 10 days
```

## 3. DELIVERY (Client) State Machine

```
┌──────────────────────┐
│  LV-01               │
│  En préparation      │  (Warehouse prepares items)
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  LV-02               │  (Warehouse assembles delivery)
│  En cours            │
│  d'assemblage        │
└──────────┬───────────┘
           │
           ├──→ Stock insufficient?
           │   ▼
           │  ┌──────────────────────┐
           │  │  LV-03               │  PURCHASE ORDER (ACH-01 triggered)
           │  │  Achat en            │
           │  │  attente             │
           │  └──────────────────────┘
           │
           ▼
    ┌──────────────────────────────────┐
    │  LV-04 or LV-05?                 │
    └──────────┬──────────────────────┘
             │
    ┌────────┴──────────┐
    │                   │
    ▼                   ▼
┌──────────────┐    ┌──────────────────┐
│  LV-04       │    │  LV-05           │
│  Partielle   │    │  Complète        │
│              │    │                  │  ──→ TRIGGERS: FC-01 (Facturation)
└────┬─────────┘    └──────┬───────────┘
     │ (Decision req.)     │
     └──────────┬──────────┘
               ▼
           ┌──────────────────┐
           │  LV-FIN          │
           │  ACHEVÉE         │  (Archived)
           └──────────────────┘

Responsible: MAGASIN (MAG role)
Alerts:
  - LV-01 without action > 10 days
  - LV-04 (partial) without decision > 30 days
```

## 4. PURCHASE ORDER (Supplier) State Machine

```
┌──────────────────────┐
│  ACH-01              │  (Triggered by stock shortage in LV-03)
│  Achat en            │
│  attente (DA)        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  ACH-02              │  (Procurement collects vendor quotes)
│  Proforma            │
│  fournisseur         │
│  en attente          │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  ACH-03              │  (Purchase order created)
│  Commande achat      │
│  en cours            │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  ACH-04              │  (Multi-level approval)
│  Commande achat      │
│  validée             │  ──→ TRIGGERS: RF-01 (Payment prep)
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  ACH-05              │  (Waiting for supplier delivery)
│  Livraison           │
│  fournisseur         │
│  en attente          │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  ACH-FIN             │  (Archived)
│  ACHEVÉE             │
└──────────────────────┘

Responsible: ACHATS (ACH role)
Alerts:
  - ACH-01 without treatment > 3 days
  - ACH-03 without validation > 5 days
  - ACH-05 delivery late vs. expected date
```

## 5. SUPPLIER PAYMENT State Machine

```
┌─────────────────────────┐
│  RF-01                  │
│  En cours de            │  (Triggered by ACH-04 validation)
│  règlement              │
└─────────────┬───────────┘
              │
              ▼
┌─────────────────────────┐
│  RF-02                  │  (Payment created: cash/check/transfer)
│  Paiement initié        │
└─────────────┬───────────┘
              │
              ▼
┌─────────────────────────┐
│  RF-03                  │  (Multi-level approval)
│  Paiement validé        │
└─────────────┬───────────┘
              │
              ▼
┌─────────────────────────┐
│  RF-04                  │  (Deposited with bank/cashier)
│  Paiement effectué      │
│  (banque/caisse)        │
└─────────────┬───────────┘
              │
              ▼
┌─────────────────────────┐
│  RF-FIN                 │  (Archived)
│  ACHEVÉ                 │
└─────────────────────────┘

Responsible: COMPTABILITÉ (COMPT role)
Alerts:
  - RF-02 without validation > 3 days (blocking)
  - RF-04 > agreed payment terms without processing
```

## 6. INVOICING & CLIENT PAYMENT State Machine

```
┌──────────────────────┐
│  FC-01               │  (Triggered by LV-05 or LV-04)
│  Facturation en      │
│  attente             │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  FC-02               │  (Invoice created and validated)
│  Facturation         │
│  achevée             │  ──→ TRIGGERS: FSC-02 (Tax request)
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  FC-03               │  (Invoice deposited with CCA)
│  Facture déposée     │
└──────────┬───────────┘
           │
    ┌──────┴──────────────┐
    │                     │
    ▼                     ▼
┌──────────────┐    ┌───────────────────┐
│ FC-04        │    │ FC-05             │  ╔═══════════════════════════════╗
│ Avance       │    │ Paiement          │  ║ Complete payment             ║
│ reçue        │    │ achevé            │  ║ received & matched           ║
│              │    │ (lettrage)        │  ║ (FC-05) →                    ║
└──┬───────────┘    └────────┬──────────┘  ║ TRIGGERS FSC-03              ║
  │ (relance                 │            ║ + FC-FIN                     ║
  │  pour solde)             │            ╚═══════════════════════════════╝
  └──────────┬───────────────┘
             ▼
       ┌──────────────────┐
       │  FC-FIN          │  (Archived)
       │  ACHEVÉE         │
       │  AVEC SUCCÈS     │
       └──────────────────┘

Responsible: FACTURATION (FACT role) + RECOUVREMENT (RECOV role)
Alerts:
  - FC-02 not deposited > 3 days
  - FC-03 without payment > 30/60/90 days (auto-relance)
  - FC-04 (advance) without final payment > 30 days
```

## 7. FISCAL DECLARATION State Machine

```
┌──────────────────────┐
│  FSC-01              │  (Triggered by BC-FIN or FC-02)
│  Commande à          │
│  déclarer            │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  FSC-02              │  (Request withholding certificate)
│  Attestation en      │
│  attente             │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  FSC-03              │  (Tax declaration submitted)
│  Commande            │
│  déclarée            │  ──→ FSC-03 + FC-05 both OK = Order closed
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  FSC-FIN             │  (Archived)
│  ACHEVÉE             │
└──────────────────────┘

Responsible: FISCALITÉ (FISC role)
Alerts:
  - FSC-01 not declared > 5 days
  - FSC-02 certificate not obtained > 30 days
```

## Automatic Transitions (System)

| From → To | Trigger | System Action |
|-----------|---------|---------------|
| PF-04 | Validation by Comptabilité | Auto-create BC-01 |
| BC-02 | Status update | Auto-notify MAG, create LV-01 |
| LV-03 (Stock check) | Availability check | Auto-create ACH-01 |
| ACH-04 | Approver validation | Auto-create RF-01 |
| LV-05 | Completed delivery | Auto-create FC-01 |
| BC-FIN | Delivery completion | Auto-create FSC-01 |
| FC-02 | Invoice validation | Auto-trigger FSC-02 |
| FC-05 + FSC-03 | Both completed | Auto-trigger FC-FIN, order closed |

## Manual Transitions (User Action)

| Transition | Required Role | Condition |
|-----------|--------------|-----------|
| PF-01 → PF-02 | COMM | Requirement received |
| PF-02 → PF-03 | COMM | Information collected |
| PF-03 → PF-04 | COMM, Approver | All signatures obtained |
| BC-01 → BC-02 | FACT, COMPT | Price agreed, BC validated |
| LV-01 → LV-02 | MAG | Items available, prep starts |
| LV-02 → LV-04/LV-05 | MAG | Delivery ready, complete/partial? |
| ACH-02 → ACH-03 | ACH | Vendor quote received, PO created |
| ACH-03 → ACH-04 | Approver | Multi-level validation |
| ACH-05 → ACH-FIN | MAG | Supplier delivery received |
| RF-02 → RF-03 | Approver, ADMIN | Payment validated |
| RF-03 → RF-04 | COMPT | Payment deposited |
| FC-01 → FC-02 | FACT | Invoice created |
| FC-02 → FC-03 | FACT, RECOV | Invoice deposited with CCA |
| FC-03 → FC-04 | RECOV | Advance received from CCA |
| FC-03 → FC-05 | COMPT | Full payment received |
| FSC-02 → FSC-03 | FISC | Tax declaration submitted |

## Timeline Summary (Ideal Case)

```
Day 1:  Expression of need → PF-01
Day 3:  Proforma signed → PF-04 → BC-01
Day 5:  BC validated → BC-02 → LV-01
Day 10: Delivery complete → LV-05 → FC-01
Day 12: Invoice issued → FC-02 → FSC-02
Day 20: Payment received → FC-05 + FSC-03
Day 21: Order closed → FC-FIN, FSC-FIN

Total cycle: ~3 weeks (Target: <45 days)
```

## Blocking Rules & Decisions

### Partial Delivery (LV-04) Decision Required
When delivery is partial, no automatic closure occurs. Recouvrement + DG must decide:
1. **Wait for remainder**: Keep order open, add to stock requisition
2. **Invoice partial**: Create FC-01 for delivered items only
3. **Cancel remainder**: Close order, document cancellation reason

### Multi-Level Approvals
For orders > threshold amounts:
- **ACH-04** (Supplier PO): Requires Approver + Manager sign-off
- **RF-03** (Supplier Payment): Requires Approver + ADMIN sign-off  
- Orders > 500k FCFA also require DG notification

### Blocking Conditions
1. Cannot create BC-02 if BC-01 not validated
2. Cannot complete LV-05 if LV-03 (stock shortage) unresolved
3. Cannot issue FC-02 if delivery incomplete
4. Cannot close order if FC-05 + FSC-03 not both complete
5. Partial delivery > 30 days requires escalation to DG
