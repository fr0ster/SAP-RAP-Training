# 🗕️ Extended Outline: Determinations in SAP RAP (Markdown, 🇬🇧)

## Table of Contents (TOC)

- [What is a Determination in RAP?](#-what-is-a-determination-in-sap-rap)
- [Types of Determinations](#-types-of-determinations)
- [Types of Determination Implementations](#-types-of-determination-implementations)
- [Declaring Determinations in Behavior Definition](#-declaring-determinations-in-behavior-definition)
- [Implementing Determinations in Behavior Implementation](#-implementing-determinations-in-behavior-implementation)
- [When to Use (and Not Use) Determinations](#-when-to-use-and-not-use-determinations)
- [Best Practices](#-best-practices)
- [Typical Usage Scenarios](#-typical-usage-scenarios)
- [Common Mistakes When Using Determinations](#-common-mistakes-when-using-determinations)

---

### 💪 What is a Determination in SAP RAP

A **Determination** is an automatic piece of business logic executed during an object's lifecycle without direct user invocation.  
[SAP Help](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)

It is used for:
- Populating dependent fields;
- Initializing values when creating an object;
- Performing calculations based on other fields.

> 📢 Determinations are *passive* handlers that automatically react to changes.

---

### 🔢 Types of Determinations

| Type | Description | Example |
|:-----|:------------|:--------|
| **Create** | Triggered when an object is created. | Setting a default `currency_code`. |
| **Update** | Triggered on field changes. | Recalculating `total_amount` when `quantity` or `price` changes. |
| **Before Save** | Triggered before persisting to the database. | Setting `finalized_flag` before commit. |
| **After Modify** | Triggered after in-memory changes. | Logging changes for analytics. |

> 📚 [SAP Help: Determination Types](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)

---

### 👩‍💻 Types of Determination Implementations

| Implementation Type | Description | Example |
|:---------------------|:------------|:--------|
| **Instance Determination** | Applies to specific instances (%key). | Recalculating order value. |
| **Static Determination** | Applies globally without instance context (rare). | Generating unique document numbers. |
| **Field-triggered Determination** | Triggered by changes to specific fields. | Recalculating taxes when amount changes. |
| **Lifecycle-triggered Determination** | Triggered at lifecycle phases. | Initializing status upon creation. |

> 📚 [SAP Help: Determination Implementation](https://help.sap.com/docs/abap-cloud/abap-rap/implement-determinations)

---

### ✏️ Declaring Determinations in Behavior Definition

```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
{
  determination set_default_values on create;
  determination recalculate_total on modify field quantity, price;
  determination finalize_status before save;
}
```

> 📚 [SAP Help: Behavior Definitions](https://help.sap.com/docs/abap-cloud/abap-rap/behavior-definitions)

---

### 🛠️ Implementing Determinations in Behavior Implementation

```abap
METHOD set_default_values.
  LOOP AT keys INTO DATA(ls_key).
    UPDATE zsalesorder
      SET status = 'DRAFT',
          currency_code = 'USD'
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.

METHOD recalculate_total.
  LOOP AT keys INTO DATA(ls_key).
    SELECT SINGLE quantity, price
      INTO @DATA(ls_order)
      FROM zsalesorder
      WHERE salesorder_id = @ls_key-salesorder_id.

    DATA(lv_total) = ls_order-quantity * ls_order-price.

    UPDATE zsalesorder
      SET total_amount = @lv_total
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.

METHOD finalize_status.
  LOOP AT keys INTO DATA(ls_key).
    UPDATE zsalesorder
      SET finalized_flag = abap_true
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.
```

> 📚 [SAP Help: Behavior Implementations](https://help.sap.com/docs/abap-cloud/abap-rap/implement-determinations)

---

### 🧐 When to Use (and Not Use) Determinations

| Use Determinations | Avoid Determinations |
|:-------------------|:---------------------|
| Populating calculated or dependent fields | Setting simple default values (use CDS Default instead) |
| Recalculating dependent values | Status transitions — prefer Actions |
| Data initialization during creation | Data validations — prefer Validations |
| Adapting legacy (brownfield) logic | Direct process control — prefer Actions |

> 📚 [SAP Best Practices: RAP Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)

---

### ✅ Best Practices

| ✅ Practice | 🔍 Why? |
|:------------|:--------|
| Use for complex calculations, not simple defaults | Code maintainability |
| Bind to specific fields | Performance optimization |
| Avoid chaining determinations | Prevent infinite loops |
| Ensure transactional consistency | Data integrity control |
| Document Field Triggers clearly | Easier maintenance |

---

# 🛠️ Typical Usage Scenarios

## ✅ Greenfield Scenarios

| Scenario | Example | Note |
|:---------|:--------|:-----|
| Calculate total based on quantity and price | After changing `quantity` or `price`, recalculate `total_amount`. | [SAP Help: Field Control and Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/field-control) |
| Dynamically set currency based on customer country | After changing `customer_id`, retrieve and set `currency_code`. | Via `on modify field customer_id`. |
| Automatically set order date upon creation | Set `order_date = sy-datum` when creating a record. | Via `on create` determination. |

> ❗ Status `Draft` is handled automatically by the framework — no need for explicit Determinations.

## ✅ Brownfield Scenarios

| Scenario | Example | Note |
|:---------|:--------|:-----|
| Recalculating dependent fields in legacy tables | Automatically update `net_amount` after `gross_amount` changes. | |
| Fetching values from lookup tables | Auto-fill product description when `product_id` is modified. | |
| Formatting data during save | Converting old date format YYMMDD to standard YYYYMMDD during save. | |

---

# ❌ Common Mistakes When Using Determinations

| Mistake | Explanation |
|:--------|:------------|
| Using Determinations for simple defaults | Better handled using `@DefaultValue`. |
| Causing infinite loops through field changes | Avoid changing trigger fields within Determination. |
| Overloading with validations inside Determinations | Should be separated into Validations. |
| Lack of documentation for field triggers | Makes maintenance difficult. |

---

# ⚡ Examples of Incorrect Determination Usage

## ❌ Example: Improper Default Setting

```abap
METHOD set_default_country.
  LOOP AT keys INTO DATA(ls_key).
    UPDATE zsalesorder
      SET country = 'UA'
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.
```
> ❗ Better handled using a CDS annotation `@DefaultValue: 'UA'`.

---

# 📖 Extracts from Official SAP Documentation

> "Determinations should automatically derive or adjust attributes during a transactional lifecycle without explicit user interaction."  
> — [SAP RAP Documentation](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)

> "Default values should primarily be defined in the CDS model where possible to avoid redundant coding."  
> — [SAP RAP Best Practices](https://help.sap.com/docs/abap-cloud/abap-rap/best-practices)

> "Use create determinations for setting derived attributes at instance creation time; use modify or before-save determinations for derived values that depend on changes made during the transaction."  
> — [SAP RAP Best Practices](https://help.sap.com/docs/abap-cloud/abap-rap/best-practices)

---

# ✅ Determination Quality Checklist

| Question | Note |
|:---------|:-----|
| Is it bound to events or field changes? | Should be `on create`, `on modify field ...`. |
| Does it avoid changing its own trigger fields? | To prevent cycles. |
| Is it calculating complex dependencies, not just simple defaults? | Simple defaults should be handled in CDS. |
| Are field triggers documented? | For transparency and maintenance. |

---

# ✨ Good Examples of Determination Implementations

## ✅ Example: Proper Total Amount Recalculation

```abap
METHOD recalculate_total_amount.
  LOOP AT keys INTO DATA(ls_key).
    SELECT SINGLE quantity, price
      INTO @DATA(ls_order)
      FROM zsalesorder
      WHERE salesorder_id = @ls_key-salesorder_id.

    IF ls_order-quantity IS NOT INITIAL AND ls_order-price IS NOT INITIAL.
      DATA(lv_total) = ls_order-quantity * ls_order-price.
      UPDATE zsalesorder
        SET total_amount = @lv_total
        WHERE salesorder_id = @ls_key-salesorder_id.
    ENDIF.
  ENDLOOP.
ENDMETHOD.
```
> ✔️ Good practice: only calculations, no changes to triggering fields.

