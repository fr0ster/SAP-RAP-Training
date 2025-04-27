# 📅 Розширений конспект: Feature Control у SAP ABAP RAP (Markdown, 🇺🇦)

## Зміст (TOC)

- [Що таке Feature Control?](#що-таке-feature-control)
- [Навіщо використовувати Feature Control?](#навіщо-використовувати-feature-control)
- [Feature Control під час розробки та під час виконання](#feature-control-під-час-розробки-та-під-час-виконання)
- [Оголошення Feature Control у Behavior Definition](#оголошення-feature-control-у-behavior-definition)
- [Імплементація Feature Control у Behavior Implementation](#імплементація-feature-control-у-behavior-implementation)
- [Кращі практики Feature Control](#кращі-практики-feature-control)
- [Типові сценарії](#типові-сценарії)
- [Типові помилки](#типові-помилки)
- [Посилання](#посилання)

---

## 🔍 Що таке Feature Control?

Feature Control у SAP ABAP RAP дозволяє динамічно активувати або деактивувати окремі функції чи дії екземплярів бізнес-об'єктів залежно від конкретних умов.

Це забезпечує тонке налаштування поведінки екземплярів під час виконання без зміни базових даних.

> **Приклад:** Активація кнопки «Скасувати замовлення» тільки коли статус замовлення — «Відправлено», а не «Завершено».

[📚 Детальніше: Developing Feature Control | SAP Help Portal](https://help.sap.com/docs/abap-cloud/abap-rap/developing-feature-control)

---

## 🎉 Навіщо використовувати Feature Control?

- **Динамічна поведінка**: регулювання доступних дій користувача на основі поточних умов.
- **Керування користувачем**: запобігання неправильним діям.
- **Безпека та відповідність**: обмеження доступу до важливих операцій.
- **Оптимізація продуктивності**: зменшення зайвих перевірок на сервері.

[📖 Додаткове читання: Developing Feature Control | SAP Help Portal](https://help.sap.com/docs/abap-cloud/abap-rap/developing-feature-control)

---

## 🌐 Feature Control під час розробки та під час виконання

| Аспект       | Під час розробки                         | Під час виконання                             |
| ------------ | ---------------------------------------- | --------------------------------------------- |
| **Визначення** | Оголошення можливостей у Behavior Definition | Динамічне управління для кожного екземпляра  |
| **Фокус**      | Структурні можливості                  | Умовна доступність                           |
| **Приклад**    | Оголошення дії «Скасувати»             | Деактивація дії якщо замовлення вже завершено |

> Під час розробки: «Чи може ця функція взагалі існувати?»  
> Під час виконання: «Чи доступна ця функція зараз?»

[🔗 Детальніше: Developing Feature Control | SAP Help Portal](https://help.sap.com/docs/abap-cloud/abap-rap/developing-feature-control)

---

## 🔧 Оголошення Feature Control у Behavior Definition

Оголошення екземплярних функцій здійснюється у файлах **behavior definition** (`.behavior`).

```abap
define behavior for /DMO/R_Travel_D alias Travel
{
  association _Booking { create ( features : instance ); }

  field ( numbering : managed, readonly ) TravelUUID;
  field ( readonly ) TravelID, OverallStatus, TotalPrice;
  field ( mandatory ) CustomerID, AgencyID, BeginDate, EndDate, CurrencyCode;
  field ( features : instance ) BookingFee;

  action ( features : instance ) acceptTravel result [1] $self;
  action ( features : instance ) rejectTravel result [1] $self;
  action ( features : instance ) deductDiscount parameter /DMO/A_TRAVEL_DISCOUNT result [1] $self;
}
```

**Примітки:**

- Додавання `( features : instance )` позначає поля, дії та асоціації як динамічно контрольовані.
- Стан (активовано/деактивовано/read-only) визначається під час виконання.

[📑 Офіційний гайд: Developing Feature Control | SAP Help Portal](https://help.sap.com/docs/abap-cloud/abap-rap/developing-feature-control)

---

## 👩‍💼 Імплементація Feature Control у Behavior Implementation

Логіка реалізується в класі **behavior implementation**, методі `GET INSTANCE FEATURES`:

```abap
METHODS get_instance_features
  IMPORTING keys REQUEST requested_features
  RESULT result_features
  FAILED failed.
```

### Приклад реалізації

```abap
METHOD get_instance_features.

  READ ENTITIES OF /DMO/R_Travel_D IN LOCAL MODE
    ENTITY Travel
      FIELDS ( OverallStatus )
      WITH CORRESPONDING #( keys )
    RESULT DATA(travels)
    FAILED failed.

  result = VALUE #( FOR ls_travel IN travels
    ( %tky = ls_travel-%tky

      %field-BookingFee      = COND #( WHEN ls_travel-OverallStatus = travel_status-accepted
                                       THEN if_abap_behv=>fc-f-read_only
                                       ELSE if_abap_behv=>fc-f-unrestricted )
      %action-acceptTravel   = COND #( WHEN ls_travel-OverallStatus = travel_status-accepted
                                       THEN if_abap_behv=>fc-o-disabled
                                       ELSE if_abap_behv=>fc-o-enabled )
      %action-rejectTravel   = COND #( WHEN ls_travel-OverallStatus = travel_status-rejected
                                       THEN if_abap_behv=>fc-o-disabled
                                       ELSE if_abap_behv=>fc-o-enabled )
      %action-deductDiscount = COND #( WHEN ls_travel-OverallStatus = travel_status-accepted
                                       THEN if_abap_behv=>fc-o-disabled
                                       ELSE if_abap_behv=>fc-o-enabled )
    ) ).

ENDMETHOD.
```

---

## 🔄 Кращі практики Feature Control

| Практика                                    | Причина                                  |
| ------------------------------------------- | ---------------------------------------- |
| Простота логіки                             | Уникнення складних залежностей           |
| Мінімізація доступу до бази                 | Попереднє завантаження необхідних полів  |
| Документація контрольованих функцій         | Простота підтримки                       |
| Розділення перевірок функцій і валідацій    | Уникнення плутанини                      |
| Послідовність результатів                   | Запобігання помилкам на фронтенді        |

---

## 💡 Типові сценарії

- **Дії**: активація дій «Скасувати», «Затвердити» за статусом.
- **Редагування полів**: поля стають read-only чи редагованими динамічно.

---

## ⚠️ Типові помилки

- Жорстке кодування доступності функцій.
- Ігнорування продуктивності.
- Змішування відповідальностей.

---

## 🔗 Посилання

- [Developing Feature Control | SAP Help Portal](https://help.sap.com/docs/abap-cloud/abap-rap/developing-feature-control)

---