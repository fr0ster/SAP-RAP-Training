# 🗓️ Extended Summary: Enabling Draft Handling in SAP RAP (Markdown, 🇬🇧)

## Table of Contents (TOC)

- [Introduction to Draft Handling](#-introduction-to-draft-handling)
- [Purpose and Benefits of Draft Handling](#-purpose-and-benefits-of-draft-handling)
- [How Draft Handling Works](#-how-draft-handling-works)
- [Step-by-Step: Enabling Draft Handling](#-step-by-step-enabling-draft-handling)
- [Checklist: Validating Draft Functionality](#-checklist-validating-draft-functionality)
- [Etag and Total Etag Concepts](#-etag-and-total-etag-concepts)
- [Key Entities and Their Responsibilities](#-key-entities-and-their-responsibilities)
- [Draft-Enabled Behavior Definition](#-draft-enabled-behavior-definition)
- [Draft Administrative Data](#-draft-administrative-data)
- [Important Considerations](#-important-considerations)
- [Best Practices](#-best-practices)
- [References](#-references)

---

## ✨ Introduction to Draft Handling

Draft handling is a built-in mechanism of the **ABAP RESTful Application Programming Model (RAP)** that allows users to temporarily save their changes without committing them to the active version of the data immediately.

This mechanism enhances user experience by providing **autosave**, **multi-step editing**, and **error recovery** capabilities.

[Learn more](https://help.sap.com/docs/abap-cloud/abap-rap/draft-handling-in-rap)

## 📅 Purpose and Benefits of Draft Handling

- Allow users to work on incomplete data without affecting active records.
- Support autosave functionality.
- Enable error recovery: users can return later to complete unfinished tasks.
- Enable collaborative editing: different users can view the draft state.


## ⚖️ How Draft Handling Works

When draft handling is enabled for a RAP business object:

- A **draft version** of the entity instance is created and stored separately from the active instance.
- Upon "activation" (save/submit), the draft is merged into the active version.
- **Draft Administrative Data** tracks draft metadata like last modified timestamp and user.
- Locking mechanisms ensure that only one user edits a draft at a time.


## 🔢 Step-by-Step: Enabling Draft Handling

[Detailed guide](https://help.sap.com/docs/abap-cloud/abap-rap/enabling-draft-handling-in-rap)

### Step 1: Define Draft Table
- Create a database table for draft instances.
- Typically similar structure to active table plus technical fields.

### Step 2: Extend the CDS View
- Extend the CDS entity with draft-specific annotations.
```abap
@AccessControl.authorizationCheck: #NOT_REQUIRED
@ObjectModel:
  {
    draftEnabled: true,
    writeActivePersistence: 'ZSalesOrder',
    draftTable: 'ZSalesOrder_Draft'
  }
define root view entity ZI_SalesOrder ...
```

### Step 3: Update Behavior Definition
- Add `with draft` and specify `draft table`.
```abap
define behavior for ZI_SalesOrder
persistent table ZSalesOrder
lock master
with draft
{
  draft table ZSalesOrder_Draft;
  define behavior draft;
}
```

### Step 4: Link Draft Administrative Data
- Either reuse SAP's standard DraftAdministrativeData or define a new administrative CDS view.
- Map administrative fields (`DraftUUID`, `LastChangedBy`, etc.).

### Step 5: Implement Required Actions
- Implement at least basic system actions: `edit`, `discard`, `prepare`, `activate`.
- Extend logic if custom behavior needed during draft lifecycle.

### Step 6: Test the Behavior
- Test create, edit, discard, activate scenarios.
- Check autosave and locking behavior.


## 🔧 Checklist: Validating Draft Functionality

✅ **Creation**
- Draft is created when you start editing an active document.
- Draft UUID is correctly generated.

✅ **Modification**
- Changes are saved into the draft version without affecting the active version.
- Autosave periodically stores the draft.

✅ **Locking**
- When one user starts editing, another user attempting to edit sees a lock message.

✅ **Activation**
- Upon activation, draft changes are merged into the active version.
- Draft entry is deleted after successful activation.

✅ **Discard**
- User can discard a draft without affecting the active version.
- Draft record is deleted on discard.

✅ **Conflict Management (Etag / Total Etag)**
- Etag validation prevents overwriting another user's changes.
- Total etag changes if any dependent entity (composition) is modified.

✅ **Draft Administrative Data**
- Administrative data like `LastChangedBy` and `LastChangedAt` are updated correctly.
- LockedBy field shows correct editor during editing.

✅ **Fallback and Recovery**
- System allows recovery of unfinished drafts.
- System differentiates between user's own drafts and drafts locked by others.


## 🔍 Etag and Total Etag Concepts

[More about ETags](https://help.sap.com/docs/abap-cloud/abap-rap/handling-etags-in-rap)

### Etag
- The **etag** represents a fingerprint of a **single entity**'s current state.
- It changes whenever the entity instance is updated (either draft or active).
- Helps in **optimistic locking**: clients check if the Etag has changed before submitting changes.

### Total Etag
- The **total etag** represents a fingerprint of the **entire business object** including all compositions.
- Useful for complex objects where multiple sub-entities (items, schedules, etc.) are involved.
- If any sub-entity changes, the total etag changes.

**Difference**
| Aspect          | Etag                        | Total Etag                       |
|-----------------|------------------------------|-----------------------------------|
| Scope           | Single entity instance       | Entire business object           |
| Trigger for Change | Update of this instance only | Update of any part of the object |
| Usage           | Conflict detection on instance level | Conflict detection on full object |


## 📃 Key Entities and Their Responsibilities

| Entity                               | Responsibility                                |
|--------------------------------------|-----------------------------------------------|
| Active Persistence                   | Stores active versions of the data           |
| Draft Persistence                    | Stores draft versions separately             |
| Draft Administrative Data            | Metadata about draft: user, timestamp, status |
| Behavior Implementation (BO Handler) | Logic for editing, saving, and activating drafts |


## 🖋️ Draft-Enabled Behavior Definition

When defining behavior for a draft-enabled entity:
- **draft table** must be specified.
- Special actions such as `prepare`, `activate`, `discard`, `resume` are automatically provided.
- Custom logic can be added via determinations, validations, or additional actions.


## 🔧 Draft Administrative Data

[Detailed view](https://help.sap.com/docs/abap-cloud/abap-rap/draft-administrative-data-in-rap)

Drafts are associated with **DraftAdministrativeData**, containing:
- **Draft UUID**: Unique draft identifier
- **Last Changed By**: User ID
- **Last Changed At**: Timestamp
- **Draft Status**: (New, Saved, In Process)
- **Locked By**: User who is currently editing the draft

These fields allow safe collaboration and consistency checks.


## 🔍 Important Considerations

- Drafts can be autosaved periodically.
- If a draft is activated, the draft entry is deleted, and the active entry is updated.
- Locks prevent multiple users from editing the same draft.
- If an error occurs during editing, the draft persists, allowing retry.
- Deletion of a draft without activation is also possible (discard).


## 📚 Best Practices

- Use **draft-enabled entities** for business scenarios requiring long-running user interactions.
- Implement **optimistic locking** using etags to prevent overwriting changes.
- Clearly separate logic for draft and active instances in behavior implementation.
- Utilize **total etags** for complex BOs to ensure consistency across compositions.
- Implement validations both at the draft and activation phases.
- Regularly clean up obsolete or abandoned drafts to optimize storage.


## 🔗 References

- [SAP Help Portal — Draft Handling Overview](https://help.sap.com/docs/abap-cloud/abap-rap/draft-handling-in-rap)
- [SAP Help Portal — Enabling Draft Handling (Step-by-Step)](https://help.sap.com/docs/abap-cloud/abap-rap/enabling-draft-handling-in-rap)
- [SAP Help Portal — Draft Administrative Data](https://help.sap.com/docs/abap-cloud/abap-rap/draft-administrative-data-in-rap)
- [SAP Help Portal — Handling ETags and Total ETags](https://help.sap.com/docs/abap-cloud/abap-rap/handling-etags-in-rap)
- [Simplest SAP RAP Example - Draft Notes](https://github.com/fr0ster/simplest_sap_rap_example/blob/master/7th_iteration/notes.md)

---