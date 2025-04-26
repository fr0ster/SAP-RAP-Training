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

# 🧐 When to Use Determinations

| Use Determinations | Avoid Determinations |
|:-------------------|:---------------------|
| Deriving calculated or dependent fields | Simple constant values in CDS entities |
| Data adjustments during modify/save phases | Validations — prefer Validations instead |
| Field-triggered recalculations | Process control — prefer Actions |

> 📊 [Reference: SAP Help — Determination and Validation Modelling](https://help.sap.com/docs/abap-cloud/abap-rap/determination-and-validation-modelling)

---

# ✅ Best Practices

| Practice | Reason |
|:---------|:-------|
| Always trigger Determination via event or field change | Prevent unnecessary executions |
| Avoid changing trigger fields inside Determination | Prevent recursion/infinite loops |
| Document which fields trigger Determinations | Easier maintenance |
| Perform only adjustments, not validations | Maintain proper separation of concerns |
> 📊 [Reference: SAP Help — Determination and Validation Modelling](https://help.sap.com/docs/abap-cloud/abap-rap/determination-and-validation-modelling)

---

# 📅 Typical Usage Scenarios

## Greenfield

| Scenario | Example |
|:---------|:--------|
| Populate total_price dynamically | Based on `quantity` and `unit_price` |
| Auto-assign order date on creation | `order_date = sy-datum` |
| Set default currency for customer | Based on `customer_id` lookup |
| Generate a unique ID before save | Generate document number before database persistency |
| Set audit fields | Update `last_changed_by`, `last_changed_at` before save |

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

# 📖 Key Concepts from SAP Documentation

The following concepts are summarized based on SAP official documentation. They are **not direct quotes**, but represent key ideas described in the official materials.

- Determinations are passive implementations that automatically derive or adjust attributes during the transactional lifecycle. They are executed only when relevant data has been modified.
- Create determinations are used to set derived attributes when an instance is created.
- Modify determinations are used to adapt attributes when certain fields change.
- Before-save determinations are used to finalize attributes after all transactional changes and validations before database persistency.

> 📊 [Reference: SAP Help — Determinations Overview](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)
> 📊 [Reference: SAP Help — Determination and Validation Modelling](https://help.sap.com/docs/abap-cloud/abap-rap/determination-and-validation-modelling)

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

Field-Based Determinations are triggered **only when specific fields are changed**. This optimization ensures that unnecessary recalculations or database operations are avoided.

```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
{
  determination update_tax on modify field quantity, unit_price;
}
```

>> 📊 [Reference: SAP Help — Developing Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/implementing-determinations)


---

# 📚 References

| Topic | Link |
|:------|:-----|
| [Determinations Overview](https://help.sap.com/docs/abap-cloud/abap-rap/determinations) |
| [Developing Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/developing-determinations) |
| [Defining Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/defining-determinations) |
| [Implementing Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/implementing-determinations) |
| [Field-Based Determination](https://help.sap.com/docs/abap-cloud/abap-rap/field-based-determination) |
| [Determination and Validation Modelling](https://help.sap.com/docs/abap-cloud/abap-rap/determination-and-validation-modelling) |

