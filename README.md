# AFRILUX CCA Order Tracking Dashboard

🚀 Unified order management system for CCA Bank Holding orders - from proforma to settlement.

## 📋 Overview

This application provides role-based dashboards for 8 different user types to track orders through a 31-step workflow spanning:
- Proforma generation
- Purchase order management
- Delivery tracking
- Invoicing
- Supplier payments
- Client collections
- Fiscal declarations

## 🛠️ Technology Stack

- **Frontend**: React 18 + TypeScript
- **Backend**: Node.js + Express.js
- **Database**: PostgreSQL
- **Authentication**: Active Directory (LDAP)
- **Real-time**: WebSocket
- **Deployment**: Docker + Docker Compose

## 📁 Project Structure

```
afrilux-cca-dashboard/
├── backend/                 # Node.js/Express API
│   ├── src/
│   │   ├── controllers/    # Request handlers
│   │   ├── models/         # Database models
│   │   ├── routes/         # API endpoints
│   │   ├── middleware/     # Auth, logging
│   │   ├── services/       # Business logic
│   │   ├── config/         # Configuration
│   │   └── server.ts       # Entry point
│   ├── migrations/         # Database migrations
│   ├── package.json
│   └── Dockerfile
├── frontend/                # React application
│   ├── src/
│   │   ├── components/     # UI components
│   │   ├── dashboards/     # Role-specific dashboards
│   │   ├── store/          # Redux state
│   │   ├── services/       # API clients
│   │   └── App.tsx
│   ├── package.json
│   └── Dockerfile
├── docs/                    # Documentation
├── docker-compose.yml
├── .env.example
└── CDC-2026-001.pdf        # Requirements
```

## 🚀 Quick Start

### Prerequisites
- Docker & Docker Compose
- Node.js 18+ (for local development)

### Development Setup

```bash
# Clone repository
git clone https://github.com/ghost3001-ai/afrilux-cca-dashboard.git
cd afrilux-cca-dashboard

# Copy environment file
cp .env.example .env

# Start with Docker Compose
docker-compose up -d

# Run database migrations
docker exec afrilux-backend npm run migrate

# Access application
# Frontend: http://localhost:3000
# API: http://localhost:5000
# Database UI: http://localhost:8080
```

## 👥 Role-Based Dashboards

- **Commercial** (COMM): Proforma tracking & BC transformation
- **Accounting/Billing** (FACT): Invoice and payment validation
- **Warehouse** (MAG): Delivery preparation & fulfillment
- **Procurement** (ACH): Supply requisitions & vendor orders
- **Collections** (RECOV): Customer payment tracking
- **Fiscal** (FISC): Tax declaration management
- **Direction** (DG): Consolidated KPI view
- **Admin** (ADMIN): System configuration & audit

## 📊 Key Metrics (KPI)

| Metric | Target | Frequency |
|--------|--------|-----------|
| Proforma → BC cycle | <5 days | Weekly |
| BC → Delivery cycle | <15 days | Weekly |
| First-time complete delivery | ≥90% | Monthly |
| Payment collection <30d | ≥80% | Monthly |
| Overall cycle time | <45 days | Monthly |
| Successful completion | ≥95% | Monthly |

## 🔄 Complete Order Workflow

```
Expression de besoin (Day 1)
    ↓
Proforma EN ATTENTE → EN COURS → EN SIGNATURE → OK (Days 1-5)
    ↓
Bon de commande EN ATTENTE → EN ATTENTE DE LIVRAISON (Days 5-7)
    ↓
Livraison EN PRÉPARATION → EN COURS D'ASSEMBLAGE → COMPLÈTE/PARTIELLE (Days 7-10)
    ↓
Facturation EN ATTENTE → ACHEVÉE → DÉPOSÉE (Days 10-12)
    ↓
Recouvrement AVANCE → PAIEMENT ACHEVÉ (Days 12-30)
    ↓
Fiscalité À DÉCLARER → ATTESTATION → DÉCLARÉE (Days 12-30)
    ↓
Commande CLÔTURÉE (Day 21-45)
```

## 🔐 Security Features

- Role-based access control (RBAC)
- Immutable audit trail
- Active Directory SSO integration
- HTTPS/TLS encryption
- Session management (4-hour timeout)
- GDPR-compliant data handling

## 📈 Features

✅ Multi-role dashboards
✅ Automated workflow with 31 status transitions
✅ Real-time WebSocket updates
✅ In-app + email notifications
✅ Configurable alerts (7/5/30/60/90-day escalations)
✅ Stock insufficiency auto-detection
✅ Partial delivery handling
✅ Supplier/Client payment tracking
✅ Tax declaration management
✅ Complete audit trail
✅ Excel/PDF exports
✅ Active Directory integration

## 🧪 Testing

```bash
# Backend tests
docker exec afrilux-backend npm test

# Frontend tests
docker exec afrilux-frontend npm test

# Coverage
docker exec afrilux-backend npm run test:coverage
```

## 📚 API Documentation

Available at `http://localhost:5000/api/docs`

### Core Endpoints

**Orders**
- `GET /api/orders` - List orders
- `POST /api/orders` - Create order
- `GET /api/orders/:id` - Get details
- `PUT /api/orders/:id/status` - Update status
- `GET /api/orders/:id/audit` - Get audit trail

**Dashboards**
- `GET /api/dashboards/:role` - Get dashboard
- `GET /api/dashboards/:role/kpis` - Get KPIs
- `GET /api/dashboards/:role/alerts` - Get alerts

**Notifications**
- `GET /api/notifications` - List notifications
- `PUT /api/notifications/:id/read` - Mark read
- `DELETE /api/notifications/:id` - Delete

**Admin**
- `GET /api/admin/users` - List users
- `POST /api/admin/users` - Create user
- `GET /api/admin/audit-logs` - Audit logs
- `POST /api/admin/config` - Update config

## 📋 Development Roadmap

### Phase 1-2: Foundation ✅
- [x] Project scaffolding
- [ ] Database schema
- [ ] Authentication setup
- [ ] Basic API structure

### Phase 3-5: Core Features
- [ ] Order CRUD operations
- [ ] Status workflow engine
- [ ] Role-based dashboards
- [ ] Audit logging
- [ ] Notifications & alerts
- [ ] Real-time updates
- [ ] KPI calculations

### Phase 6-8: Integration & Deployment
- [ ] Active Directory SSO
- [ ] Sage 100 Cloud integration
- [ ] Docker containerization
- [ ] Production deployment
- [ ] User documentation

## 📞 Support

Contact: CHIEJE TCHATCHOUANG Yves (Chef Service Informatique)
Email: yves.chieje@afriluxsmart.cm

## 📄 License

CONFIDENTIEL — AFRILUX SMART SOLUTIONS SA — 2026

## 🔗 Documentation

- [Database Schema](./docs/DATABASE.md)
- [Workflow State Machines](./docs/WORKFLOW.md)
- [API Documentation](./docs/API.md)
- [Deployment Guide](./docs/DEPLOYMENT.md)
