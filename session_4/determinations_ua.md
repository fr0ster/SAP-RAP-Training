# 📅 Розширений конспект: Determinations у SAP RAP (Markdown, 🇺🇦)

## Зміст

- [Що таке Determination у RAP?](#що-таке-determination-у-rap)
- [Типи Determinations](#типи-determinations)
- [Оголошення Determinations у Behavior Definition](#оголошення-determinations-у-behavior-definition)
- [Імплементація Determinations у Behavior Implementation](#імплементація-determinations-у-behavior-implementation)
- [Коли використовувати Determinations](#коли-використовувати-determinations)
- [Кращі практики](#кращі-практики)
- [Типові сценарії використання](#типові-сценарії-використання)
- [Поширені помилки при використанні Determinations](#поширені-помилки-при-використанні-determinations)
- [Ключові концепції з документації SAP](#ключові-концепції-з-документації-sap)
- [Чеклист якості Determinations](#чеклист-якості-determinations)
- [Field-Based Determinations](#field-based-determinations)
- [Посилання](#посилання)

---

# 💪 Що таке Determination у RAP

**Determination** у SAP RAP — це пасивна, автоматично викликана реалізація, яка коригує або виводить значення атрибутів упродовж життєвого циклу транзакції (створення, зміна полів, перед збереженням), без прямого виклику користувачем.

> 🖊️ Determinations запускаються **тільки при зміні даних**.

Використовуються для:
- Обчислення атрибутів
- Заповнення залежних полів
- Ініціалізації значень при створенні екземпляра

> 📊 [Посилання: SAP Help — Determinations Overview](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)

---

# 🔢 Типи Determinations

| Тип | Тригер | Приклад |
|:-----|:--------|:--------|
| **On Create** | Створення нового екземпляра | `status = 'New'` при створенні |
| **On Modify** | При зміні зазначених полів | Перерахунок `total_price` при зміні `quantity` або `unit_price` |
| **Before Save** | Перед збереженням у БД після всіх змін | Генерація унікального ID, аудитні поля |

> 🖊️ After Modify Determination **не існує** в RAP.

> 📊 [Посилання: SAP Help — Developing Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/developing-determinations)

---

# ✏️ Оголошення Determinations у Behavior Definition

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

> 📊 [Посилання: SAP Help — Defining Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/defining-determinations)

---

# 🛠️ Імплементація Determinations у Behavior Implementation

(приклади імплементації аналогічні до оригіналу)

---

# 🧐 Коли використовувати Determinations

| Використовувати | Не використовувати |
|:-----------------|:-------------------|
| Обчислення або залежні поля | Статичні константи (CDS) |
| Коригування даних під час транзакції | Валідації — краще Validations |
| Перерахунки залежні від полів | Контроль процесу — краще Actions |

---

# ✅ Кращі практики

(аналогічно до оригіналу)

---

# 📅 Типові сценарії використання

(аналогічно до оригіналу)

---

# ❌ Поширені помилки при використанні Determinations

(аналогічно до оригіналу)

---

# 📖 Ключові концепції з документації SAP

(аналогічно до оригіналу)

---

# ✅ Чеклист якості Determinations

(аналогічно до оригіналу)

---

# 📈 Field-Based Determinations

Determinations, які запускаються **тільки при зміні визначених полів**. Це дозволяє уникнути зайвих перерахунків чи операцій з базою даних.

```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
{
  determination update_tax on modify field quantity, unit_price;
}
```

> 📊 [Посилання: SAP Help — Implementing Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/implementing-determinations)

---

# 📚 Посилання

| Topic | Link |
|:------|:-----|
| [Determinations Overview](https://help.sap.com/docs/abap-cloud/abap-rap/determinations) |
| [Developing Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/developing-determinations) |
| [Defining Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/defining-determinations) |
| [Implementing Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/implementing-determinations) |
| [Field-Based Determination](https://help.sap.com/docs/abap-cloud/abap-rap/field-based-determination) |
| [Determination and Validation Modelling](https://help.sap.com/docs/abap-cloud/abap-rap/determination-and-validation-modelling) |