# Diagrama Entidade-Relacionamento (ER)

## Diagrama ER do Sistema de Controle de Estoque e PDV

```mermaid
erDiagram
    users {
        uuid id PK
        varchar name
        varchar email UK
        varchar password_hash
        varchar role
        boolean is_active
        boolean first_login
        timestamp last_login
        timestamp created_at
        timestamp updated_at
    }

    categories {
        uuid id PK
        varchar name UK
        text description
        boolean is_active
        timestamp created_at
        timestamp updated_at
    }

    suppliers {
        uuid id PK
        varchar name
        varchar document UK
        varchar email
        varchar phone
        text address
        boolean is_active
        timestamp created_at
        timestamp updated_at
    }

    products {
        uuid id PK
        varchar name
        text description
        uuid category_id FK
        uuid supplier_id FK
        varchar barcode UK
        varchar sku UK
        decimal cost_price
        decimal sale_price
        integer current_stock
        integer min_stock
        integer max_stock
        varchar unit
        text image_url
        boolean is_active
        timestamp created_at
        timestamp updated_at
    }

    customers {
        uuid id PK
        varchar name
        varchar document UK
        varchar email
        varchar phone
        text address
        date birth_date
        integer loyalty_points
        boolean is_active
        timestamp created_at
        timestamp updated_at
    }

    cash_registers {
        uuid id PK
        uuid user_id FK
        decimal opening_amount
        decimal closing_amount
        decimal expected_amount
        decimal difference
        timestamp opened_at
        timestamp closed_at
        varchar status
    }

    sales {
        uuid id PK
        varchar sale_number UK
        uuid user_id FK
        uuid customer_id FK
        uuid cash_register_id FK
        decimal subtotal
        decimal discount_amount
        decimal discount_percentage
        decimal total_amount
        varchar payment_method
        integer installments
        varchar status
        text notes
        timestamp created_at
        timestamp updated_at
    }

    sale_items {
        uuid id PK
        uuid sale_id FK
        uuid product_id FK
        integer quantity
        decimal unit_price
        decimal total_price
        decimal discount_amount
        timestamp created_at
    }

    stock_movements {
        uuid id PK
        uuid product_id FK
        uuid user_id FK
        varchar movement_type
        integer quantity
        integer previous_stock
        integer new_stock
        decimal unit_cost
        uuid reference_id
        varchar reference_type
        text reason
        timestamp created_at
    }

    price_history {
        uuid id PK
        uuid product_id FK
        uuid user_id FK
        decimal old_cost_price
        decimal new_cost_price
        decimal old_sale_price
        decimal new_sale_price
        text reason
        timestamp created_at
    }

    inventory_sessions {
        uuid id PK
        uuid user_id FK
        varchar name
        varchar status
        timestamp started_at
        timestamp completed_at
    }

    inventory_items {
        uuid id PK
        uuid inventory_session_id FK
        uuid product_id FK
        integer system_stock
        integer counted_stock
        integer difference
        text notes
        timestamp counted_at
        timestamp created_at
    }

    sync_queue {
        uuid id PK
        varchar table_name
        uuid record_id
        varchar operation
        jsonb data
        varchar status
        text error_message
        integer attempts
        timestamp created_at
        timestamp synced_at
    }

    user_sessions {
        uuid id PK
        uuid user_id FK
        varchar token UK
        timestamp expires_at
        timestamp created_at
        timestamp last_activity
    }

    audit_logs {
        uuid id PK
        uuid user_id FK
        varchar action
        varchar table_name
        uuid record_id
        jsonb old_values
        jsonb new_values
        inet ip_address
        text user_agent
        timestamp created_at
    }

    %% Relacionamentos
    products ||--o{ categories : "belongs to"
    products ||--o{ suppliers : "supplied by"
    
    sales ||--|| users : "sold by"
    sales ||--o{ customers : "sold to"
    sales ||--o{ cash_registers : "registered in"
    
    sale_items ||--|| sales : "belongs to"
    sale_items ||--|| products : "contains"
    
    stock_movements ||--|| products : "affects"
    stock_movements ||--|| users : "performed by"
    
    price_history ||--|| products : "tracks"
    price_history ||--|| users : "changed by"
    
    cash_registers ||--|| users : "operated by"
    
    inventory_sessions ||--|| users : "created by"
    inventory_items ||--|| inventory_sessions : "belongs to"
    inventory_items ||--|| products : "counts"
    
    user_sessions ||--|| users : "belongs to"
    audit_logs ||--o{ users : "performed by"
```

## Índices Principais por Tabela

### Índices de Performance
- **users**: email, role
- **products**: name, barcode, sku, category_id, current_stock
- **sales**: sale_number, user_id, customer_id, created_at, status
- **sale_items**: sale_id, product_id
- **stock_movements**: product_id, user_id, movement_type, created_at
- **cash_registers**: user_id, status, opened_at

### Índices de Busca
- **customers**: name, document, phone
- **suppliers**: name, document
- **categories**: name
- **sync_queue**: status, table_name, created_at
- **audit_logs**: user_id, action, table_name, created_at

## Constraints e Validações

### Check Constraints
- **users.role**: IN ('admin', 'manager', 'operator')
- **sales.payment_method**: IN ('cash', 'card', 'pix', 'credit')
- **sales.status**: IN ('pending', 'completed', 'cancelled')
- **stock_movements.movement_type**: IN ('in', 'out', 'adjustment', 'inventory')
- **cash_registers.status**: IN ('open', 'closed')
- **inventory_sessions.status**: IN ('active', 'completed', 'cancelled')
- **sync_queue.operation**: IN ('INSERT', 'UPDATE', 'DELETE')
- **sync_queue.status**: IN ('pending', 'synced', 'error')
- **sale_items.quantity**: > 0

### Foreign Key Constraints
- Todas as referências FK com ON DELETE apropriado
- CASCADE para itens dependentes (sale_items, inventory_items)
- RESTRICT para entidades principais (products, users)

### Unique Constraints
- **users**: email
- **products**: barcode, sku
- **customers**: document
- **suppliers**: document
- **categories**: name
- **sales**: sale_number
- **user_sessions**: token

## Triggers Automáticos

### Update Timestamps
- Trigger `update_updated_at_column()` em todas as tabelas com campo `updated_at`

### Auditoria Automática
- Logs automáticos de alterações em tabelas críticas
- Registro de IP e User-Agent para rastreabilidade

### Validações de Negócio
- Validação de estoque antes de vendas
- Atualização automática de estoque após vendas
- Cálculo automático de diferenças em inventários
