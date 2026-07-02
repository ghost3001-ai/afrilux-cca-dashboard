# Database Schema - AFRILUX CCA Dashboard

## Overview

PostgreSQL-based relational database supporting the complete order lifecycle with audit trails and status tracking.

## Core Tables

### 1. `users` (Utilisateurs)
User accounts linked to Active Directory.

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username VARCHAR(100) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  ldap_dn VARCHAR(500),
  department_id UUID REFERENCES departments(id),
  is_active BOOLEAN DEFAULT true,
  is_admin BOOLEAN DEFAULT false,
  last_login TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);
```

### 2. `roles` (Rôles)
Role definitions for RBAC.

```sql
CREATE TABLE roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code VARCHAR(20) UNIQUE NOT NULL, -- COMM, FACT, MAG, ACH, COMPT, RECOV, FISC, DG, ADMIN
  name VARCHAR(100) NOT NULL,
  description TEXT,
  access_level VARCHAR(50), -- LECTURE, LECTURE+SAISIE, ADMIN TOTAL
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3. `user_roles` (Utilisateurs-Rôles)
Association between users and roles.

```sql
CREATE TABLE user_roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role_id UUID NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
  assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  assigned_by_id UUID REFERENCES users(id),
  UNIQUE(user_id, role_id)
);

CREATE INDEX idx_user_roles_user_id ON user_roles(user_id);
CREATE INDEX idx_user_roles_role_id ON user_roles(role_id);
```

### 4. `orders` (Commandes CCA)
Main order entity representing each CCA order.

```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_number VARCHAR(50) UNIQUE NOT NULL,
  client_id VARCHAR(50) NOT NULL DEFAULT 'CCA',
  description TEXT NOT NULL,
  amount_ht DECIMAL(15, 2) NOT NULL,
  amount_ttc DECIMAL(15, 2),
  currency VARCHAR(3) DEFAULT 'XAF',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by_id UUID NOT NULL REFERENCES users(id),
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_by_id UUID REFERENCES users(id),
  deleted_at TIMESTAMP,
  is_active BOOLEAN DEFAULT true,
  metadata JSONB DEFAULT '{}'
);

CREATE INDEX idx_orders_order_number ON orders(order_number);
CREATE INDEX idx_orders_created_at ON orders(created_at);
CREATE INDEX idx_orders_client_id ON orders(client_id);
```

### 5. `order_statuses` (Statuts des Objets)
Tracking of status history for each order object.

```sql
CREATE TABLE order_statuses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  object_type VARCHAR(50) NOT NULL, -- PROFORMA, BC_CLIENT, LIVRAISON_CLIENT, etc.
  status_code VARCHAR(20) NOT NULL, -- PF-01, BC-02, etc.
  status_name VARCHAR(255) NOT NULL,
  description TEXT,
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  changed_by_id UUID NOT NULL REFERENCES users(id),
  change_reason TEXT,
  metadata JSONB DEFAULT '{}',
  is_current BOOLEAN DEFAULT true
);

CREATE INDEX idx_order_statuses_order_id ON order_statuses(order_id);
CREATE INDEX idx_order_statuses_status_code ON order_statuses(status_code);
CREATE INDEX idx_order_statuses_changed_at ON order_statuses(changed_at);
```

### 6. `audit_logs` (Journalisation)
Immutable audit trail for all actions.

```sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID REFERENCES orders(id) ON DELETE SET NULL,
  user_id UUID NOT NULL REFERENCES users(id),
  action VARCHAR(100) NOT NULL, -- STATUS_CHANGE, COMMENT_ADDED, EXPORT, etc.
  object_type VARCHAR(50), -- ORDER, PROFORMA, BC, etc.
  object_id UUID,
  old_value JSONB,
  new_value JSONB,
  change_reason TEXT,
  ip_address INET,
  user_agent TEXT,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_logs_order_id ON audit_logs(order_id);
CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_timestamp ON audit_logs(timestamp);
```

### 7. `proformas` (Proformas)
Proforma documents.

```sql
CREATE TABLE proformas (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  proforma_number VARCHAR(50) UNIQUE NOT NULL,
  amount_ht DECIMAL(15, 2) NOT NULL,
  amount_ttc DECIMAL(15, 2),
  vat_amount DECIMAL(15, 2),
  issued_at TIMESTAMP,
  issued_by_id UUID REFERENCES users(id),
  target_issue_date DATE,
  signed_at TIMESTAMP,
  signed_by_id UUID REFERENCES users(id),
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_proformas_order_id ON proformas(order_id);
CREATE INDEX idx_proformas_proforma_number ON proformas(proforma_number);
```

### 8. `purchase_orders` (Bons de Commande)
Purchase orders for both client and supplier.

```sql
CREATE TABLE purchase_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  po_number VARCHAR(50) UNIQUE NOT NULL,
  po_type VARCHAR(20) NOT NULL, -- CLIENT, SUPPLIER
  supplier_id UUID REFERENCES suppliers(id),
  amount_ht DECIMAL(15, 2) NOT NULL,
  amount_ttc DECIMAL(15, 2),
  issued_at TIMESTAMP,
  issued_by_id UUID REFERENCES users(id),
  validated_at TIMESTAMP,
  validated_by_id UUID REFERENCES users(id),
  expected_delivery_date DATE,
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_purchase_orders_order_id ON purchase_orders(order_id);
CREATE INDEX idx_purchase_orders_po_number ON purchase_orders(po_number);
```

