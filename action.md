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

**Action** — це окрема бізнес-операція, яка виконує зміну стану або поведінки Business Object поза межами стандартних CRUD-операцій (Create, Read, Update, Delete, Lock).

На відміну від стандартних операцій, Actions використовуються для:

- зміни статусу об'єкта (наприклад, перевести замовлення в режим "Очікування"),
- виконання складної бізнес-логіки (наприклад, масове оновлення або підтвердження декількох записів),
- запуску специфічних процесів (наприклад, ініціалізація розрахунків, копіювання даних, масове створення залежних об'єктів).

> 📢 Стандартні CRUD операції: Read, Create, Update, Delete, Lock — працюють із базовим управлінням записами.
> Actions розширюють функціональність бізнес-об'єктів через специфічні сценарії змін та обробки даних.

> 🔍 Приклади нестандартних Actions:
>
> - Set On Hold (перевести замовлення в режим очікування),
> - Release From Hold (зняти замовлення з очікування),
> - Approve (затвердити запис),
> - Recalculate Pricing (перерахувати ціни).

### 🔢 Типи Actions

| Тип                   | Опис                                                                                                 |
| --------------------- | ---------------------------------------------------------------------------------------------------- |
| **Internal Action**   | Викликається лише всередині логіки Business Object, недоступна для зовнішніх викликів.               |
| **Static Action**     | Виконується без прив'язки до конкретної інстанції, використовуючи `%cid` для ідентифікації виклику.  |
| **Repeatable Action** | Дозволяє багаторазове виконання для тієї ж інстанції протягом однієї транзакції за допомогою `%cid`. |
| **Factory Action**    | Створює нові інстанції в пам'яті з тимчасовими ідентифікаторами `%cid`, без негайного запису в базу. |
| **Save Action**       | Автоматично викликається у фазі збереження для фіналізації змін перед комітом.                       |

> ℹ️ `%cid` — тимчасовий ідентифікатор створених або змінених об'єктів під час транзакції до моменту їх реального збереження в базі даних. [Докладніше](https://help.sap.com/docs/abap-cloud/abap-rap/actions#define-actions)

### 👩‍💻 Типи імплементації Actions

| Тип імплементації               | Опис                                                                                      |
| ------------------------------- | ----------------------------------------------------------------------------------------- |
| **Instance Action**             | Працює з конкретною інстанцією через `%key`, обов'язково прив'язана до існуючого запису.  |
| **Static Action**               | Викликається без конкретної інстанції, `%cid` служить для ідентифікації операції у сесії. |
| **Action з параметрами**        | Приймає структуру `%param`, що містить вхідні дані для обробки всередині Action.          |
| **Action з результатним типом** | Створює об'єкти у пам'яті за допомогою `%cid`, що будуть фізично записані після Save.     |
| **Factory Action**              | Масово створює нові об'єкти, з можливістю встановлення зв'язків через %param.             |

### 📚 Службові параметри в SAP RAP Actions

| Символ | Опис | Посилання |
|:-------|:-----|:----------|
| **%cid** | Тимчасовий локальний ID створеної або обробленої інстанції під час транзакції до її збереження в базу. Використовується для зв'язку між об'єктами в пам'яті. | [SAP Help: Working with %cid](https://help.sap.com/docs/abap-cloud/abap-rap/actions#define-actions) |
| **%key** | Ідентифікатор існуючої інстанції Business Object, що використовується для доступу або зміни запису в базі. | [SAP Help: Instance Actions and %key](https://help.sap.com/docs/abap-cloud/abap-rap/actions#instance-and-static-actions) |
| **%param** | Структура параметрів, що передається у виклик Action для отримання додаткових вхідних даних. | [SAP Help: Actions with Parameters](https://help.sap.com/docs/abap-cloud/abap-rap/actions#actions-with-parameters) 

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
| ✅ Практика | 🔍 Чому? | 🔗 Офіційне посилання |
|:-----------|:--------|:-----------------------|
| Завжди повертайте `$self` | Щоб UI оновився після зміни об'єкта | [SAP Help: Actions and $self](https://help.sap.com/docs/abap-cloud/abap-rap/actions#define-actions) |
| Виконуйте бізнес-валідації в `validate_action` | Перевірити дані перед збереженням | [SAP Help: Validations in RAP](https://help.sap.com/docs/abap-cloud/abap-rap/validations) |
| Не змінюйте ключові поля без потреби | Порушується ідентифікація запису | [SAP Best Practices: Avoid Updating Key Fields](https://help.sap.com/docs/abap-cloud/abap-rap/behavior-definitions#restrictions) |
| Використовуйте ETag | Захист від конфліктів паралельного оновлення | [SAP Help: Managing Concurrency with ETags](https://help.sap.com/docs/abap-cloud/abap-rap/concurrency-control#etag) |
| Керуйте транзакціями правильно (ROLLBACK) | Забезпечення цілісності даних при помилках | [SAP Help: Transactional Behavior](https://help.sap.com/docs/abap-cloud/abap-rap/actions#transactional-aspects) |
| Розділяйте логіку Draft і Active | Інакше можливі помилки у статусах і збереженні | [SAP Help: Draft Handling](https://help.sap.com/docs/abap-cloud/abap-rap/draft-capability#draft-actions) |

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
| **Instance Action** | Works with `%key` for a specific instance |
| **Static Action** | Uses `%cid`, not tied to a specific instance |
| **Action with Parameters** | Accepts `%param` structure for input data |
| **Action with Result Type Entity** | Creates new instances via `%cid` |
| **Factory Action** | Creates new objects via `%cid` and `%cid_ref` |

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

