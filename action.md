## 📅 Розширений конспект: Actions в SAP RAP (Markdown, 🇺🇦)

### Основні теми
- [Що таке Action у RAP?](https://help.sap.com/docs/abap-cloud/abap-rap/actions)
- [Types: Instance Actions vs Static Actions](https://help.sap.com/docs/abap-cloud/abap-rap/actions#instance-and-static-actions)
- [Defining Actions in Behavior Definition](https://help.sap.com/docs/abap-cloud/abap-rap/actions#define-actions)
- [Implementing Actions in Behavior Implementation](https://help.sap.com/docs/abap-cloud/abap-rap/actions#implement-actions)
- [Transactional behavior of Actions](https://help.sap.com/docs/abap-cloud/abap-rap/actions#transactional-aspects)
- [Draft-enabled Actions](https://help.sap.com/docs/abap-cloud/abap-rap/draft-capability#draft-actions)
- [Action Validations](https://help.sap.com/docs/abap-cloud/abap-rap/validations)

### 🔄 Що таке Action в SAP RAP
**Action** — це explicit (експліцитна, явно викликана) операція над об'єктом, яка є **нестандартною модифікаційною операцією** (non-standard modify operation). Вона не є частиною стандартних операцій, таких як читання (Read), створення (Create), оновлення (Update) або видалення (Delete) бізнес-об'єктів.

> 📢 Стандартні операції в RAP: Read, Create, Update, Delete, Lock.
> Всі інші зміни через Actions вважаються нестандартними модифікаціями.

> 🔍 **Приклади:** Approve Order, Reject Request, Post Invoice.

### 🔢 Типи Actions (Behavior Definition)

| Тип | Опис |
|:----|:-----|
| **Instance Action** | Прив'язана до конкретного об'єкта (ID) |
| **Static Action** | Виконується без прив'язки до конкретного ID |
| **Internal Action** | Може бути викликана лише зсередини BO (наприклад, із determination) |
| **Repeatable Action** | Може бути виконана кілька разів на одному екземплярі в одному запиті |
| **Factory Action** | Створює нові екземпляри об'єкта |
| **Save Action** | Викликається тільки під час фази Save (FINALIZE або ADJUST_NUMBERS)

### ✏️ Визначення Actions у Behavior Definition
```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
{
  action approve result [1] $self;
  action reject  result [1] $self;
}
```

### 👩‍💻 Реалізація Actions у Behavior Implementation

| Тип Імплементації | Опис |
|:------------------|:-----|
| **Instance Action** | Імпортує `%key` і опціонально `%cid_ref` |
| **Static Action** | Імпортує `%cid` як operation ID |
| **Action with Parameters** | Імпортує `%param` для вхідних параметрів |
| **Action with Result Entity** | Імпортує `%cid` для новостворених екземплярів |
| **Factory Action** | Імпортує `%cid` і `%cid_ref`, створює нові об'єкти |

```abap
METHOD approve.
  LOOP AT keys INTO DATA(ls_key).
    UPDATE zsalesorder
      SET status = 'APPROVED'
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.
```

### 🛠️ Транзакційна поведінка Actions
- Actions виконуються в рамках транзакції Save.
- Якщо Action змінює дані, потрібно повернути `result`.
- Save Actions дозволені тільки в Save Sequence.

### 📝 Draft-enabled Actions
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
  action approve result [1] $self;
  validate action approve on save { my_validation; }
}
```

### ✅ Best Practices
- Завжди оголошуй результат `$self`, якщо об'єкт змінюється, щоб UI отримав актуальні дані.
- Валідуй бізнес-умови в `validate_action` перед оновленням стану об'єкта.
- Уникай модифікації базових полів (`key fields`, `created_by`, `created_at`) без вагомих причин.
- Для важливих бізнес-процесів додавай перевірку ETag для запобігання втраті змін.
- Використовуй транзакційні контрольні механізми (`ROLLBACK`) при невдачах, щоб зберегти консистентність даних.
- Для draft-enabled об'єктів розрізняй обробку чернеток і активних записів окремо.

---

## 📅 Extended Summary: Actions in SAP RAP (Markdown, 🇬🇧)

### Main Topics
- [What are Actions in RAP?](https://help.sap.com/docs/abap-cloud/abap-rap/actions)
- [Types: Instance Actions vs Static Actions](https://help.sap.com/docs/abap-cloud/abap-rap/actions#instance-and-static-actions)
- [Defining Actions in Behavior Definition](https://help.sap.com/docs/abap-cloud/abap-rap/actions#define-actions)
- [Implementing Actions in Behavior Implementation](https://help.sap.com/docs/abap-cloud/abap-rap/actions#implement-actions)
- [Transactional behavior of Actions](https://help.sap.com/docs/abap-cloud/abap-rap/actions#transactional-aspects)
- [Draft-enabled Actions](https://help.sap.com/docs/abap-cloud/abap-rap/draft-capability#draft-actions)
- [Action Validations](https://help.sap.com/docs/abap-cloud/abap-rap/validations)

### 🔄 What are Actions in SAP RAP
**Action** — an explicit non-standard modify operation on a business object, triggered by the user (UI or API).

> 📢 Standard operations in RAP cover: Read, Create, Update, Delete, Lock.
> All other changes via Actions are considered non-standard modifications.

> 🔍 **Examples:** Approve Order, Reject Request, Post Invoice.

### 🔢 Types of Actions (Behavior Definition)

| Type | Description |
|:-----|:------------|
| **Instance Action** | Linked to a specific object (ID) |
| **Static Action** | Executed without reference to a specific ID |
| **Internal Action** | Executed only internally within the BO (e.g., from a determination) |
| **Repeatable Action** | Can be executed multiple times on the same instance within one request |
| **Factory Action** | Creates new entity instances |
| **Save Action** | Can be triggered only during Save Sequence (FINALIZE or ADJUST_NUMBERS)

### ✏️ Defining Actions in Behavior Definition
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

| Implementation Type | Description |
|:---------------------|:------------|
| **Instance Action** | Imports `%key` and optionally `%cid_ref` |
| **Static Action** | Imports `%cid` as operation ID |
| **Action with Parameters** | Imports `%param` for input parameters |
| **Action with Result Entity** | Imports `%cid` to identify newly created entities |
| **Factory Action** | Imports `%cid` and `%cid_ref` for instance creation |

```abap
METHOD approve.
  LOOP AT keys INTO DATA(ls_key).
    UPDATE zsalesorder
      SET status = 'APPROVED'
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.
```

### 🛠️ Transactional behavior of Actions
- Actions are executed within Save transactions.
- Actions modifying data must return `result`.
- Save Actions can only be triggered during Save Sequence.

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
  action approve result [1] $self;
  validate action approve on save { my_validation; }
}
```

### ✅ Best Practices
- Always return `$self` if the object is updated, to refresh UI data.
- Perform business validations in `validate_action` before changing object state.
- Avoid modifying fundamental fields (`key fields`, `created_by`, `created_at`) unless necessary.
- For critical processes, ensure ETag checking to prevent concurrent modification issues.
- Use transactional control (`ROLLBACK`) on failures to maintain data consistency.
- For draft-enabled objects, handle drafts and active instances separately.

---

