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

### 🔢 Checklist: Що перевіряти для Concurrency Control
- Чи є поле `last_changed_at` з анотацією `@Semantics.systemLastChangedAt`
- Чи включено `lock master; validate_lock;` в Behavior Definition
- Чи реалізовано `LOCK` для Unmanaged RAP
- Чи реалізовано `VALIDATE_LOCK` для додаткових перевірок
- Чи правильно обробляється If-Match на користувацькому UI

### ⚠️ Типові помилки при Concurrency Control
- Не вказано `@Semantics.systemLastChangedAt` на полі
- Не включено `lock master;` в Behavior Definition
- Не реалізовано `LOCK` для Unmanaged RAP
- Невірна обробка If-Match на UI стороні
- Невірна побудова SAVE/ROLLBACK логіки

### 🔹 Best Practices for OCC in RAP
- Завжди додавай `last_changed_at` в Root Entity.
- Відокривай `lock master; validate_lock;` у всіх Behavior Definition.
- Реалізуй кастомні `VALIDATE_LOCK`, якщо є сложні бізнес-валідації.
- Розглядай логіку optimistic та draft handling разом.

---