# 📅 Extended Outline: Determinations in SAP RAP (Markdown, 🇬🇧)

## Table of Contents (TOC)

- [What is a Determination in RAP?](#what-is-a-determination-in-rap)
- [Types of Determinations](#types-of-determinations)
- [Declaring Determinations in Behavior Definition](#declaring-determinations-in-behavior-definition)
- [Implementing Determinations in Behavior Implementation](#implementing-determinations-in-behavior-implementation)
- [When to Use Determinations](#when-to-use-determinations)
- [Best Practices](#best-practices)
- [Typical Usage Scenarios](#typical-usage-scenarios)
- [Common Mistakes When Using Determinations](#common-mistakes-when-using-determinations)
- [Key Concepts from SAP Documentation](#key-concepts-from-sap-documentation)
- [Quality Checklist for Determinations](#quality-checklist-for-determinations)
- [Field-Based Determinations](#field-based-determinations)
- [References](#references)

---

# 💪 What is a Determination in RAP

A **Determination** in SAP RAP is a passive, automatically invoked implementation that adjusts or derives attributes during an object's transactional lifecycle (such as creation, field changes, or before save), without explicit user invocation.

> 🖊️ Determinations are triggered **only when data is modified**.

They are used to:
- Derive calculated attributes
- Populate dependent fields
- Initialize instance values during creation

> 📊 [Reference: SAP Help — Determinations Overview](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)

---

# 🔢 Types of Determinations

| Type | Trigger | Example |
|:-----|:--------|:--------|
| **On Create** | When a new instance is created | Set `status = 'New'` upon creation |
| **On Modify** | When specified fields are modified | Recalculate `total_price` if `quantity` or `unit_price` changes |
| **Before Save** | After all transactional changes and validations, before database persistency | Generate unique ID, set audit fields like `last_changed_by`, or trigger a custom event |

> 🖊️ After Modify Determination **does not exist** in RAP.

> 📊 [Reference: SAP Help — Developing Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/developing-determinations)

---

# ✏️ Declaring Determinations in Behavior Definition

```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
{
  determination set_default_values on create;
  determination recalculate_total on modify field quantity, unit_price;
  determination finalize_status before save;
}
```

> 📊 [Reference: SAP Help — Defining Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/defining-determinations)

---

# 🛠️ Implementing Determinations in Behavior Implementation

```abap
METHOD set_default_values.
  LOOP AT keys INTO DATA(ls_key).
    UPDATE zsalesorder
      SET currency_code = 'USD',
          order_date = @sy-datum
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.

METHOD recalculate_total.
  LOOP AT keys INTO DATA(ls_key).
    SELECT SINGLE quantity, unit_price
      INTO @DATA(ls_order)
      FROM zsalesorder
      WHERE salesorder_id = @ls_key-salesorder_id.

    IF ls_order-quantity IS NOT INITIAL AND ls_order-unit_price IS NOT INITIAL.
      DATA(lv_total) = ls_order-quantity * ls_order-unit_price.
      UPDATE zsalesorder
        SET total_price = @lv_total
        WHERE salesorder_id = @ls_key-salesorder_id.
    ENDIF.
  ENDLOOP.
ENDMETHOD.

METHOD finalize_status.
  LOOP AT keys INTO DATA(ls_key).
    UPDATE zsalesorder
      SET finalized_flag = abap_true,
          last_changed_by = @sy-uname,
          last_changed_at = @sy-datum
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.
```

> 📊 [Reference: SAP Help — Implementing Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/implementing-determinations)

---

# 📅 Typical Usage Scenarios

## Greenfield

| Scenario | Example |
|:---------|:--------|
| Populate total_price dynamically | Based on `quantity` and `unit_price` |
| Auto-assign order date on creation | `order_date = sy-datum` |
| Set default currency for customer | Based on `customer_id` lookup |
| Generate a unique ID before save | Generate document number before database persistency |
| Set audit fields | Update `last_changed_by`, `last_changed_at` in `before save` determination |

## Brownfield

| Scenario | Example |
|:---------|:--------|
| Recalculate legacy field values | Net Amount = Gross Amount - Discount |
| Standardize old data formats before save | Date conversion from YYMMDD to YYYYMMDD |
| Trigger custom event before persistency | Publish event before database commit |

---

# ❌ Common Mistakes When Using Determinations

| Mistake | Explanation |
|:--------|:------------|
| Using Determination for static default values | Should be handled using constant values in CDS entities |
| Causing recursive triggers | Changing trigger field inside Determination |
| Mixing validations inside Determinations | Should be separated into Validations |

---

# 📖 Official Quotes from SAP Documentation

> "Determinations are passive implementations that automatically derive attributes during the transaction lifecycle. They are executed **only when relevant data has been modified**."
> — [SAP RAP Documentation](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)

> "Use create determinations to set derived attributes upon instance creation; use modify determinations to adapt values based on field changes; use before-save determinations to finalize attributes before database persistency."
> — [SAP RAP Best Practices](https://help.sap.com/docs/abap-cloud/abap-rap/best-practices-for-determinations)

---

# ✅ Quality Checklist for Determinations

| Question | Note |
|:---------|:-----|
| Is the Determination event-triggered? | Should be linked to create, modify, or before-save |
| Are field triggers documented and isolated? | Transparent and easy to maintain |
| Is only derived/calculated logic included? | No validation or status changes |
| Is transactional consistency ensured? | Mandatory for successful save |

---

# 📈 Field-Based Determinations

Field-Based Determinations are triggered **only when specific fields are changed**.
This optimization ensures that unnecessary recalculations or database operations are avoided.

```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
{
  determination update_tax on modify field quantity, unit_price;
}
```

- `on modify field quantity, unit_price` ensures that `update_tax` runs only when `quantity` or `unit_price` are modified.
- Helps improve performance and transactional efficiency.

> 📊 [Reference: SAP Help — Field-Based Determination](https://help.sap.com/docs/abap-cloud/abap-rap/field-based-determination)

---

# 📗 References

| Topic | Link |
|:------|:-----|
| Determinations Overview | [Link](https://help.sap.com/docs/abap-cloud/abap-rap/determinations-overview) |
| Use Create Determination | [Link](https://help.sap.com/docs/abap-cloud/abap-rap/use-create-determination) |
| Use Modify Determination | [Link](https://help.sap.com/docs/abap-cloud/abap-rap/use-modify-determination) |
| Use Before Save Determination | [Link](https://help.sap.com/docs/abap-cloud/abap-rap/use-before-save-determination) |
| Field-Based Determination | [Link](https://help.sap.com/docs/abap-cloud/abap-rap/field-based-determination) |
| Implement a Determination | [Link](https://help.sap.com/docs/abap-cloud/abap-rap/implement-a-determination) |
| Best Practices for Determinations | [Link](https://help.sap.com/docs/abap-cloud/abap-rap/best-practices-for-determinations) |