### 9. `deliveries` (Livraisons)
Delivery tracking for client and supplier.

```sql
CREATE TABLE deliveries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  delivery_number VARCHAR(50) UNIQUE NOT NULL,
  delivery_type VARCHAR(20) NOT NULL, -- CLIENT, SUPPLIER
  planned_date DATE,
  actual_date DATE,
  quantity_expected INT,
  quantity_delivered INT,
  is_partial BOOLEAN DEFAULT false,
  prepared_by_id UUID REFERENCES users(id),
  delivered_by_id UUID REFERENCES users(id),
  received_by_id UUID REFERENCES users(id),
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_deliveries_order_id ON deliveries(order_id);
CREATE INDEX idx_deliveries_delivery_number ON deliveries(delivery_number);
```

### 10. `invoices` (Factures)
Client and supplier invoices.

```sql
CREATE TABLE invoices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  invoice_number VARCHAR(50) UNIQUE NOT NULL,
  invoice_type VARCHAR(20) NOT NULL, -- CLIENT, SUPPLIER
  amount_ht DECIMAL(15, 2) NOT NULL,
  amount_vat DECIMAL(15, 2),
  amount_ttc DECIMAL(15, 2),
  issued_at TIMESTAMP,
  issued_by_id UUID REFERENCES users(id),
  due_date DATE,
  is_partial BOOLEAN DEFAULT false,
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_invoices_order_id ON invoices(order_id);
CREATE INDEX idx_invoices_invoice_number ON invoices(invoice_number);
```

### 11. `payments` (Paiements)
Payment tracking for client and supplier.

```sql
CREATE TABLE payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID REFERENCES orders(id) ON DELETE CASCADE,
  invoice_id UUID REFERENCES invoices(id) ON DELETE CASCADE,
  payment_type VARCHAR(20) NOT NULL, -- CLIENT, SUPPLIER, ADVANCE
  amount DECIMAL(15, 2) NOT NULL,
  method VARCHAR(50), -- CASH, CHECK, BANK_TRANSFER, MOBILE_MONEY
  status VARCHAR(50), -- INITIATED, VALIDATED, COMPLETED
  initiated_at TIMESTAMP,
  initiated_by_id UUID REFERENCES users(id),
  validated_at TIMESTAMP,
  validated_by_id UUID REFERENCES users(id),
  completed_at TIMESTAMP,
  reference_number VARCHAR(100),
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_payments_order_id ON payments(order_id);
CREATE INDEX idx_payments_invoice_id ON payments(invoice_id);
CREATE INDEX idx_payments_status ON payments(status);
```

### 12. `fiscal_declarations` (Déclarations Fiscales)
Tax declarations per order.

```sql
CREATE TABLE fiscal_declarations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  declaration_number VARCHAR(50) UNIQUE,
  amount_ht DECIMAL(15, 2) NOT NULL,
  amount_vat DECIMAL(15, 2),
  declared_at TIMESTAMP,
  declared_by_id UUID REFERENCES users(id),
  withholding_certificate_requested BOOLEAN DEFAULT false,
  withholding_certificate_received BOOLEAN DEFAULT false,
  certificate_number VARCHAR(100),
  certificate_received_at TIMESTAMP,
  certificate_amount DECIMAL(15, 2),
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_fiscal_declarations_order_id ON fiscal_declarations(order_id);
```

### 13. `notifications` (Notifications)
In-app notifications.

```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  order_id UUID REFERENCES orders(id) ON DELETE CASCADE,
  type VARCHAR(50) NOT NULL, -- STATUS_CHANGE, ALERT, ACTION_REQUIRED
  title VARCHAR(255) NOT NULL,
  message TEXT NOT NULL,
  is_read BOOLEAN DEFAULT false,
  read_at TIMESTAMP,
  action_url VARCHAR(500),
  severity VARCHAR(20), -- INFO, WARNING, ERROR, CRITICAL
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_order_id ON notifications(order_id);
CREATE INDEX idx_notifications_is_read ON notifications(is_read);
```

### 14. `departments` (Départements)
Department/service reference.

```sql
CREATE TABLE departments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code VARCHAR(20) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  manager_id UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 15. `suppliers` (Fournisseurs)
Supplier reference data.

```sql
CREATE TABLE suppliers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  supplier_code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255),
  phone VARCHAR(20),
  address TEXT,
  payment_terms VARCHAR(100),
  is_active BOOLEAN DEFAULT true,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Views

### `v_active_orders`
View of currently active orders with their current status.

```sql
CREATE VIEW v_active_orders AS
SELECT 
  o.id,
  o.order_number,
  o.description,
  o.amount_ht,
  o.created_at,
  u.first_name || ' ' || u.last_name as created_by,
  os.object_type,
  os.status_code,
  os.status_name,
  os.changed_at as status_changed_at,
  EXTRACT(DAY FROM CURRENT_TIMESTAMP - os.changed_at) as days_in_status
FROM orders o
JOIN users u ON o.created_by_id = u.id
JOIN order_statuses os ON o.id = os.order_id
WHERE o.is_active = true
  AND os.is_current = true
  AND os.status_code NOT LIKE '%FIN'
ORDER BY o.created_at DESC;
```

## Data Retention Policy

- Active orders: No deletion (soft delete with `deleted_at`)
- Audit logs: Retained indefinitely
- Notifications: Archived after 90 days
- Temporary records: Cleaned up monthly

## Backup Strategy

- Daily automated backups
- 30-day retention
- Point-in-time recovery capability
- Encrypted backup storage
