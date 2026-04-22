# ABC Trading ERP System

ระบบ ERP สำหรับบริษัท ABC Trading — ค้าส่งและค้าปลีกสินค้าอุปโภคบริโภค

## ช่องทางการขาย
- **POS** — หน้าร้าน
- **Sales Representative** — ฝ่ายขาย
- **E-commerce** — ออนไลน์

## เอกสาร
| ไฟล์ | รายละเอียด |
|------|-----------|
| [docs/er-diagram.md](docs/er-diagram.md) | ER Diagram — โครงสร้างฐานข้อมูลทั้งหมด |
| [Interactive ER Diagram](https://dbdiagram.io/d/69e88c921bbca033120d41e8) | dbdiagram.io — zoom/pan/คลิก entity ได้ |
| [docs/business-workflow.md](docs/business-workflow.md) | Business Workflow — กระบวนการทางธุรกิจทุก Module |

## Module หลัก
- **Customer Management** — ข้อมูลลูกค้า, ที่อยู่, เครดิต
- **Product & Pricing** — สินค้า, Price List, Promotion
- **Inventory** — คลังสินค้า, Stock Movement
- **Sales** — Quotation → Order → Delivery → Invoice → Payment
- **POS** — Session, Transaction, ใบเสร็จ
- **Purchase** — PO → GR → Vendor Bill (3-Way Matching)
- **Return & Refund** — Return Order, Credit Note
- **AR/AP** — ลูกหนี้/เจ้าหนี้การค้า
