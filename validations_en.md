# 🗕️ Extended Outline: Validations in SAP RAP (Markdown, 🇬🇧)

## Table of Contents (TOC)

- [What is a Validation in RAP?](#what-is-a-validation-in-rap)
- [Types of Validations](#types-of-validations)
- [Declaring Validations in Behavior Definition](#declaring-validations-in-behavior-definition)
- [Implementing Validations in Behavior Implementation](#implementing-validations-in-behavior-implementation)
- [When to Use Validations](#when-to-use-validations)
- [Best Practices](#best-practices)
- [Typical Usage Scenarios](#typical-usage-scenarios)
- [Common Mistakes When Using Validations](#common-mistakes-when-using-validations)
- [Key Concepts from SAP Documentation](#key-concepts-from-sap-documentation)
- [Quality Checklist for Validations](#quality-checklist-for-validations)
- [Field-Based Validations](#field-based-validations)
- [References](#references)

---

# 🛡️ What is a Validation in RAP

A **Validation** in SAP RAP is an optional part of the business object behavior that checks the consistency of business object instances based on trigger conditions.

> 🖊️ Validations do **not** modify data — they **only verify** it.

They are used to:
- Validate business rules
- Ensure mandatory field completion
- Prevent invalid transactions from being saved

> 📊 [Reference: SAP Help — Validations Overview](https://help.sap.com/docs/abap-cloud/abap-rap/validations)

---

# 🔢 Types of Validations

| Type | Trigger | Example |
|:-----|:--------|:--------|
| **On Save** | Before database persistency | Validate mandatory fields are filled |
| **On Modify** | When specified fields are modified | Validate quantity is positive when updated |

---

# ✏️ Declaring Validations in Behavior Definition

```abap
define behavior for /DMO/I_Travel_M alias travel
  ...
{
  ...
  validation validateCustomer on save { create; field customer_id; }
  validation validateAgency   on save { create; field agency_id; }
  validation validateDates    on save { create; field begin_date, end_date; }
  validation validateStatus   on save { create; field overall_status; }
  validation validateCurrencyCode on save { create; field currency_code; }
  validation validateBookingFee   on save { create; field booking_fee; }
}
```

> 📊 [Reference: SAP Help — Defining Validations](https://help.sap.com/docs/abap-cloud/abap-rap/defining-validations)

---

# 🛠️ Implementing Validations in Behavior Implementation

```abap
CLASS lhc_travel IMPLEMENTATION.

  METHOD validate_dates.

    READ ENTITIES OF /DMO/I_Travel_M IN LOCAL MODE
      ENTITY travel
        FIELDS ( begin_date end_date )
        WITH CORRESPONDING #( keys )
      RESULT DATA(travels).

    LOOP AT travels INTO DATA(travel).

      IF travel-end_date < travel-begin_date.

        APPEND VALUE #( %tky = travel-%tky ) TO failed-travel.

        APPEND VALUE #( %tky = travel-%tky
                        %msg = NEW /dmo/cm_flight_messages(
                                   textid     = /dmo/cm_flight_messages=>begin_date_bef_end_date
                                   severity   = if_abap_behv_message=>severity-error
                                   begin_date = travel-begin_date
                                   end_date   = travel-end_date
                                   travel_id  = travel-travel_id )
                        %element-begin_date   = if_abap_behv=>mk-on
                        %element-end_date     = if_abap_behv=>mk-on
                     ) TO reported-travel.

      ELSEIF travel-begin_date < cl_abap_context_info=>get_system_date( ).

        APPEND VALUE #( %tky = travel-%tky ) TO failed-travel.

        APPEND VALUE #( %tky = travel-%tky
                        %msg = NEW /dmo/cm_flight_messages(
                                    textid   = /dmo/cm_flight_messages=>begin_date_on_or_bef_sysdate
                                    severity = if_abap_behv_message=>severity-error )
                        %element-begin_date  = if_abap_behv=>mk-on
                        %element-end_date    = if_abap_behv=>mk-on
                      ) TO reported-travel.

      ENDIF.

    ENDLOOP.

  ENDMETHOD.

ENDCLASS.
```

> 📊 [Reference: SAP Help — Implementing Validations](https://help.sap.com/docs/abap-cloud/abap-rap/implementing-validations)

---

# 🧐 When to Use Validations

Use a Validation when you need to:

- Ensure data consistency and correctness of business object instances based on field changes or modify operations.
- Automatically prevent inconsistent data from being saved by rejecting invalid instances into the `FAILED-<entity>` table.
- Provide consumer-friendly warning or error messages through the `REPORTED-<entity>` table.
- Validate data without modifying it.
- Optimize runtime by tying validations to specific fields or operations (field-based validations).

> 📖 **Important**:  
> - Validations must not use EML modify statements.  
> - In managed scenarios with draft, it is recommended to assign validations to the `PREPARE` phase for correct messaging.  
> - Execution order of multiple validations triggered by the same event is not guaranteed.

---

# ✅ Best Practices

| Practice | Reason |
|:---------|:-------|
| Keep validation logic lightweight | Avoid performance bottlenecks |
| Provide meaningful messages | Help users correct their data |
| Separate validations clearly from data adjustments | Maintain clean transactional flow |
| Prefer field-based validations when possible | Optimize execution only when relevant fields change |

> 📊 [Reference: SAP Help — Determination and Validation Modelling](https://help.sap.com/docs/abap-cloud/abap-rap/determination-and-validation-modelling)

---

# 🗕️ Typical Usage Scenarios

## Greenfield

| Scenario | Example |
|:---------|:--------|
| Mandatory fields enforcement | `customer_id`, `order_date` must be provided |
| Business rule checks | `delivery_date > order_date` |
| Range validations | `discount between 0 and 100` |

## Brownfield

| Scenario | Example |
|:---------|:--------|
| Legacy data consistency | Dates in valid format |
| Business compliance enforcement | Legacy field combinations follow new standards |

---

# ❌ Common Mistakes When Using Validations

| Mistake | Explanation |
|:--------|:------------|
| Modifying data inside Validation | Only checks allowed, no data changes |
| Overloading Validation with heavy logic | Should stay efficient |
| Mixing Determination and Validation logic | Must be separated for clean design |

---

# 📖 Key Concepts from SAP Documentation

The following points summarize SAP official guidance:

- Validations ensure that the transactional data is consistent and correct.
- They must not modify the data they check.
- Field-based validations help optimize performance by triggering only when necessary.
- Validation failures are collected in the `FAILED-<entity>` table and are presented as errors to the user, while `REPORTED-<entity>` holds warning or informational messages.
- Execution order of validations is undefined if multiple validations are triggered by the same event.

> 📊 [Reference: SAP Help — Validations Overview](https://help.sap.com/docs/abap-cloud/abap-rap/validations)

---

# ✅ Quality Checklist for Validations

| Question | Note |
|:---------|:-----|
| Is validation lightweight and efficient? | Avoid heavy DB queries inside |
| Are user-friendly messages provided? | Helps users fix errors |
| Is validation separated from data modification? | Ensures clean architecture |
| Are field-specific validations isolated? | Optimizes performance |

---

# 📈 Field-Based Validations

Field-Based Validations are triggered **only when specified fields change**.  
This optimization minimizes unnecessary checks.

```abap
define behavior for /DMO/I_Travel_M alias travel
  ...
{
  validation validateDates on save { create; field begin_date, end_date; }
}
```

> 📊 [Reference: SAP Help — Field-Based Validation](https://help.sap.com/docs/abap-cloud/abap-rap/implementing-validations)

---

# 📚 References

| Topic | Link |
|:------|:-----|
| [Validations Overview](https://help.sap.com/docs/abap-cloud/abap-rap/validations) |
| [Defining Validations](https://help.sap.com/docs/abap-cloud/abap-rap/defining-validations) |
| [Implementing Validations](https://help.sap.com/docs/abap-cloud/abap-rap/implementing-validations) |
| [Field-Based Validation](https://help.sap.com/docs/abap-cloud/abap-rap/implementing-validations) |
| [Determination and Validation Modelling](https://help.sap.com/docs/abap-cloud/abap-rap/determination-and-validation-modelling) |

