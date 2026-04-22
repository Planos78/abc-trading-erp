# ERP System — ABC Trading Co.
## Part 2: Business Workflow

---

## 1. Sales Workflow — ช่องทาง POS (หน้าร้าน)

```mermaid
flowchart TD
    A([ลูกค้าเข้าร้าน]) --> B[เปิด POS Session\nby Cashier]
    B --> C[สแกนสินค้า / เลือกสินค้า]
    C --> D{มีสินค้าในสต็อก?}
    D -- ไม่มี --> E[แจ้งลูกค้า / หาทางเลือก]
    D -- มี --> F[ระบบดึงราคาจาก Price List\nตาม Customer Type]
    F --> G{มี Promotion?}
    G -- มี --> H[คำนวณส่วนลด Promotion]
    G -- ไม่มี --> I[สรุปยอดรวม]
    H --> I
    I --> J[เลือกวิธีชำระเงิน\nเงินสด / บัตร / QR / โอน]
    J --> K{ชำระสำเร็จ?}
    K -- ไม่สำเร็จ --> J
    K -- สำเร็จ --> L[สร้าง Sales Order + POS Transaction]
    L --> M[ตัดสต็อกอัตโนมัติ]
    M --> N[ออกใบเสร็จ / Tax Invoice]
    N --> O[ปิด Session / สรุปยอดประจำวัน]
    O --> P([จบ])
```

---

## 2. Sales Workflow — ช่องทาง Sales Representative

```mermaid
flowchart TD
    A([Sales Rep เข้าพบลูกค้า]) --> B{ลูกค้าเก่าหรือใหม่?}
    B -- ใหม่ --> C[สร้าง Customer Profile\nกำหนด Credit Limit / Payment Term]
    B -- เก่า --> D[ดึงข้อมูลลูกค้าและ\nประวัติการสั่งซื้อ]
    C --> D
    D --> E[สร้าง Quotation\nพร้อม Price List ตาม Tier]
    E --> F[ส่ง Quotation ให้ลูกค้า]
    F --> G{ลูกค้าตอบรับ?}
    G -- ปฏิเสธ --> H[บันทึกเหตุผล / ติดตาม]
    G -- ต่อรอง --> E
    G -- ยืนยัน --> I[Convert Quotation → Sales Order]
    I --> J{เช็ค Credit Limit}
    J -- เกิน Limit --> K[ส่ง Manager Approve]
    K --> L{Manager อนุมัติ?}
    L -- ไม่อนุมัติ --> M[แจ้งลูกค้า / ปรับยอด]
    L -- อนุมัติ --> N
    J -- ไม่เกิน --> N[Confirm Sales Order]
    N --> O[ระบบ Reserve Stock]
    O --> P[สร้าง Delivery Order]
    P --> Q[จัดส่งสินค้า]
    Q --> R[อัปเดตสถานะ Delivered]
    R --> S[สร้าง Invoice]
    S --> T{Payment Term}
    T -- เงินสด / โอนทันที --> U[รับชำระ → สร้าง Payment]
    T -- เครดิต 30/60/90 วัน --> V[ติดตาม AR]
    V --> W{ครบกำหนด?}
    W -- ยังไม่ครบ --> V
    W -- ครบ / เกินกำหนด --> U
    U --> X[Allocate Payment → Invoice]
    X --> Y([Invoice Paid — จบ])
```

---

## 3. Sales Workflow — ช่องทาง E-commerce

```mermaid
flowchart TD
    A([ลูกค้าสั่งซื้อออนไลน์]) --> B[ระบบรับ Order อัตโนมัติ]
    B --> C{ลูกค้ามีบัญชีในระบบ?}
    C -- ไม่มี --> D[สร้าง Customer อัตโนมัติ]
    C -- มี --> E[Sync ข้อมูล]
    D --> E
    E --> F{สต็อกพร้อม?}
    F -- ไม่พร้อม --> G[แจ้งลูกค้า Back-order\nหรือยกเลิก]
    F -- พร้อม --> H[สร้าง Sales Order\nChannel = ECOMMERCE]
    H --> I[ชำระเงินออนไลน์\nQR / Credit Card]
    I --> J{Payment Gateway ยืนยัน?}
    J -- ไม่สำเร็จ --> K[แจ้งลูกค้า / รอ 30 นาที]
    K --> I
    J -- สำเร็จ --> L[Reserve Stock]
    L --> M[สร้าง Delivery Order]
    M --> N[จัดส่งพัสดุ / ขนส่ง]
    N --> O[อัปเดต Tracking Number]
    O --> P{ส่งถึงมือลูกค้า?}
    P -- ไม่ --> Q[ติดตาม / แก้ไขปัญหา]
    Q --> P
    P -- ใช่ --> R[อัปเดต Delivered]
    R --> S[ออก Invoice / ใบเสร็จ Email]
    S --> T([จบ])
```

---

## 4. Inventory Management Workflow

```mermaid
flowchart TD
    A([สินค้าเข้าระบบ]) --> B{ประเภทการรับสินค้า}
    B -- ซื้อจาก Vendor --> C[Goods Receipt\nอ้างอิง PO]
    B -- โอนจากคลังอื่น --> D[Stock Transfer]
    B -- ปรับปรุงสต็อก --> E[Stock Adjustment\nโดย Warehouse Manager]

    C --> F[ตรวจสอบคุณภาพ / ปริมาณ]
    F --> G{ตรงตาม PO?}
    G -- ไม่ตรง --> H[บันทึก Discrepancy\nแจ้ง Purchasing]
    G -- ตรง --> I[อัปเดต Stock ON HAND]
    D --> I
    E --> I

    I --> J[Stock พร้อมจ่าย]

    K([คำสั่งขาย / POS]) --> L{Stock Available?}
    L -- ไม่พอ --> M[แจ้งเตือน Low Stock\nสร้าง Reorder Request]
    L -- พอ --> N[Reserve Stock\n+1 qty_reserved]
    N --> O[Picking & Packing]
    O --> P[ส่งสินค้า]
    P --> Q[ตัด Stock ON HAND\n-1 qty_reserved]

    M --> R{ต้องสั่งซื้อ?}
    R -- ใช่ --> S[สร้าง Purchase Order]
    R -- รอก่อน --> T[Monitor]
```

