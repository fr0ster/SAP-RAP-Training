## 📅 Розширений конспект: Concurrency Control в SAP RAP (Markdown, 🇺🇦)

### Основні теми
- Optimistic Locking
- Pessimistic Locking
- OCC через `last_changed_at`
- Behavior Definition: `lock master; validate_lock;`
- Managed vs Unmanaged RAP
- Методи LOCK, VALIDATE_LOCK, UPDATE

### 🔄 Що таке Concurrency Control в RAP
Concurrency Control в SAP RAP забезпечує що паралельні зміни об'єктів не пошкодять дані.

### ✅ Базовий сценарій (Managed RAP)

1. **CDS Entity**:
```abap
@Semantics.systemLastChangedAt: true
last_changed_at
```

2. **Behavior Definition**:
```abap
managed implementation in class ZBP_SalesOrder unique;
lock master;
validate_lock;
```

- Етаг (ETag) автоматично створюється з `last_changed_at`
- RAP сам перевіряє If-Match

### ❌ Сценарій не базовий (Unmanaged RAP)

Треба реалізувати методи:

- `LOCK`: ручна перевірка last_changed_at
- `VALIDATE_LOCK`: додаткові валідації
- `UPDATE`, `DELETE`, `CREATE`: явне збереження даних


### ✨ Що стандартно перевіряєть RAP

- Чи співпадає ETag (`last_changed_at`)
- Чи вірні дані загалом (`validate_lock`)
- Чи валідні бізнес-умови (`validation`)
- Чи вдалося зберегти (`save`) або ролбек

---

## 📅 Extended Summary: Concurrency Control in SAP RAP (Markdown, 🇬🇧)

### Key Topics
- Optimistic Locking
- Pessimistic Locking
- OCC via `last_changed_at`
- Behavior Definition: `lock master; validate_lock;`
- Managed vs Unmanaged RAP
- Methods: LOCK, VALIDATE_LOCK, UPDATE

### 🔄 What is Concurrency Control in RAP?
Concurrency Control in SAP RAP ensures that parallel changes to business objects do not corrupt data.

### ✅ Basic Scenario (Managed RAP)

1. **CDS Entity**:
```abap
@Semantics.systemLastChangedAt: true
last_changed_at
```

2. **Behavior Definition**:
```abap
managed implementation in class ZBP_SalesOrder unique;
lock master;
validate_lock;
```

- ETag is auto-generated from `last_changed_at`
- RAP automatically checks If-Match during updates

### ❌ Non-Basic Scenario (Unmanaged RAP)

You must manually implement:

- `LOCK`: manual check of `last_changed_at`
- `VALIDATE_LOCK`: business validations
- `UPDATE`, `DELETE`, `CREATE`: manual data persistence


### ✨ What RAP Framework Checks Automatically

- Whether ETag (`last_changed_at`) matches
- Whether data is valid (`validate_lock`)
- Whether business conditions are valid (`validation`)
- Whether save succeeds or rollback is needed

---

