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

- ETag автоматично створюється з `last_changed_at`
- RAP сам перевіряє If-Match

### ❌ Сценарій не базовий (Unmanaged RAP)

Треба реалізувати методи:

- `LOCK`: ручна перевірка last_changed_at
- `VALIDATE_LOCK`: додаткові валідації
- `UPDATE`, `DELETE`, `CREATE`: явне збереження даних

### 📈 Приклади

**Managed RAP:**
- Поле `last_changed_at` в CDS.
- `lock master; validate_lock;` в Behavior Definition.
- Без якоїсь додаткової імплементації методів.

**Unmanaged RAP:**
```abap
METHOD lock_salesorder.
  LOOP AT entities INTO DATA(ls_entity).
    SELECT SINGLE last_changed_at FROM zsalesorder
      WHERE salesorder_id = @ls_entity-salesorder_id
      INTO @DATA(lv_db_last_changed_at).
    IF ls_entity-last_changed_at <> lv_db_last_changed_at.
      APPEND VALUE #( %key = ls_entity-%key
                      %msg = new_message( id = 'ZSO' number = '003' v1 = ls_entity-salesorder_id ) )
             TO result-failed.
    ENDIF.
  ENDLOOP.
ENDMETHOD.
```

### 🛍️ Діаграма: Сервісний піпелайн Save + OCC

```plaintext
READ Entity
  └→ Return last_changed_at
User Edits Entity
User Sends PATCH (If-Match: ETag)
  └→ RAP:
     └→ Compare ETag with DB
     └→ Call VALIDATE_LOCK (optional additional checks)
     └→ If OK → SAVE + COMMIT
     └→ If Fail → Error (E_CONCURRENCY_FAILED) + ROLLBACK
```

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

### 📈 Examples

**Managed RAP:**
- `last_changed_at` field in CDS.
- `lock master; validate_lock;` in Behavior Definition.
- No additional implementation required.

**Unmanaged RAP:**
```abap
METHOD lock_salesorder.
  LOOP AT entities INTO DATA(ls_entity).
    SELECT SINGLE last_changed_at FROM zsalesorder
      WHERE salesorder_id = @ls_entity-salesorder_id
      INTO @DATA(lv_db_last_changed_at).
    IF ls_entity-last_changed_at <> lv_db_last_changed_at.
      APPEND VALUE #( %key = ls_entity-%key
                      %msg = new_message( id = 'ZSO' number = '003' v1 = ls_entity-salesorder_id ) )
             TO result-failed.
    ENDIF.
  ENDLOOP.
ENDMETHOD.
```

### 🛍️ Diagram: Save + OCC Pipeline

```plaintext
READ Entity
  └→ Return last_changed_at
User Edits Entity
User Sends PATCH (If-Match: ETag)
  └→ RAP:
     └→ Compare ETag with DB
     └→ Call VALIDATE_LOCK (optional additional checks)
     └→ If OK → SAVE + COMMIT
     └→ If Fail → Error (E_CONCURRENCY_FAILED) + ROLLBACK
```

---

