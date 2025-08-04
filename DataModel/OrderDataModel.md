# OMS Data Flow: Multi-line Order from Shopify

## Problem Statement

A customer places a multi-line order on **Shopify (mobile app)**. The order contains:

- 1x Blue Denim Jacket (`DJ-BL-01`)
- 2x White Graphic T-shirts (`GT-WH-02`)
- 1x Black Sneakers (`SNK-BK-04`)

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

| orderId   | orderItemSeqId | productId    | quantity |
|-----------|----------------|--------------|----------|
| ORDER1001 | 00001          | DJ-BL-01     | 1        |
| ORDER1001 | 00002          | GT-WH-02     | 2        |
| ORDER1001 | 00003          | SNK-BK-04    | 1        |

---

### OrderItemShipGroup

**Purpose**: Represents shipment grouping.  
**Created When**: 
- Initially at order creation with a placeholder facility (`NA_FACILITY`).
- After **brokering**, a new OISG is created for each fulfillment facility.  
**Note**: All items that will be shipped together in a single shipment are grouped under the same OrderItemShipGroup. This grouping is based on shared shipping attributes such as delivery address, carrier, shipping facility, and shipment method. If an order contains items that require different handling or delivery addresses, separate OrderItemShipGroup records are created for each shipment group.

**Before Brokering**:

| orderId   | shipGroupSeqId | facilityId   | carrierPartyId | shipmentMethodTypeId |
|-----------|----------------|--------------|----------------|-----------------------|
| ORDER1001 | 00001          | NA_FACILITY  | NA             | STANDARD              |

**After Brokering**:  
OMS checks inventory and allocates:
- Jacket → Warehouse A (Mumbai)
- T-shirts → Store 22 (Delhi)
- Sneakers → Warehouse A (Mumbai)

So, two ship groups are created:

| orderId   | shipGroupSeqId | facilityId | carrierPartyId | shipmentMethodTypeId |
|-----------|----------------|------------|----------------|-----------------------|
| ORDER1001 | 00002          | MUMBAI_DC  | BLUEDART       | STANDARD              |
| ORDER1001 | 00003          | DELHI_STR22| DTDC           | STANDARD              |

---

### OrderItemShipGrpAssoc

**Purpose**: Maps order items to their shipping groups.

**Before Brokering**:

| orderId   | orderItemSeqId | shipGroupSeqId | quantity |
|-----------|----------------|----------------|----------|
| ORDER1001 | 00001          | 00001          | 1        |
| ORDER1001 | 00002          | 00001          | 2        |
| ORDER1001 | 00003          | 00001          | 1        |

**After Brokering**:

| orderId   | orderItemSeqId | shipGroupSeqId | quantity |
|-----------|----------------|----------------|----------|
| ORDER1001 | 00001          | 00002          | 1        |
| ORDER1001 | 00002          | 00003          | 2        |
| ORDER1001 | 00003          | 00002          | 1        |

---

### OrderItemShipGrpInvRes

**Purpose**: Once fulfillment facility is assigned, inventory is reserved from the real facility.

| orderId   | orderItemSeqId | shipGroupSeqId | quantity | reservedFacilityId |
|-----------|----------------|----------------|----------|---------------------|
| ORDER1001 | 00001          | 00002          | 1        | MUMBAI_DC           |
| ORDER1001 | 00002          | 00003          | 2        | DELHI_STR22         |
| ORDER1001 | 00003          | 00002          | 1        | MUMBAI_DC           |

---

## Summary of Flow

1. Shopify Order Created → Comes into OMS with an external ID.
2. `OrderHeader` is created with metadata.
3. `OrderItem` lines are created for each SKU.
4. A default `OrderItemShipGroup` (with `NA` facility) is created before brokering.
5. `OrderItemShipGrpAssoc` links each item to the ship group.
6. After brokering:
   - Items are split based on inventory location.
   - Separate `OrderItemShipGroup` entries are created for each fulfillment center.
   - `OrderItemShipGrpAssoc` is updated.
   - `OrderItemShipGrpInvRes` is created to reflect reserved inventory.
