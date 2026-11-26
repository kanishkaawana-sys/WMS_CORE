# WMS Core — Warehouse Management System

A full-stack Warehouse Management System built with **React**, **Node.js**, **Express**, and **PostgreSQL** (`warehouse_db`). Designed around strict database-level integrity, `wms_core` prevents overselling under concurrent order load through row-level locking, automates low-stock detection via PostgreSQL triggers, and preserves an immutable audit trail of all inventory movements.

---

## Core Architecture & Features

- **ACID Transactions & Concurrency Control:** Wraps stock allocation and order fulfillment in explicit PostgreSQL transactions (`BEGIN` / `COMMIT` / `ROLLBACK`) with row-level locks (`SELECT ... FOR UPDATE`), preventing race conditions and negative stock balances when multiple requests target the same SKU simultaneously.
- **Automated Reorder Triggers:** Executes the `trg_inventory_reorder_check` PostgreSQL trigger whenever inventory quantities change, automatically flagging items that drop below their configured reorder threshold.
- **Immutable Transaction Ledger:** Logs every inbound receipt, outbound fulfillment, and stock adjustment in an append-only ledger table rather than mutating historical records in place.
- **Raw SQL Backend Layer:** Interacts directly with PostgreSQL using parameterized SQL queries and schema-enforced constraints (`CHECK`, `FOREIGN KEY`, `UNIQUE`) without ORM overhead.

---

## Tech Stack

- **Frontend:** React
- **Backend:** Node.js, Express, CORS, dotenv
- **Database:** PostgreSQL (`warehouse_db`)

---

## Repository Structure

```text
wms_core/
├── warehouse-backend/
│   ├── init.sql        # PostgreSQL tables, constraints, triggers, and initial schema
│   ├── .env            # Database connection string and server port configuration
│   └── package.json    # Backend dependencies and runtime scripts
└── README.md
```

---

## Local Setup & Installation

### 1. Clone the Repository
```bash
git clone https://github.com/apurvamarya/wms_core.git
cd wms_core/warehouse-backend
```

### 2. Initialize the PostgreSQL Database
Ensure PostgreSQL is running locally, then create the database and load the schema and triggers:
```bash
createdb warehouse_db
psql -d warehouse_db -f init.sql
```

### 3. Configure Environment Variables
Create a `.env` file inside `warehouse-backend/` with your local PostgreSQL credentials:
```env
PORT=5000
DATABASE_URL=postgresql://localhost:5432/warehouse_db
```

### 4. Install Dependencies & Start the Server
```bash
npm install
npm start
```

---

## Data Integrity Guarantees

| Mechanism | PostgreSQL Implementation | Purpose |
| :--- | :--- | :--- |
| **Pessimistic Locking** | `SELECT ... FOR UPDATE` inside transactions | Serializes concurrent writes on active inventory rows during order fulfillment. |
| **Threshold Monitoring** | `trg_inventory_reorder_check` trigger | Evaluates post-update stock quantities and triggers low-stock alerts automatically. |
| **Audit Trail** | Append-only ledger table | Guarantees non-repudiable history of all stock inflows, outflows, and adjustments. |

---
