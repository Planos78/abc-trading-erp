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

## 8. AR Aging & Credit Collection Workflow

```mermaid
flowchart TD
    A([Invoice ถึงกำหนดชำระ]) --> B[ระบบคำนวณ AR Aging\nทุกวันอัตโนมัติ]
    B --> C{เกินกำหนดกี่วัน?}
    C -- 1-7 วัน --> D[แจ้งเตือนอัตโนมัติ\nEmail / LINE ลูกค้า]
    C -- 8-30 วัน --> E[แจ้ง Sales Rep\nให้ติดตามโดยตรง]
    C -- 31-60 วัน --> F[Manager รับทราบ\nส่ง Demand Letter]
    C -- 60+ วัน --> G[Block Credit\nห้าม SO ใหม่]
    D --> H{ลูกค้าชำระ?}
    E --> H
    F --> H
    G --> I[ส่งเรื่องให้ Legal\nพิจารณาดำเนินการ]
    H -- ใช่ --> J[รับ Payment\nAllocate → Invoice]
    H -- ไม่ --> K[Escalate ขึ้น Level ถัดไป]
    J --> L{ชำระครบ?}
    L -- ครบ --> M[Unblock Credit อัตโนมัติ]
    L -- บางส่วน --> N[อัปเดต amount_due\nติดตามต่อ]
    M --> O([จบ])
    N --> B
```

---

## 9. Credit Block / Unblock Workflow

```mermaid
flowchart TD
    A([Trigger: AR เกิน 60 วัน\nหรือ Credit Limit เกิน]) --> B[ระบบ Block Customer อัตโนมัติ]
    B --> C[บันทึก AUDIT_LOG\nแจ้ง Sales Rep + Manager]
    C --> D{มี SO Draft ค้างอยู่?}
    D -- ใช่ --> E[Hold SO — ห้าม Confirm]
    D -- ไม่ --> F[รอการชำระ]
    E --> F
    F --> G{ลูกค้าชำระหนี้ค้าง?}
    G -- ไม่ --> F
    G -- ใช่ --> H[Finance ยืนยันการรับเงิน]
    H --> I{Credit ยังไม่เกิน Limit?}
    I -- เกิน --> J[Manager ปรับ Credit Limit\nหรือขอ Deposit]
    I -- ไม่เกิน --> K[Unblock Customer]
    J --> K
    K --> L[แจ้ง Sales Rep\nสามารถ Process SO ได้]
    L --> M([จบ])
```

---

## 10. Stock Take / Physical Count Workflow

```mermaid
flowchart TD
    A([กำหนดวันนับสต็อก]) --> B[Warehouse Manager\nสร้าง Stock Count Sheet]
    B --> C[Lock การเบิก-จ่ายชั่วคราว\nระหว่างนับ]
    C --> D[ทีม Warehouse นับจริง\nบันทึกผลทีละ SKU]
    D --> E[ระบบเปรียบเทียบ\nนับจริง vs ระบบ]
    E --> F{มี Discrepancy?}
    F -- ไม่มี --> G[ยืนยัน Stock ถูกต้อง]
    F -- มี --> H{Variance เกิน Threshold?}
    H -- ไม่เกิน --> I[บันทึก Minor Adjustment\nอัตโนมัติ]
    H -- เกิน --> J[สอบสวนหาสาเหตุ\nขาด / เกิน / เสียหาย]
    J --> K{หาสาเหตุได้?}
    K -- ได้ --> L[บันทึกสาเหตุ\nรับผิดชอบตามนโยบาย]
    K -- ไม่ได้ --> M[Manager อนุมัติ\nForce Adjustment]
    L --> N[Stock Adjustment\nอัปเดต qty_on_hand]
    M --> N
    I --> N
    G --> O[Unlock การเบิก-จ่าย]
    N --> O
    O --> P([จบ])
```

---

## 11. Product Review Workflow

