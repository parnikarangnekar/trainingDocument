# OMS Data Flow: Multi-line Order from Shopify

## Problem Statement

A customer places a multi-line order on **Shopify (mobile app)**. The order contains:

- SKU 1: `TSHIRT-RED-M`, Quantity: 2  
- SKU 2: `JEANS-BLUE-32`, Quantity: 1  

This order is ingested into **Hotwax OMS**, which stores data across multiple **UDM (Universal Data Model)** entities.

- The Hotwax OMS order ID is stored in `orderId`.
- The Shopify order ID is stored in `externalId` of the `OrderHeader` entity.

---

## Entity-wise Data Representation

### OrderHeader

**Purpose**: Stores the main information about the order.  
**Created When**: As soon as the order is placed.  
**Note**: Root of the order structure. All related items and statuses are linked to this.

| orderId   | externalId    | salesChannelEnumId | statusId       |
|-----------|---------------|--------------------|----------------|
| ORDER1001 | SHOP12345678  | SCP_SHOPIFY_APP    | ORDER_CREATED  |

---

### OrderItem

**Purpose**: Stores each product or SKU included in the order.  
**Created When**: Immediately after `OrderHeader` is created.  
**Note**: One order may have multiple `OrderItem` entries.

| orderId   | orderItemSeqId | productId        | quantity |
|-----------|----------------|------------------|----------|
| ORDER1001 | 00001          | TSHIRT-RED-M     | 2        |
| ORDER1001 | 00002          | JEANS-BLUE-32    | 1        |

---

### OrderItemShipGroup

**Purpose**: Represents shipment grouping.  
**Created When**: 
- Initially at order creation with a placeholder facility (`NA_FACILITY`).
- After **brokering**, a new OISG is created for each fulfillment facility.
**Note**: All items that will be shipped together in a single shipment are grouped under the same OrderItemShipGroup.
This grouping is based on shared shipping attributes such as delivery address, carrier, shipping facility and shipment method. If an order contains items that require different handling or delivery addresses, separate OrderItemShipGroup records are created for each shipment group.

**Before Brokering**:
| orderId   | shipGroupSeqId | facilityId   | carrierPartyId | shipmentMethodTypeId |
|-----------|----------------|--------------|----------------|-----------------------|
| ORDER1001 | 00001          | NA_FACILITY  | NA             | STANDARD              |

**After Brokering**:  
- OMS allocates items to a real warehouse (`DELHI_DC`, for example).
- A new ship group (`00002`) is created to represent the actual shipment from this warehouse.

| orderId   | shipGroupSeqId | facilityId | carrierPartyId | shipmentMethodTypeId |
|-----------|----------------|------------|----------------|-----------------------|
| ORDER1001 | 00002          | DELHI_DC   | BLUEDART       | STANDARD              |

> The new `shipGroupSeqId` reflects a **real-world shipment**, which can now be assigned to a carrier and delivery timeline.


---

### OrderItemShipGrpAssoc

**Purpose**: Maps order items to their shipping groups.

| orderId   | orderItemSeqId | shipGroupSeqId | quantity |
|-----------|----------------|----------------|----------|
| ORDER1001 | 00001          | 00001          | 2        |
| ORDER1001 | 00002          | 00001          | 1        |

**After Brokering**: 
Remaps order items from `00001` (placeholder/virtual facility) to actual shipment group `00002`.

| orderId   | orderItemSeqId | shipGroupSeqId | quantity |
|-----------|----------------|----------------|----------|
| ORDER1001 | 00001          | 00002          | 2        |
| ORDER1001 | 00002          | 00002          | 1        |

---

### OrderItemShipGrpInvRes

**Purpose**: Once fulfillment facility is assigned, inventory is reserved from the real facility.

| orderId   | orderItemSeqId | shipGroupSeqId | quantity | reservedFacilityId |
|-----------|----------------|----------------|----------|---------------------|
| ORDER1001 | 00001          | 00002          | 2        | DELHI_DC            |
| ORDER1001 | 00002          | 00002          | 1        | DELHI_DC            |
---

## Summary of Flow

1. Shopify Order Created → Comes into OMS with an external ID.
2. `OrderHeader` is created with metadata.
3. `OrderItem` lines are created for each SKU.
4. A default `OrderItemShipGroup` (with `NA` facility) is created before brokering.
5. `OrderItemShipGrpAssoc` links each item to the ship group.
6. After brokering:
   - `facilityId` is updated in `OrderItemShipGroup`.
   - `OrderItemShipGrpInvRes` is created to reflect reserved inventory.
