# Retailer OMS Journey – Complete Example

## Business Setup: Meet the Retailer

A fashion retailer named **NotNaked** operates with the following infrastructure:

- **Sales Channels**:
  - Online store on **Shopify**

  - 10+ stores across India

- **Warehouses**:
  - **Warehouse A** (Mumbai) – handles bulk inventory and online orders
  - **Warehouse B** (Bangalore) – supports South Indian region

- **Products**:
  - 500+ SKUs including clothing, footwear, and accessories
  - Example SKUs:
    - `DJ-BL-01`: Blue Denim Jacket
    - `GT-WH-02`: White Graphic T-shirt
    - `SNK-BK-04`: Black Sneakers

- **Inventory** is distributed across warehouses and stores, synced daily with the **Order Management System (OMS)**.

## Goal of OMS

To manage customer orders from any channel, allocate inventory, coordinate fulfillment, and track returns and refunds, all from a central system.

---

## Step-by-Step: A Customer Journey through OMS

### Step 1: Order Placement

Customer Aisha places an order on the Shopify website:

- 1x Blue Denim Jacket (`DJ-BL-01`)
- 2x White Graphic T-shirts (`GT-WH-02`)
- Paid via UPI

OMS captures:
- Order details
- Customer address
- Payment confirmation
- Order status: `ORDER_CREATED`

---

### Step 2: Inventory Allocation

OMS checks inventory availability:

| SKU        | Required | Warehouse A | Store 22 (Delhi) | Warehouse B |
|------------|----------|-------------|------------------|-------------|
| DJ-BL-01   | 1        | In Stock    | Out of stock     | In Stock    |
| GT-WH-02   | 2        | Only 1      | 3 available      | 0           |

OMS decides:
- Jacket → Fulfilled from **Warehouse A**
- T-shirts → Fulfilled from **Store 22**

Two fulfillment groups are created.

---

### Step 3: Shipment Creation

OMS sends pick-pack-ship requests to both locations:

- **Warehouse A**:
  - Picker scans jacket SKU
  - Packing label generated
  - ShipHawk API assigns tracking number `SHK1245XYZ`
  - Status: `Packed → Shipped`

- **Store 22**:
  - Staff picks 2 t-shirts
  - Uses Shiprocket for courier
  - Tracking number: `SRK5678PQR`
  - Status: `Packed → Shipped`

OMS links shipment tracking to order items.

---

### Step 4: Invoice Generation

After shipment:
- OMS calls ERP system
- GST-compliant invoices generated for each fulfillment in ERP

---

### Step 5: Order Delivery

- Jacket delivered on Day 2 (via ShipHawk)
- T-shirts delivered on Day 3 (via Shiprocket)

OMS updates:
- Shipment status: `Delivered`
- Order item status: `Completed`
- Order overall status: `ORDER_COMPLETED`

---

### Step 6: Return Request

Aisha returns 1 t-shirt due to size issue:

- Logs request via Shopify
- OMS creates return case:
  - Linked to original order
  - Reason: `Size issue`
  - Status: `RETURN_INITIATED`

- Return pickup arranged via Shiprocket
- Item received at Store 22

OMS updates:
- Refund triggered via payment gateway
- Return status: `Returned & Refunded`
- Inventory updated (t-shirt added back to Store 22)

---

## Behind the Scenes: Systems Working Together

| Function              | System Used           |
|-----------------------|------------------------|
| Order Capture         | Shopify → OMS API      |
| Inventory Check       | OMS + ERP		 |
| Fulfillment Logic     | OMS + Warehouse/Store  |
| Shipping & Labels     | ShipHawk, Shiprocket   |
| Invoice Generation    | ERP System             |
| Notifications         | Email service	         |
| Return Processing     | OMS + Courier          |

---

## Final Order Summary in OMS

| Item            | Source       | Shipment ID | Status       | Return Status     |
|-----------------|--------------|-------------|--------------|--------------------|
| Denim Jacket    | Warehouse A  | SHK1245XYZ  | Delivered    | -                  |
| T-shirt (x2)    | Store 22     | SRK5678PQR  | Delivered    | 1 Returned         |

---

## Summary

OMS handled the full lifecycle:

- Order capture from Shopify
- Smart inventory splitting
- Multi-location fulfillment
- Shipment and tracking
- Invoicing and delivery
- Return and refund process

## **The Role of the Order Management System (OMS)**

The OMS is the backbone of NotNaked’s omnichannel capabilities, serving multiple critical functions:
1. **Order Aggregation and Processing**
   * **Centralized Order Management:** Consolidates orders from all sales channels (online, in-store, mobile) into a single system.  
   * **Order Routing:** Determines the optimal fulfillment location based on proximity, inventory levels, and predefined business rules.  
   * **Inventory Reservation:** Reserves stock at the appropriate location to prevent overselling.  
2. **Inventory Management**
   * **Real-Time Inventory Tracking:** Monitors stock levels across warehouses and stores, ensuring accurate availability.  
   * **Inventory Synchronization:** Updates inventory data between Shopify, HotWax Commerce, and NetSuite to maintain consistency.  
3. **Fulfillment Coordination**
   * **BOPIS Fulfillment:** Manages in-store pickup orders, notifying store associates and updating order statuses.  
   * **Ship-from-Store:** Enables stores to fulfill online orders, optimizing delivery times and reducing shipping costs.  
   * **Warehouse Fulfillment:** Coordinates standard and express shipping orders from warehouse locations.  
4. **Integration with Other Systems**
   * **Shipping Management (ShipHawk):** Integrates with ShipHawk to automate shipping label generation, rate shopping, and carrier selection for both warehouse and store shipments.  
   * **Returns Management (Loop Returns):** Works with Loop Returns to handle returns and exchanges efficiently, updating inventory and processing refunds or exchanges.  
5. **Customer Experience Enhancement**
   * **Notifications and Alerts:** Automates communication with customers regarding order confirmations, shipment tracking, and pickup readiness.  
   * **Returns and Exchanges Processing:** Simplifies the handling of returns and exchanges through integration with Loop Returns.  
   * **Order Modifications:** Allows for cancellations and adjustments within allowable time frames, managed by CSRs if necessary.
All from a single unified platform.

