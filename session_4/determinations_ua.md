# 📅 Розширений конспект: Determinations в SAP RAP (Markdown, 🇺🇦)

## Структура / Зміст (TOC)

- [Що таке Determination у RAP?](#-що-таке-determination-в-sap-rap)
- [Типи Determinations](#-типи-determinations)
- [Типи імплементації Determinations](#-типи-імплементації-determinations)
- [Оголошення Determinations в Behavior Definition](#-оголошення-determinations-в-behavior-definition)
- [Імплементація Determinations у Behavior Implementation](#-імплементація-determinations-у-behavior-implementation)
- [Коли слід (і не слід) використовувати Determination](#-коли-слід-і-не-слід-використовувати-determination)
- [Кращі практики](#-кращі-практики)
- [Типові сценарії використання](#-типові-сценарії-використання)
- [Типові помилки при використанні Determination](#-типові-помилки-при-використанні-determination)

---

### 💪 Що таке Determination в SAP RAP

**Determination** — це автоматичний фрагмент бізнес-логіки, який виконується під час зміни стану об'єкта без прямого виклику. [SAP Help](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)

Використовується для:
- Заповнення залежних полів;
- Ініціалізації значень при створенні об'єкта;
- Обчислень, що залежать від інших полів.

> 📢 Determinations — "пасивні" обробки, що автоматично реагують на зміни.

---

### 🔢 Типи Determinations

| Тип | Опис | Приклад |
|:---|:-----|:--------|
| **Create** | Спрацьовує при створенні об'єкта. | Встановлення `currency_code` за замовчуванням. |
| **Update** | При зміні полів. | Перерахунок `total_amount` при зміні `quantity` або `price`. |
| **Before Save** | Перед записом у БД. | Встановити `finalized_flag` перед комітом. |
| **After Modify** | Після змін в пам'яті. | Логування змін для аналітики. |

> 📚 [SAP Help: Determination Types](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)

---

### 👩‍💻 Типи імплементації Determinations

| Тип імплементації | Опис | Приклад |
|:------------------|:-----|:--------|
| **Instance Determination** | Для конкретної інстанції (%key). | Перерахунок вартості замовлення. |
| **Static Determination** | Без інстанції (рідко). | Генерація унікального номера документа. |
| **Field-triggered Determination** | При зміні певних полів. | Перерахунок податків при зміні суми. |
| **Lifecycle-triggered Determination** | На фазах життєвого циклу. | Ініціалізація статусу при створенні. |

> 📚 [SAP Help: Determination Implementation](https://help.sap.com/docs/abap-cloud/abap-rap/implement-determinations)

---

### ✏️ Оголошення Determinations в Behavior Definition

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

### 🛠️ Імплементація Determinations у Behavior Implementation

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

### 🧠 Коли слід (і не слід) використовувати Determination

| Використовувати | Не використовувати |
|:----------------|:-------------------|
| Заповнення складених/обчислюваних полів | Просте встановлення постійних значень через CDS Default |
| Перерахунок залежних значень | Статусні переходи — краще через Actions |
| Ініціалізація даних при створенні | Валідація даних — краще через Validations |
| Адаптація логіки у brownfield | Пряме управління процесами (Actions краще) |

> 📚 [SAP Best Practices: RAP Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)

---

### ✅ Кращі практики

| ✅ Практика | 🔍 Чому? |
|:-----------|:---------|
| Використовувати для обчислень, а не для постійних значень | Оптимізація підтримки коду |
| Прив'язувати до конкретних полів | Підвищення продуктивності |
| Уникати ланцюгових викликів Determination | Запобігання нескінченним циклам |
| Стежити за транзакційною цілісністю | Контроль узгодженості даних |
| Документувати Field Triggers | Полегшення підтримки |

---

# 🛠️ Типові сценарії використання

## ✅ Greenfield сценарії

| Сценарій | Приклад | Примітка |
|:---------|:--------|:---------|
| Обчислення суми на основі кількості і ціни | Після зміни `quantity` або `price` перерахувати `total_amount` через Determination. | [SAP Help: Field Control and Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/field-control) |
| Динамічне встановлення валюти на основі країни клієнта | При зміні `customer_id`, отримати валюту клієнта з довідника і підставити в `currency_code`. | Через `on modify field customer_id`. |
| Автоматичне встановлення дати замовлення при створенні | При створенні встановити `order_date = sy-datum`. | Через Determination `on create`. |

> ❗ Статус `Draft` обробляється системно і не потребує окремої Determination.

## ✅ Brownfield сценарії

| Сценарій | Приклад | Примітка |
|:---------|:--------|:---------|
| Перерахунок залежних полів у старих таблицях | Після оновлення `gross_amount` автоматично оновити `net_amount` за старими правилами. | |
| Витяг значень із lookup-таблиць | Автоматичне підставлення опису продукту за `product_id` при зміні запису. | |
| Форматування даних при збереженні | Перетворення старого формату дати YYMMDD у стандартний YYYYMMDD при записі. | |

---

# ❌ Типові помилки при використанні Determination

| Помилка | Пояснення |
|:--------|:----------|
| Використання Determination для простих дефолтів | Краще робити через `@DefaultValue`. |
| Зациклення змін тригерних полів | Викликає нескінченний ланцюг викликів. |
| Перевантаження валідаціями | Має бути через Validations, не через Determinations. |
| Відсутність документації полів-тригерів | Ускладнює супровід проекту. |

---

# ⚡ Приклади некоректного використання Determinations

## ❌ Приклад: Неправильна установка дефолтів

```abap
METHOD set_default_country.
  LOOP AT keys INTO DATA(ls_key).
    UPDATE zsalesorder
      SET country = 'UA'
      WHERE salesorder_id = @ls_key-salesorder_id.
  ENDLOOP.
ENDMETHOD.
```
> ❗ Це краще вирішувати через CDS `@DefaultValue: 'UA'`.

---

# 📖 Витяги з офіційної документації SAP

> "Determinations should automatically derive or adjust attributes during a transactional lifecycle without explicit user interaction."  
> — [SAP RAP Documentation](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)

> "Default values should primarily be defined in the CDS model where possible to avoid redundant coding."  
> — [SAP RAP Best Practices](https://help.sap.com/docs/abap-cloud/abap-rap/best-practices)

> "Use create determinations for setting derived attributes at instance creation time; use modify or before-save determinations for derived values that depend on changes made during the transaction."  
> — [SAP RAP Best Practices](https://help.sap.com/docs/abap-cloud/abap-rap/best-practices)

---

# ✅ Чекліст перевірки якісної Determination

| Питання | Примітка |
|:--------|:---------|
| Чи прив'язана до подій або змін полів? | Має бути `on create`, `on modify field ...`. |
| Чи не змінює поля, що тригерять саму Determination? | Щоб уникнути циклів. |
| Чи обчислює складні залежності, а не просто дефолти? | Просте заповнення — краще в CDS. |
| Чи має документований перелік тригерних полів? | Для прозорої підтримки. |

---

# ✨ Добрі приклади реалізації Determinations

## ✅ Приклад: Коректний перерахунок Total

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
> ✔️ Добра практика: тільки обчислення без змін тригерних полів.

---