---

## 5. Purchase / Procurement Workflow

```mermaid
flowchart TD
    A([ความต้องการสินค้า]) --> B{แหล่งที่มา}
    B -- Reorder Point ต่ำกว่ากำหนด --> C[ระบบแจ้งเตือนอัตโนมัติ]
    B -- ฝ่ายขายร้องขอ --> D[Sales Request]
    B -- Warehouse ร้องขอ --> E[Warehouse Request]
    C --> F[Purchasing ตรวจสอบ\nเลือก Vendor]
    D --> F
    E --> F
    F --> G[สร้าง Purchase Order]
    G --> H[Manager อนุมัติ PO]
    H --> I{อนุมัติ?}
    I -- ไม่อนุมัติ --> J[แก้ไข / ยกเลิก]
    I -- อนุมัติ --> K[ส่ง PO ให้ Vendor]
    K --> L[Vendor ยืนยัน & จัดส่ง]
    L --> M[รับสินค้า — สร้าง Goods Receipt]
    M --> N[ตรวจสอบสินค้า]
    N --> O{ตรงตาม PO?}
    O -- ไม่ตรง --> P[แจ้ง Vendor\nReturn / Debit Note]
    O -- ตรง --> Q[อัปเดต Stock]
    Q --> R[รับ Vendor Invoice]
    R --> S[จับคู่ 3-Way Matching\nPO + GR + Invoice]
    S --> T{ตรงกัน?}
    T -- ไม่ตรง --> U[ยึดการชำระ\nสอบถาม Vendor]
    T -- ตรง --> V[สร้าง Vendor Bill]
    V --> W[ชำระตาม Payment Term]
    W --> X([จบ])
```

---

## 6. Return & Refund Workflow

```mermaid
flowchart TD
    A([ลูกค้าแจ้งคืนสินค้า]) --> B[ตรวจสอบ Original Order]
    B --> C{อยู่ในเงื่อนไขการคืน?}
    C -- ไม่ --> D[แจ้งลูกค้า — ปฏิเสธ]
    C -- ใช่ --> E[สร้าง Return Order]
    E --> F[อนุมัติโดย Supervisor]
    F --> G{อนุมัติ?}
    G -- ไม่ --> D
    G -- ใช่ --> H[รับสินค้าคืน]
    H --> I{สภาพสินค้า}
    I -- ดี --> J[นำกลับสต็อก\nStock Movement IN]
    I -- เสียหาย --> K[บันทึกของเสีย\nStock Adjustment]
    J --> L[สร้าง Credit Note]
    K --> L
    L --> M{ประเภทคืนเงิน}
    M -- เงินสด / โอน --> N[ออก Refund Payment]
    M -- เครดิต --> O[บันทึก Credit Balance\nใช้ตัดยอดครั้งต่อไป]
    N --> P([จบ])
    O --> P
```

---

## 7. ภาพรวม Module และความเชื่อมโยง

```mermaid
graph LR
    subgraph "ช่องทางขาย"
        POS[🏪 POS]
        SR[👤 Sales Rep]
        EC[🌐 E-commerce]
    end

    subgraph "Sales Module"
        QT[Quotation]
        SO[Sales Order]
        DL[Delivery]
        IV[Invoice]
        PY[Payment / AR]
    end

    subgraph "Inventory Module"
        STK[Stock]
        WH[Warehouse]
        SM[Stock Movement]
    end

    subgraph "Purchase Module"
        PO[Purchase Order]
        GR[Goods Receipt]
        VB[Vendor Bill / AP]
    end

    subgraph "Return Module"
        RO[Return Order]
        CN[Credit Note]
    end

    POS --> SO
    SR --> QT --> SO
    EC --> SO

    SO --> DL --> SM
    SO --> IV --> PY

    SM --> STK
    WH --> STK
    GR --> SM
    PO --> GR --> VB

    SO --> RO --> CN
    RO --> SM
    CN --> PY
```

---

## 8. Business Rules สำคัญ

| Rule | รายละเอียด |
|------|-----------|
| Credit Check | ทุก SO ของลูกค้า Wholesale ต้องเช็ค Credit Limit ก่อน Confirm |
| Price Tier | ราคาขึ้นกับ Customer Type (retail/wholesale) + Channel + Promotion |
| Stock Reserve | เมื่อ Confirm SO ระบบ Reserve Stock ทันที — ป้องกัน Over-sell |
| 3-Way Matching | ก่อนจ่ายเงิน Vendor ต้องตรวจสอบ PO + GR + Bill ตรงกัน |
| Return Window | กำหนดวันคืนสินค้าตามนโยบาย (เช่น 7 วัน) — ตรวจสอบอัตโนมัติ |
| POS Session | Cashier ต้อง Open/Close Session ทุกวัน — ยอดเงินสดต้องตรงกับระบบ |
| AR Aging | Invoice เกินกำหนดชำระ → แจ้งเตือน Sales Rep → Block Credit |
