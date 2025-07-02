# Flag Definitions
Flag definitions detail the structure of `data` and `resolution`; though, these fields are ultimately communicated via
strings of serialized JSON.

You may omit fields from JSON when irrelevant, such as when a duplicate flag is confirmed. The `falseDuplicate` field
will not be present in the resolution string and consumers may consider it `false`.

**Note**: JSON field names should be treated case-insensitive

## List of Flags

* [`MANUAL_REJECT`](#manualreject)
* [`DUPLICATE`](#duplicate)
* [`EXCESSIVE_POINTS`](#excessivepoints)
* [`USER_SET_STORE`](#usersetstore)
* [`TOO_OLD`](#tooold)
* [`MISSING_RECEIPT_INFORMATION`](#missingreceiptinformation)
* [`MISSING_ITEM_INFORMATION`](#missingiteminformation)
* [`USER_UPDATED_ITEMS`](#userupdateditems)

---

## MANUAL_REJECT <a id="manualreject"></a>

Occurs when the "reject" button is pressed from the support dashboard - puts receipt in rejected state.

You can undo a manual reject by manually approving the receipt via a similar "save/accept" button on the support
dashboard.

| Automations        |         | 
| ------------------ | ------- |
| Auto-resolvable    | `false` |
| Active on creation | `false` |

### Schema

| Field                   | Type      | Purpose |
| ----------------------- | --------- | ------- |
| `resolution.rejectedAt` | `integer` | Unix millisecond timestamp representing when the receipt was last rejected (a receipt can be approved and rejected multiple times) |
| `resolution.approvedAt` | `integer` | Unix millisecond timestamp representing when the receipt was last accepted (a receipt can be approved and rejected multiple times) |

A manual rejection flag only prevents receipt processing if `rejectedAt > approvedAt`.

[back to top](#Flag-Definitions)

---

## DUPLICATE <a id="duplicate"></a>

Occurs when Analogo detects a receipt as a duplicate of another receipt. Typically this means that the store name,
purchase date, total, and certain other attributes match exactly; refer to
the [Analogo documentation](https://bitbucket.org/fetchrewards/analogo/src/main/README.md)
for more details.

Duplicate flags are **always** confirmed by default and cause receipt rejection. As such, the only way a user can
challenge a duplicate is to contact support directly (roughly 99% of duplicates *are* duplicates).

| Automations        |         |
| ------------------ | ------- |
| Auto-resolvable    | `false` |
| Active on creation | `false` |

### Schema

| Field                                  | Type       | Purpose |
| -------------------------------------- | ---------- | ------- |
| `data.matchedIds`                      | `string` | Indicates which receipt IDs appear to be duplicates of the receipt this flag is assigned to. |
| `data.sameUser`                        | `boolean`  | Denotes whether this receipt is a duplicate of another receipt in the submitting user's history. |
| `resolution.falseDuplicate`            | `boolean`  | If a receipt is verified as unique, this field will be `true`. |
| `resolution.confirmed`                 | `boolean`  | If a receipt is verified as a duplicate, this field will be `true`. |

[back to top](#Flag-Definitions)

---

## EXCESSIVE_POINTS <a id="excessivepoints"></a>

Occurs when an item *price* is greater than the threshold for the flag. There is a threshold per unit ("item cost
threshold") and another threshold for the entire line item ("final price threshold").

Excessive points only apply to **partner** items that receive points.

These thresholds are defined in
the [Receipt Processor config](https://bitbucket.org/fetchrewards/receipt-processor/src/main/config/prod.yml)
under `excessiveItemPoints`.

If the receipt's point total is less than `200`, we will ignore excessive points.

| Automations        |         |
| ------------------ | ------- |
| Auto-resolvable    | `false` |
| Active on creation | `true`  |

### Schema

| Field                                  | Type               | Purpose |
| -------------------------------------- | ------------------ | ------- |
| `data.items`                           | `map[integer]item` | Which items were identified with excessive prices. |
| `resolution.shouldAward`               | `boolean`          | If set to `true`, the receipt is reprocessed with the resolution's items. |
| `resolution.items`                     | `map[integer]item` | The accurate item data - the items stored on the backend will be updated with the items in this list. |

| Field                    | Type             | Purpose |
| ------------------------ | ---------------- | ------- |
| `item.quantityPurchased` | `float (64-bit)` | How many of the item were purchased. |
| `item.finalPrice`        | `string`         | The total price of the item (unit price is derived from this divided by quantity purchased). |
| `item.unitOfMeasure`     | `string`         | The unit of measure for this item, such as `lb`, `gal`, etc. |
| `item.description`       | `string`         | A short product description (SPD) of this item. |
| `item.upc`               | `string`         | The item's UPC. |
| `item.fido`              | `string`         | The item's FIDO. |

The `integer` key for item maps represents the line on the receipt the item appeared (0-indexed).

If `resolution.shouldAward` is `false`, the receipt will be rejected.

If a resolution item differs from a data item on any of the following fields:

- `upc`
- `fido`

The **entire** item will be overwritten to match the resolution item. Otherwise, only the `quantityPurchased`
, `finalPrice`, and `unitOfMeasure` are persisted to the backend (technically, the unit price of an item is derived from
the aforementioned fields and also persisted to the backend)

[back to top](#Flag-Definitions)

---

## USER_SET_STORE <a id="usersetstore"></a>

Occurs when a user provides the store name on the receipt - we could not accurately read it or we did not recognize it.

If the receipt's point total before bonus points and lapsed-user offers is less than `250` *and* the receipt *does not*
qualify for a store-related offer, we
ignore user-set stores.

| Automations        |         |
| ------------------ | ------- |
| Auto-resolvable    | `false` |
| Active on creation | `true`  |

### Schema

| Field              | Type     | Purpose |
| ------------------ | -------- | ------- |
| `data.store`       | `string` | The user's provided store. |
| `resolution.store` | `string` | The accurate store name. |

The store name is canonicalized if the receipt's store name is to be changed.

This flag **never** induces receipt rejection.

[back to top](#Flag-Definitions)

---

## TOO_OLD <a id="tooold"></a>

Occurs when the receipt's purchase date is outside the acceptable scanning age of a receipt (at the time of writing, 14
days before day scanned).

| Automations        |         |
| ------------------ | ------- |
| Auto-resolvable    | `false` |
| Active on creation | `false` |

Much like duplicate flags, too-old flags are always created inactive, thus causing receipt rejection (the vast majority
of too-old flags are resolved with the same date and are thus still too old).

### Schema

| Field                     | Type     | Purpose |
| ------------------------- | -------- | ------- |
| `data.purchaseDate`       | `string` | The captured purchase date. |
| `data.purchaseTime`       | `string` | The captured purchase time. |
| `resolution.purchaseDate` | `string` | The actual purchase date. |
| `resolution.purchaseTime` | `string` | The actual purchase time. |

[back to top](#Flag-Definitions)

---

## MISSING_RECEIPT_INFORMATION <a id="missingreceiptinformation"></a>

Occurs when a receipt is missing key fields:

- Purchase date
- Store name
- Total

| Automations        |         |
| ------------------ | ------- |
| Auto-resolvable    | `false` |
| Active on creation | `true`  |

Missing receipt information is **always** flagged if conditions apply (i.e. there is no point threshold).

### Schema

| Field                              | Type      | Purpose |
| ---------------------------------- | --------- | ------- |
| `data.originalReceiptFields`       | `fields`  | The fields present/read from the receipt. |
| `data.userSuppliedReceiptFields`   | `fields`  | The fields that were missing and thus user supplied.
| `resolution.verifiedReceiptFields` | `fields`  | The actual fields of the receipt. |
| `resolution.resolved`              | `boolean` | Whether the flag is actually resolved. |

| Field                 | Type     | Purpose |
| --------------------- | -------- | ------- |
| `fields.purchaseDate` | `string` | The receipt's purchase date. |
| `fields.purchaseTime` | `string` | The receipt's purchase time. |
| `fields.total`        | `string` | The receipt's total as a string (as captured) |
| `fields.storeName`    | `string` | The receipt's store name. |

Any fields that are not blank in the resolution will overwrite the original fields of the receipt.

This flag **never** induces receipt rejection.

[back to top](#Flag-Definitions)

---

## MISSING_ITEM_INFORMATION <a id="missingiteminformation"></a>

Occurs when an item on the receipt is missing key information:

- Quantity
- Final Price
- Unit of Measure

| Automations        |         |
| ------------------ | ------- |
| Auto-resolvable    | `false` |
| Active on creation | `true`  |

Missing item information is **always** flagged if conditions are met, i.e. there is no point threshold the receipt must
meet to be flagged.

| Field                              | Type      | Purpose |
| ---------------------------------- | --------- | ------- |
| `data.itemFields`       | `itemFields`  | The fields present/read from the receipt. |
| `resolution.itemFields` | `itemFields`  | The actual fields of the receipt. |

| Field                          | Type              | Purpose |
| ------------------------------ | ----------------- | ------- |
| `itemFields.index`             | `integer`         | The index of the item on the receipt (0-indexed) |
| `itemFields.quantityPurchased` | `float (64-bit)`  | The number or amount of items purchased. |
| `itemFields.unitOfMeasure`     | `string`          | The unit of measure for this item (`lb`, `kg`, etc.) |
| `itemFields.finalPrice`        | `string`          | The item's total price. |

If any items exist in `resolution.itemFields`, the extant receipt items are overwritten.

If `itemFields.finalPrice` is empty, it is assumed to be 0.

Unit price for items is derived from `itemFields.quantityPurchased / itemFields.finalPrice`.
If `itemFields.quantityPurchased == 0`, then the unit price is set to the item's final price.

[back to top](#Flag-Definitions)

---

## USER_UPDATED_ITEMS <a id="userupdateditems"></a>

Occurs when a user make corrections to their receipt after scanning.

| Automations        |        |
| ------------------ | ------ |
| Auto-resolvable    | `true` |
| Active on creation | `true` |

A user can only submit corrections when the receipt is not in flagged status. Submitting corrections means the user will
need to wait for approval before correcting again.

Auto-resolutions of this flag succeed if the total points awarded to user updates is less than 200 points (this applies
across all user's corrections)

Corrections are only "positive" - a user cannot remove items, only add new and update existing items.

| Field                            | Type           | Purpose |
| -------------------------------- | -------------- | ------- |
| `data.userRequestedItemUpdates`  | `itemUpdate[]` | The items the user requested edits to. |
| `data.userRequestedNewItems`     | `itemUpdate[]` | The items the user requested to add to the receipt. |
| `resolution.verifiedItemUpdates` | `itemUpdate[]` | The verified (by support) items to update with user changes. |
| `resolution.verifiedNewItems`    | `itemUpdate[]` | The verified (by support) items to add to the receipt. |
| `resolution.existingItemCount`   | `integer`      | The number of items on the receipt before this update. Used to set the index of new items. |
| `resolution.resolved`            | `boolean`      | Whether the flag is actually resolved.  |

| Field                          | Type             | Purpose |
| ------------------------------ | ---------------- | ------- |
| `itemUpdate.index`             | `integer`        | The item's index. 0 if a new item. |
| `itemUpdate.description`       | `string`         | A short description for the item. |
| `itemUpdate.upc`               | `string`         | The item's UPC. |
| `itemUpdate.fido`              | `string`         | The item's FIDO. |
| `itemUpdate.quantityPurchased` | `float (64-bit)` | The number or amount of this item purchased. |
| `itemUpdate.unitOfMeasure`     | `string`         | The unit of measure for this item (`lb`, `kg`, etc.) |
| `itemUpdate.finalPrice`        | `string`         | The final price of this item (total price of all of these items) |

Like with missing item information, if `itemFields.finalPrice` is empty, it is assumed to be 0.

As well, like missing item information, unit price for items is derived
from `itemFields.quantityPurchased / itemFields.finalPrice`. If `itemFields.quantityPurchased == 0`, then the unit price
is set to the item's final price.

[back to top](#Flag-Definitions)

---