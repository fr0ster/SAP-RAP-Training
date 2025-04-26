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

- ETag is automatically generated from `last_changed_at`
- RAP automatically checks If-Match

### ❌ Non-Basic Scenario (Unmanaged RAP)

You must manually implement:

- `LOCK`: manual last_changed_at check
- `VALIDATE_LOCK`: additional validations
- `UPDATE`, `DELETE`, `CREATE`: explicit data persistence

### 📈 Examples

**Managed RAP:**
- `last_changed_at` field in CDS.
- `lock master; validate_lock;` in Behavior Definition.
- No additional method implementation needed.

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

### 🛍️ Diagram: Save + OCC Service Pipeline

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

### 🔢 Checklist: What to Check for Concurrency Control
- Ensure `last_changed_at` field with `@Semantics.systemLastChangedAt` annotation
- `lock master; validate_lock;` present in Behavior Definition
- Implement `LOCK` method for Unmanaged RAP
- Implement `VALIDATE_LOCK` method for additional consistency checks
- Ensure If-Match is correctly handled at the client/UI level

### ⚠️ Common Mistakes in Concurrency Control
- Missing `@Semantics.systemLastChangedAt` on field
- Missing `lock master;` in Behavior Definition
- No `LOCK` implementation for Unmanaged RAP
- Incorrect If-Match handling on client/UI
- Incorrect SAVE/ROLLBACK transaction management

### 🔹 Best Practices for OCC in RAP
- Always add `last_changed_at` to the Root Entity.
- Always enable `lock master; validate_lock;` in behavior.
- Implement custom `VALIDATE_LOCK` for complex validations.
- Consider combining optimistic locking with draft handling for better user experience.

---