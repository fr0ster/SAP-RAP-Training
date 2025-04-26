## 📅 Розширений конспект: Actions в SAP RAP (Markdown, 🇺🇦)

### Основні теми
- Що таке Action у RAP?
- Types: Instance Actions vs Static Actions
- Defining Actions in Behavior Definition
- Implementing Actions in Behavior Implementation
- Transactional behavior of Actions
- Draft-enabled Actions
- Action Validations
- Best Practices

### 🔄 Що таке Action в SAP RAP
**Action** — це explicit (експліцитна, явно викликана) операція над об'єктом, що запускається явно користувачем (UI або API).

> 🔍 **Приклади:** Approve Order, Reject Request, Post Invoice.

### 🔢 Types of Actions

| Type | Опис |
|:-----|:-----|
| **Instance Action** | Прив'язана до конкретного об'єкта (ID) |
| **Static Action** | Вільна дія, яка не требує ID |

### 🖊️ Defining Actions in Behavior Definition
```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
{
  action approve result [1] $self;
  action reject  result [1] $self;
}
```
- `result [1] $self;` — Action повертає оновлений об'єкт.

### 👩‍💻 Implementing Actions in Behavior Implementation
```abap
METHOD approve.
  LOOP AT keys INTO DATA(ls_key).
    UPDATE zsalesorder
      SET status = 'APPROVED'
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.
```

### 🛠️ Transactional Behavior of Actions
- Actions виконуються у межах транзакції Save.
- Якщо Action змінює дані, повертай `result`.

### 📝 Draft-enabled Actions
- Для draft-enabled об'єктів дозволені special actions:
```abap
draft action submit result [1] $self;
draft action discard result [1] $self;
```

### 📆 Action Validations
- Валідації для Actions
```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
{
  action approve result [1] $self;
  validate action approve on save { my_validation; }
}
```

### 🔹 Best Practices
- Серед результатів використовуй `$self`
- Додавай validate_action methods
- Обов'язково керуй транзакціями

---

## 📅 Extended Summary: Actions in SAP RAP (Markdown, 🇬🇧)

### Main Topics
- What are Actions in RAP?
- Types: Instance Actions vs Static Actions
- Defining Actions in Behavior Definition
- Implementing Actions in Behavior Implementation
- Transactional behavior of Actions
- Draft-enabled Actions
- Action Validations
- Best Practices

### 🔄 What are Actions in SAP RAP
**Action** — this is an **explicitly invoked** operation on an object, triggered by the user (UI or API).

> 🔍 **Examples:** Approve Order, Reject Request, Post Invoice.

### 🔢 Types of Actions

| Type | Description |
|:-----|:------------|
| **Instance Action** | Linked to a specific object (ID) |
| **Static Action** | Not linked to any ID |

### 🖊️ Defining Actions in Behavior Definition
```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
{
  action approve result [1] $self;
  action reject  result [1] $self;
}
```

### 👩‍💻 Implementing Actions in Behavior Implementation
```abap
METHOD approve.
  LOOP AT keys INTO DATA(ls_key).
    UPDATE zsalesorder
      SET status = 'APPROVED'
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.
```

### 🛠️ Transactional Behavior of Actions
- Actions run within Save transactions.
- If Action modifies data, must return `result`.

### 📝 Draft-enabled Actions
- Special Actions for draft-enabled objects:
```abap
draft action submit result [1] $self;
draft action discard result [1] $self;
```

### 📆 Action Validations
```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
{
  action approve result [1] $self;
  validate action approve on save { my_validation; }
}
```

### 🔹 Best Practices
- Always use `$self` as result if modifying the object
- Implement validate_action methods for checks
- Handle transaction management properly

---