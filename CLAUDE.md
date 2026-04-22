# ABC Trading ERP — Project Context

## โปรเจกต์นี้คืออะไร
ระบบ ERP สำหรับบริษัท ABC Trading ดำเนินธุรกิจค้าส่งและค้าปลีกสินค้าอุปโภคบริโภค  
ช่องทางขาย: POS (หน้าร้าน), Sales Representative, E-commerce

## เป้าหมาย
- รวมศูนย์ข้อมูลการขายทุกช่องทาง
- ควบคุมกระบวนการทางธุรกิจให้เป็นมาตรฐาน
- ลดความผิดพลาด เพิ่มความโปร่งใส
- รองรับการเติบโตในอนาคต

## เอกสารหลัก
- `docs/er-diagram.md` — ER Diagram ครบทุก Entity และ Relationship
- `docs/business-workflow.md` — Flowchart ทุก Business Process

## Module หลัก
| Module | Entity หลัก |
|--------|------------|
| Customer | Customer, CustomerAddress, CustomerContact |
| Product & Pricing | Product, ProductCategory, PriceList, PriceListItem, Promotion |
| Inventory | Warehouse, Stock, StockMovement |
| Sales | Quotation, SalesOrder, SalesOrderLine |
| POS | POSTerminal, POSSession, POSTransaction |
| Delivery | DeliveryOrder, DeliveryLine |
| Invoice & AR | Invoice, Payment, PaymentAllocation |
| Return | ReturnOrder, ReturnOrderLine, CreditNote |
| Purchase & AP | Vendor, PurchaseOrder, GoodsReceipt, VendorBill |
| HR | Employee, SalesTeam |

## Business Rules สำคัญ
- **Credit Check** — SO ของ Wholesale ต้องเช็ค Credit Limit ก่อน Confirm
- **Stock Reserve** — Confirm SO → Reserve Stock ทันที ป้องกัน Over-sell
- **Price Tier** — ราคาขึ้นกับ Customer Type + Channel + Promotion
- **3-Way Matching** — PO + GR + Vendor Bill ต้องตรงก่อนจ่ายเงิน
- **Return Window** — ตรวจสอบวันคืนอัตโนมัติ
- **POS Session** — Open/Close Session ทุกวัน ยอดเงินสดต้องตรงระบบ
- **AR Aging** — Invoice เกินกำหนด → แจ้ง Sales Rep → Block Credit

## Stack (ยังไม่ได้กำหนด — รอตัดสินใจ)
- Backend: ?
- Frontend: ?
- Database: ?

## สิ่งที่ยังไม่ได้ทำ
- [ ] เลือก Tech Stack
- [ ] Database Schema (SQL)
- [ ] API Design
- [ ] UI/UX Wireframe
- [ ] Authentication & Role-Based Access Control
- [ ] Reporting & Dashboard

## Prompt ถัดไป (สั่งตามลำดับ)

**Step 1 — เลือก Stack**
> "เราจะทำ ERP นี้ด้วย [เช่น FastAPI + PostgreSQL + React] ช่วยออกแบบ project structure และ folder layout ให้หน่อย"

**Step 2 — สร้าง Database Schema**
> "สร้าง SQL migration files จาก ER Diagram ใน docs/er-diagram.md ครบทุก table"

**Step 3 — API Design**
> "ออกแบบ REST API endpoints สำหรับ Sales Module ทั้งหมด พร้อม request/response schema"

**Step 4 — RBAC**
> "ออกแบบ Role-Based Access Control สำหรับ ERP นี้ — role หลักคือ Admin, Cashier, Sales Rep, Warehouse, Purchasing, Manager, Viewer"

**Step 5 — เริ่ม Implement Module แรก**
> "เริ่ม implement Customer Management module — CRUD API + validation + unit test"
