## 📅 Розширений конспект: Concurrency Control в SAP RAP (Markdown, 🇺🇦)

### Основні теми
- Optimistic Locking
- Pessimistic Locking
- OCC через `last_changed_at`
- Behavior Definition: `lock master; validate_lock;`
- Managed vs Unmanaged RAP
- Методи LOCK, VALIDATE_LOCK, UPDATE

### 🔄 Що таке Concurrency Control в RAP
Concurrency Control в SAP RAP забезпечує, що паралельні зміни об'єктів не шкідливо впливають на стан даних.

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
Треба реалізувати ручно:
- `LOCK`
- `VALIDATE_LOCK`
- `UPDATE`, `DELETE`, `CREATE`

### 📈 Приклади

**Managed RAP:**
- Поле `last_changed_at` в CDS.
- `lock master; validate_lock;` в Behavior Definition.
- Без додаткової реалізації методів.

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

### 🛍️ Діаграма Save + OCC
```plaintext
READ Entity
  └→ Return last_changed_at
User Edits Entity
User Sends PATCH (If-Match: ETag)
  └→ RAP:
     └→ Compare ETag with DB
     └→ Call VALIDATE_LOCK
     └→ If OK → SAVE + COMMIT
     └→ If Fail → Error + ROLLBACK
```

### 🔢 Checklist
- Поле `last_changed_at` з анотацією
- Behavior Definition має `lock master; validate_lock;`
- Реалізація `LOCK` для unmanaged моделей
- Реалізація додаткових перевірок у `VALIDATE_LOCK`
- Коректна обробка If-Match у клієнті (UI)

### ⚠️ Типові помилки
- Немає `@Semantics.systemLastChangedAt`
- Немає `lock master;` в поведінці
- Пропущено метод `LOCK`
- Неправильна обробка If-Match на UI
- Неправильний Save/Rollback механізм

### 🔹 Best Practices
- Завжди вказуй `last_changed_at`.
- Активуй `lock master; validate_lock;`.
- Використовуй додаткові перевірки у `VALIDATE_LOCK`.
- Комбінуй OCC з draft handling для кращого UX.

---

## 📅 Extended Summary: Concurrency Control in SAP RAP (Markdown, 🇬🇧)

### Main Topics
- Optimistic Locking
- Pessimistic Locking
- OCC via `last_changed_at`
- Behavior Definition: `lock master; validate_lock;`
- Managed vs Unmanaged RAP
- Methods: LOCK, VALIDATE_LOCK, UPDATE

### 🔄 What is Concurrency Control in RAP?
Concurrency Control in SAP RAP ensures that parallel changes to business objects do not corrupt the data.

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

- ETag is automatically generated.
- RAP checks If-Match on update.

### ❌ Non-Basic Scenario (Unmanaged RAP)
- Manual implementation of `LOCK`
- `VALIDATE_LOCK`
- `UPDATE`, `DELETE`, `CREATE`

### 📈 Examples

**Managed RAP:**
- `last_changed_at` in CDS.
- `lock master; validate_lock;` in behavior.

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

### 🛎️ Save + OCC Service Pipeline
```plaintext
READ Entity
  └→ Return last_changed_at
User Edits Entity
User Sends PATCH (If-Match: ETag)
  └→ RAP:
     └→ Compare ETag with DB
     └→ Call VALIDATE_LOCK
     └→ If OK → SAVE + COMMIT
     └→ If Fail → Error + ROLLBACK
```

### 🧲 Checklist
- Field `last_changed_at` with annotation
- `lock master; validate_lock;` present
- Implement `LOCK` for unmanaged RAP
- Additional business checks in `VALIDATE_LOCK`
- Correct handling of If-Match at UI

### ⚠️ Common Mistakes
- Missing `@Semantics.systemLastChangedAt`
- No `lock master;` setting
- No LOCK method in unmanaged
- If-Match incorrectly processed
- Incorrect transaction save/rollback

### 🔹 Best Practices
- Always set `last_changed_at` field.
- Always enable `lock master; validate_lock;`.
- Add complex validations inside `VALIDATE_LOCK`.
- Combine OCC and draft handling when needed.

---