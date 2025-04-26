# 📅 Розширений конспект: Actions в SAP RAP (Markdown, 🇺🇦)

## Структура / Зміст (TOC)
- [Що таке Action у RAP?](#-що-таке-action-в-sap-rap)
- [Типи Actions](#-типи-actions)
- [Типи імплементації Actions](#-типи-імплементації-actions)
- [Оголошення Actions в Behavior Definition](#-оголошення-actions-в-behavior-definition)
- [Імплементація Actions у Behavior Implementation](#-імплементація-actions-у-behavior-implementation)
- [Транзакційна поведінка Actions](#-транзакційна-поведінка-actions)
- [Actions у чернеткових об'єктах](#-actions-у-чернеткових-об'єктах)
- [Валідація Actions](#-валідація-actions)
- [Кращі практики](#-кращі-практики)

### 💪 Що таке Action в SAP RAP
**Action** — це явна, нестандартна операція зміни Business Object, що не входить до CRUD (стандартні) операцій.

> 📢 Стандартні CRUD операції: Read, Create, Update, Delete, Lock. Всі інші зміни через Actions вважаються нестандартними.

> 🔍 Приклади: Set On Hold, Release From Hold.

### 🔢 Типи Actions
| Тип | Опис |
|:----|:-----|
| **Internal Action** | Викликається лише всередині BO |
| **Static Action** | Виконується без прив'язки до конкретної інстанції |
| **Repeatable Action** | Може виконуватись кілька разів для однієї інстанції |
| **Factory Action** | Створює нові інстанції |
| **Save Action** | Виконується лише під час Save-послідовності |

### 👩‍💻 Типи імплементації Actions
| Тип імплементації | Опис |
|:------------------|:-----|
| **Instance Action** | Працює з `%key` для конкретної інстанції |
| **Static Action** | Працює через `%cid`, без прив'язки до інстанції |
| **Action з параметрами** | Використовує структуру `%param` для вхідних даних |
| **Action з результатним типом** | Повертає нові інстанції через `%cid` |
| **Factory Action** | Створює нові об'єкти через `%cid` і `%cid_ref` |

### ✏️ Оголошення Actions в Behavior Definition
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

### 🛠️ Імплементація Actions у Behavior Implementation
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

### 🛠️ Транзакційна поведінка Actions
- Actions виконуються всередині транзакції Save.
- Якщо Action змінює дані, обов'язково потрібно повертати `result`.
- Save Actions викликаються лише під час Save Sequence.

### 📝 Actions у чернеткових об'єктах
```abap
draft action submit result [1] $self;
draft action discard result [1] $self;
```

### 📋 Валідація Actions
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

### ✅ Кращі практики
- Завжди повертайте `$self` при зміні об'єкта для оновлення UI.
- Виконуйте бізнес-валідації в `validate_action` перед змінами.
- Уникайте змін ключових полів (`key fields`, `created_by`, `created_at`) без потреби.
- Використовуйте ETag для уникнення конфліктів паралельного оновлення.
- Правильно керуйте транзакціями (`ROLLBACK`) для цілісності даних.
- Чітко розділяйте логіку для чернеткових і активних інстанцій.

---

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
An **Action** is an explicit, non-standard operation on a Business Object that is not part of the CRUD (Create, Read, Update, Delete) operations.

> 📢 Standard CRUD operations: Read, Create, Update, Delete, Lock. Any other modification via Actions is considered non-standard.

> 🔍 Examples: Set On Hold, Release From Hold.

### 🔢 Action Types
| Type | Description |
|:-----|:------------|
| **Internal Action** | Called only internally within the BO |
| **Static Action** | Executed without binding to a specific instance |
| **Repeatable Action** | Can be triggered multiple times for the same instance |
| **Factory Action** | Creates new instances |
| **Save Action** | Executed only during the Save sequence |

### 👩‍💻 Action Implementation Types
| Implementation Type | Description |
|:---------------------|:------------|
| **Instance Action** | Works with `%key` |
| **Static Action** | Uses `%cid` |
| **Action with Parameters** | Accepts `%param` |
| **Action with Result Type Entity** | Returns newly created entities via `%cid` |
| **Factory Action** | Creates new instances with `%cid` and `%cid_ref` |

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
- Actions are executed within Save Transactions.
- If an Action modifies data, it must return `result`.
- Save Actions are only triggered during Save Sequence.

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
- Always return `$self` when updating the object to ensure correct UI update.
- Perform business validations in `validate_action` before changing the object state.
- Avoid modifying key fields (`key fields`, `created_by`, `created_at`) unless necessary.
- Use ETag to prevent parallel update conflicts.
- Manage transactions properly (`ROLLBACK`) to ensure data consistency.
- Separate handling of Draft and Active instances clearly.