```mermaid
flowchart TD
    A([ลูกค้าได้รับสินค้า\nสถานะ Delivered]) --> B[ระบบส่งคำขอ Review\nEmail / LINE / App]
    B --> C{ลูกค้า Review?}
    C -- ไม่ --> D[ส่งอีกครั้งหลัง 3 วัน\nmax 2 ครั้ง]
    D --> C
    C -- ใช่ --> E[กรอก Rating 1-5 ดาว\n+ Comment]
    E --> F{Rating < 3?}
    F -- ใช่ --> G[แจ้ง Customer Service\nให้ติดต่อกลับ]
    G --> H[บันทึก Complaint\nติดตามแก้ไข]
    F -- ไม่ --> I[ระบบ Pending Approve]
    H --> I
    I --> J[Admin ตรวจสอบ Review]
    J --> K{ผ่านมาตรฐาน?}
    K -- ไม่ --> L[Reject + แจ้งเหตุผล]
    K -- ใช่ --> M[Publish Review]
    M --> N[อัปเดต Product Rating Average]
    N --> O([จบ])
```

---

## 12. Price List Management Workflow

```mermaid
flowchart TD
    A([ต้องการปรับราคา]) --> B{ที่มาของการปรับ}
    B -- ต้นทุนขึ้น --> C[Purchasing แจ้ง\nCost เปลี่ยน]
    B -- กลยุทธ์การตลาด --> D[Marketing เสนอ\nราคาใหม่]
    B -- Promotion ชั่วคราว --> E[สร้าง Promotion\nกำหนด valid_from/to]
    C --> F[สร้าง Price List ใหม่\nกำหนด valid_from]
    D --> F
    F --> G[Manager / MD อนุมัติ]
    G --> H{อนุมัติ?}
    H -- ไม่ --> I[แก้ไขและส่งใหม่]
    I --> G
    H -- ใช่ --> J[Activate Price List ใหม่\nDeactivate ของเก่า]
    E --> K[Activate Promotion]
    J --> L[แจ้ง Sales Rep\nผ่าน Notification]
    K --> L
    L --> M[ระบบดึงราคาใหม่\nอัตโนมัติทุก SO/POS]
    M --> N([จบ])
```

---

## 13. Business Rules สำคัญ

| Rule | รายละเอียด |
|------|-----------|
| Credit Check | ทุก SO ของลูกค้า Wholesale ต้องเช็ค Credit Limit ก่อน Confirm |
| Price Tier | ราคาขึ้นกับ Customer Type (retail/wholesale) + Channel + Promotion |
| Stock Reserve | เมื่อ Confirm SO ระบบ Reserve Stock ทันที — ป้องกัน Over-sell |
| 3-Way Matching | ก่อนจ่ายเงิน Vendor ต้องตรวจสอบ PO + GR + Bill ตรงกัน |
| Return Window | กำหนดวันคืนสินค้าตามนโยบาย (เช่น 7 วัน) — ตรวจสอบอัตโนมัติ |
| POS Session | Cashier ต้อง Open/Close Session ทุกวัน — ยอดเงินสดต้องตรงกับระบบ |
| AR Aging | Invoice เกินกำหนดชำระ → แจ้งเตือน Sales Rep → Block Credit |
| LOT/Batch | สินค้าอุปโภคบริโภคต้องระบุ Lot + Expiry Date — FIFO จ่ายของเก่าก่อน |
| Audit Trail | ทุก action (create/update/approve) บันทึก AUDIT_LOG — ย้อนดูได้ทุก step |
| Review Gate | ลูกค้า Review ได้เฉพาะ Order ที่ Delivered แล้วเท่านั้น — 1 Order 1 Review |
| Loyalty Earn | ทุกการซื้อ 1 บาท = 1 Point — แลกได้เมื่อครบ 500 Point |
| RBAC | ทุก User มี Role — ระบบ enforce permission ทุก endpoint ไม่มี bypass |
