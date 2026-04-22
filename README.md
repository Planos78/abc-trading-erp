# ABC Trading ERP System

ระบบ ERP สำหรับบริษัท ABC Trading ดำเนินธุรกิจค้าส่งและค้าปลีกสินค้าอุปโภคบริโภค  
ออกแบบให้รองรับ 3 ช่องทางขายในระบบเดียว พร้อม Business Rules ครบถ้วน

---

## เอกสารหลัก

| ไฟล์ | รายละเอียด |
|------|-----------|
| [docs/er-diagram.md](docs/er-diagram.md) | ER Diagram — 35 Entity ครบทุก Module |
| [Interactive ER Diagram](https://dbdiagram.io/d/69e88c921bbca033120d41e8) | dbdiagram.io — zoom/pan/คลิก relationship ได้ |
| [docs/business-workflow.md](docs/business-workflow.md) | Business Workflow — 15 กระบวนการ + Business Rules |
| [docs/schema.dbml](docs/schema.dbml) | DBML Schema — import dbdiagram.io ได้ทันที |

---

## Module ที่ออกแบบ

| Module | Entity หลัก | หมายเหตุ |
|--------|------------|---------|
| Customer | Customer, Address, Contact | รองรับ Retail + Wholesale + Credit Limit |
| Product & Pricing | Product, PriceList, Promotion | Price Tier A/B/C แยกตาม Channel |
| Inventory | Warehouse, Stock, LOT | LOT/Batch tracking + Expiry Date |
| Sales | Quotation → SO → Delivery → Invoice | ครบ 3 ช่องทาง |
| POS | Terminal, Session, Transaction | Open/Close Session + Cash Reconcile |
| Purchase | PO → GR → VendorBill | 3-Way Matching |
| AR/AP | Invoice, Payment, Allocation | AR Aging + Credit Block อัตโนมัติ |
| Return | ReturnOrder, CreditNote | Return Window + สภาพสินค้า |
| Auth & RBAC | User, Role, AuditLog | 7 Role + Audit Trail ทุก Action |
| Loyalty | LoyaltyPoint, Transaction | Earn/Redeem Points |
| Review | ProductReview | Gate: เฉพาะ Order ที่ Delivered |

---

## Key Design Decisions

### 1. SALES_CHANNEL เป็น Entity แยก ไม่ใช่ Enum
```
sales_order.channel_id → sales_channel.id
```
เพราะในอนาคตอาจเพิ่มช่องทางใหม่ (LINE Shopping, TikTok Shop) โดยไม่ต้อง migrate schema — แค่ insert row ใหม่

### 2. STOCK แยก 3 field
```
qty_on_hand = qty_reserved + qty_available
```
ป้องกัน Over-sell — เมื่อ Confirm SO ระบบ +qty_reserved ทันที สินค้าถูก lock ก่อนจัดส่ง ไม่รอให้ตัดจริง

### 3. PAYMENT_ALLOCATION เป็น Junction Table
```
payment → payment_allocation ← invoice
```
รองรับ 1 Payment จ่ายหลาย Invoice และ 1 Invoice รับจากหลาย Payment (partial payment) โดยไม่ต้อง hard-link

### 4. VENDOR_BILL link ทั้ง PO และ GR
```
vendor_bill.po_id + vendor_bill.gr_id
```
3-Way Matching ทำได้ใน query เดียว — ตรวจสอบ PO + GR + Bill ตรงกันก่อนจ่ายเงิน ลดความเสี่ยงจ่ายเกิน/จ่ายผิด

### 5. AUDIT_LOG เป็น Table แยก
```
audit_log(entity_type, entity_id, old_value json, new_value json)
```
ไม่ bloat ทุก table ด้วย updated_by/updated_at — ย้อนดูทุก state change ได้ ทั้ง compliance และตรวจสอบการโกง

---

## Tech Stack Recommendation

> โจทย์ยังไม่ได้กำหนด Stack — นี่คือ Recommendation สำหรับ Production

| Layer | เลือก | เหตุผล |
|-------|-------|--------|
| **Backend** | FastAPI (Python) | Type-safe, async, OpenAPI doc อัตโนมัติ, ecosystem ML พร้อม |
| **Frontend** | Next.js (React) | SSR สำหรับ E-commerce, CSR สำหรับ Back Office |
| **Database** | PostgreSQL | ACID, JSON support, Row-level Security สำหรับ RBAC |
| **Cache** | Redis | Stock reservation lock, Session, Rate limiting |
| **Queue** | Celery + Redis | Async notification (email/LINE/SMS), Report generation |
| **Auth** | JWT + Refresh Token | Stateless, รองรับ Multi-device |

### ทำไมไม่เลือก
- **MongoDB** — ERP มี Relationship ซับซ้อน ต้องการ ACID Transaction
- **Django** — Overhead สูงเกินสำหรับ API-first ERP
- **MySQL** — JSON support และ Window Function อ่อนกว่า PostgreSQL

---

## Business Rules สำคัญ

| Rule | กลไก |
|------|------|
| Credit Check | SO wholesale → เช็ค `credit_limit` vs ยอดค้าง AR → block หรือ escalate Manager |
| Stock Reserve | Confirm SO → `qty_reserved++`, `qty_available--` ทันที atomic |
| Price Tier | ดึง PriceListItem ตาม customer_type + channel + วันที่ valid |
| 3-Way Matching | `po.total ≈ gr.qty × cost ≈ vendor_bill.total` — tolerance ±1% |
| Return Window | `return_date - original_order.delivery_date ≤ policy_days` |
| AR Aging | Cron ทุกวัน → classify 0-30 / 31-60 / 61-90 / 90+ → auto notify + block |
| LOT/FIFO | จ่ายของ lot expiry_date น้อยที่สุดก่อน — ลด expired stock |

---

## Workflow ที่ออกแบบ (15 กระบวนการ)

1. Sales — POS
2. Sales — Sales Representative (Quotation → Credit Check → Delivery)
3. Sales — E-commerce (Payment Gateway → Auto Stock Reserve)
4. Inventory Management
5. Purchase / Procurement (3-Way Matching)
6. Return & Refund
7. Module Overview Diagram
8. AR Aging & Collection
9. Credit Block / Unblock
10. Stock Take / Physical Count
11. Product Review (Rating Gate)
12. Price List Management
13. Stock Transfer ระหว่างคลัง
14. Reporting & Dashboard
15. Business Rules Summary
