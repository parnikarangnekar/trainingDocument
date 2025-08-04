# OMS Data Flow: Multi-line Order from Shopify

## 📌 Problem Statement

A customer places a multi-line order on **Shopify (mobile app)**. The order contains:

- **SKU 1**: `TSHIRT-RED-M`, Quantity: 2  
- **SKU 2**: `JEANS-BLUE-32`, Quantity: 1  

This order is ingested into **Hotwax OMS**, which stores data across multiple **UDM (Unified Data Model)** entities.

- The **Hotwax OMS order ID** is stored in `orderId`.  
- The **Shopify order ID** is stored in `externalId` of the `OrderHeader` entity.

---

## Entity-wise Data Representation

### OrderHeader
**Purpose:**  
Stores the main information about the order.

**Created When:**  
As soon as the order is placed by the customer (e.g., from website, POS, or API).

**Note:**  
Acts as the root of the order structure. All related items and statuses are linked to this.

---

| Field              | Sample Value          | Description                                 |
|-------------------|-----------------------|---------------------------------------------|
| `orderId`         | `ORDER1001`           | Internal Hotwax OMS order ID                |
| `externalId`      | `SHOP12345678`        | Shopify Order ID                            |
| `salesChannelEnumId` | `SCP_SHOPIFY_APP` | Enum for identifying sales channel (Shopify app) |
| `statusId`        | `ORDER_CREATED`       | Current status of the order                 |



---

### OrderItem
**Purpose:**  
Stores each product or SKU included in the order.

**Created When:**  
Immediately after `OrderHeader` is created.

**Note:**  
One order may have multiple `OrderItem` entries, each representing a unique SKU or product.

---


| Field              | Sample Value               | Description                            |
|-------------------|----------------------------|----------------------------------------|
| `orderItemSeqId`  | `00001`, `00002`           | Line identifiers for items             |
| `productId`       | `TSHIRT-RED-M`, `JEANS-BLUE-32` | Product SKUs                        |
| `quantity`        | `2`, `1`                   | Quantity ordered per SKU               |
| `orderId`         | `ORDER1001`                | Reference to `OrderHeader`             |



---

###  OrderItemShipGroup
**Purpose:**  
Represents shipment grouping (e.g., by shipping address or delivery time).

**Created When:**  
At order creation, especially if there are multiple delivery addresses or methods.

**Note:**  
An order can have multiple ship groups — e.g., part of the order goes to one address, rest to another.

---

| Field                  | Sample Value        | Description                                             |
|-----------------------|---------------------|---------------------------------------------------------|
| `shipGroupSeqId`      | `00001`             | Identifier for the shipping group                      |
| `facilityId`          | `NA_FACILITY`       | Virtual facility (set before brokering)            |
| `carrierPartyId`      | `NA`                | Virtual carrier ID                                 |
| `shipmentMethodTypeId`| `STANDARD`          | Shipping method selected                               |
| `orderId`             | `ORDER1001`         | Reference to the order                                 |

> A default "NA" facility record with `shipGroupSeqId` = `00001` is **always created before brokering**.

---

###  OrderItemShipGrpAssoc


| Field              | Sample Value            | Description                                            |
|-------------------|-------------------------|--------------------------------------------------------|
| `orderItemSeqId`  | `00001`, `00002`        | Item line IDs                                          |
| `shipGroupSeqId`  | `00001`                 | Shipping group assignment                             |
| `quantity`        | `2`, `1`                | Quantity of each item going in the group              |

> **Purpose**: Maps order items to their shipping groups.

---

###  OrderItemShipGrpInvRes
**Purpose:**  
Represents inventory reservation for each order item within a ship group.

**Created When:**  
After the brokering process is complete.


**Note:**  
After brokering decides which warehouse/store will fulfill the item, this entity is populated to lock the stock.

---

| Field              | Sample Value           | Description                                             |
|-------------------|------------------------|---------------------------------------------------------|
| `orderItemSeqId`  | `00001`, `00002`       | Item line IDs with inventory reserved                  |
| `shipGroupSeqId`  | `00001`                | Shipping group linked to the reserved stock            |
| `quantity`        | `2`, `1`               | Quantity reserved per SKU                              |
| `orderId`         | `ORDER1001`            | Reference to the main order                            |



---

##  Summary of Flow

1. **Shopify Order Created** → Comes into OMS with an external ID.
2. **OrderHeader** is created with metadata.
3. **OrderItem** lines are created for each SKU.
4. A default **OrderItemShipGroup** (with `NA` facility) is created before brokering.
5. **OrderItemShipGrpAssoc** links each item to the ship group.
6. After brokering:
    - `facilityId` is updated in `OrderItemShipGroup`.
    - `OrderItemShipGrpInvRes` is created to reflect reserved inventory.

---

