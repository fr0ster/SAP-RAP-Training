# 📅 Extended Summary: Actions in SAP RAP (Markdown, 🇬🇧)

## Table of Contents (TOC)

- [What is an Action in RAP?](#-what-is-an-action-in-sap-rap)
- [Action Types](#-action-types)
- [Action Implementation Types](#-action-implementation-types)
- [Defining Actions in Behavior Definition](#-defining-actions-in-behavior-definition)
- [Implementing Actions in Behavior Implementation](#-implementing-actions-in-behavior-implementation)
- [Transactional Behavior of Actions](#-transactional-behavior-of-actions)
- [Draft-enabled Actions](#-draft-enabled-actions)
- [Action Validations](#-action-validations)
- [Best Practices](#-best-practices)

### 💪 What is an Action in SAP RAP
An **Action** is an explicit business operation that changes the state or structure of a Business Object, which cannot be realized through standard Create, Read, Update, Delete, or Lock operations.

Actions are used for:
- changing the status of an object (e.g., set an order "On Hold"),
- executing specific business logic or complex processing procedures (e.g., price recalculation, approval workflows),
- initiating mass or conditional changes that go beyond simple record creation or modification.

> 📢 Standard CRUD operations: Read, Create, Update, Delete, Lock — handle basic record management.
> Actions extend object capabilities to support specific business scenarios.

> 🔍 Example Actions:
> - Set On Hold (set order to waiting mode)
> - Release From Hold (remove order from hold)
> - Approve (approve an object)
> - Recalculate Pricing (recalculate prices based on new conditions)

### 🔢 Action Types
| Type | Description |
|:-----|:------------|
| **Internal Action** | Triggered only within the Business Object logic, not available for external calls. |
| **Static Action** | Executed without reference to a specific instance, uses `%cid` for call identification. |
| **Repeatable Action** | Allows multiple executions on the same instance during a transaction using `%cid`. |
| **Factory Action** | Creates new instances in memory with temporary `%cid` identifiers, without immediate database persistence. |
| **Save Action** | Automatically triggered during the save phase to finalize changes before commit. |

> ℹ️ `%cid` — Temporary identifier for objects created or modified during a transaction before database persistence. [More Details](https://help.sap.com/docs/abap-cloud/abap-rap/actions#define-actions)

### 👩‍💻 Action Implementation Types
| Implementation Type | Description |
|:---------------------|:------------|
| **Instance Action** | Works with a specific instance via `%key`, must be tied to an existing record. |
| **Static Action** | Triggered without reference to a specific instance; `%cid` identifies the operation within the session. |
| **Action with Parameters** | Accepts a `%param` structure containing input data for processing. |
| **Action with Result Type Entity** | Creates new objects in memory with `%cid`, which will be persisted after Save. |
| **Factory Action** | Mass-creates new objects, allowing relation setup through `%cid_ref`. |

### 📚 Special Parameters in SAP RAP Actions
| Symbol | Description | Link |
|:-------|:------------|:-----|
| **%cid** | Temporary local ID of an object created or modified during a transaction, used for linking instances in memory. | [SAP Help: Working with %cid](https://help.sap.com/docs/abap-cloud/abap-rap/actions#define-actions) |
| **%key** | Identifier of an existing Business Object instance, used to access or modify database records. | [SAP Help: Instance Actions and %key](https://help.sap.com/docs/abap-cloud/abap-rap/actions#instance-and-static-actions) |
| **%param** | Structure of parameters passed into the Action for additional input data. | [SAP Help: Actions with Parameters](https://help.sap.com/docs/abap-cloud/abap-rap/actions#actions-with-parameters) |

### ✏️ Defining Actions in Behavior Definition
```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
{
  action set_on_hold result [1] $self;
  action release_from_hold result [1] $self;
  static action refresh_all;
}
```

### 🛠️ Implementing Actions in Behavior Implementation
```abap
METHOD set_on_hold.
  LOOP AT keys INTO DATA(ls_key).
    UPDATE zsalesorder
      SET status = 'ON_HOLD'
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.

METHOD release_from_hold.
  LOOP AT keys INTO DATA(ls_key).
    UPDATE zsalesorder
      SET status = 'RELEASED'
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.

METHOD refresh_all.
  UPDATE zsalesorder
    SET last_refresh = sy-datum.
ENDMETHOD.
```

### 🛠️ Transactional Behavior of Actions
- Actions are executed within Save transactions.
- If an Action changes data, it must return `result`.
- Save Actions are only triggered during the Save Sequence.

### 📝 Draft-enabled Actions
```abap
draft action submit result [1] $self;
draft action discard result [1] $self;
```

### 📋 Action Validations
```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
{
  action set_on_hold result [1] $self;
  validate action set_on_hold on save { my_validation; }

  action release_from_hold result [1] $self;
  validate action release_from_hold on save { my_validation; }
}
```

### ✅ Best Practices
| ✅ Practice | 🔍 Why? | 🔗 Official Reference |
|:-----------|:-------|:----------------------|
| Always return `$self` | To refresh UI after object modification | [SAP Help: Actions and $self](https://help.sap.com/docs/abap-cloud/abap-rap/actions#define-actions) |
| Perform validations in `validate_action` | Check business rules before saving | [SAP Help: Validations in RAP](https://help.sap.com/docs/abap-cloud/abap-rap/validations) |
| Avoid modifying key fields unless necessary | To maintain record identity | [SAP Best Practices: Avoid Updating Key Fields](https://help.sap.com/docs/abap-cloud/abap-rap/behavior-definitions#restrictions) |
| Use ETag | Prevent concurrent update conflicts | [SAP Help: Managing Concurrency with ETags](https://help.sap.com/docs/abap-cloud/abap-rap/concurrency-control#etag) |
| Manage transactions properly (ROLLBACK) | Ensure data consistency on errors | [SAP Help: Transactional Behavior](https://help.sap.com/docs/abap-cloud/abap-rap/actions#transactional-aspects) |
| Separate Draft and Active logic | Avoid inconsistencies between drafts and active instances | [SAP Help: Draft Handling](https://help.sap.com/docs/abap-cloud/abap-rap/draft-capability#draft-actions) |

---