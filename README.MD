# 🧩 Mock Clone Refactoring Task Guide

This guide explains how to:
- ✅ Claim your task in the shared spreadsheet  
- ✅ Apply the refactoring using the official guide  
- ✅ Run and upload mutation test reports  
- ✅ Record your results in the personal tab

By the end, you’ll know what to do for each test case and how to report your results properly.

---

## 1️⃣ Claim Your Task

### 🔍 Where to Find It

All assignments are managed in this shared spreadsheet:  
📄 [**MCI Refactoring Assignment & Verification**](https://docs.google.com/spreadsheets/d/1hkJn9Bx_ZC6QTH_zWtqiKJSOCH4_wKoQ):

<https://docs.google.com/spreadsheets/d/1hkJn9Bx_ZC6QTH_zWtqiKJSOCH4_wKoQ>

The refactoring guides and clone-related reports are available here:  
📁 [**GitHub Repository – MCI Refactor Tracker**](https://github.com/SIT-RiSE/MCI-refactor-tracker):

<https://github.com/SIT-RiSE/MCI-refactor-tracker>

---

### 📝 What to Do
In [**MCI Refactoring Assignment & Verification**](https://docs.google.com/spreadsheets/d/1hkJn9Bx_ZC6QTH_zWtqiKJSOCH4_wKoQ)
1. **Open the `batch assignment & hunting` sheet**  
   - Find a batch with an empty `Assigned To` column  
   - **Write your name** to claim it

2. **Go to the `batch_test_mapping` sheet**  
   - Use `Ctrl+F` to find your `batch_id`
   - Copy **all rows** with the same `batch_id`

3. **Paste the rows into your personal tab**  
   - Go to the sheet named after you
   - Paste all copied rows there — these are your tasks

---

## 2️⃣ Work on Each Mock Clone Instance (MCI)

You will complete your task **MCI by MCI**.  
Each MCI may affect **one or more test classes**.  
> 📌 For step-by-step help on refactoring, refer to:  
[**How to Read the Refactoring Guide**](https://github.com/SIT-RiSE/MCI-refactor-tracker/blob/main/How%20to%20Read%20the%20Refactoring%20Guide.md)

---

### 🧪 Step 1: Run Mutation Tests — *Before Refactoring*

For **each test class** affected by the MCI:

- Run **unit tests** and **mutation tests**
- Upload the PIT report to GitHub:

> 📌 For step-by-step help on mutation tests, refer to:  
[**How to Run Mutation Tests**](https://github.com/SIT-RiSE/MCI-refactor-tracker/blob/main/How%20to%20Run%20Mutation%20Tests.md)


```
mutation-results/<Your Name>/before/\<MCI_id>_<TestClassName>.txt
```

---

### 🛠 Step 2: Apply the Refactoring

Follow the guide:

- Insert the reusable method or class
- Apply the suggested diff to each test class

---

### 📥 Update Your Personal Sheet

In your personal sheet:

1. **Paste the first 5 columns** (`batch_id`, `project`, `MCI_id`, `Test_id`, `Assigned_To`) from the `batch_test_mapping` sheet  
   *(Do this when you first take your task.)*

2. As you finish each test case, **fill in the following result columns manually**:

| Column | What to Enter |
|--------|----------------|
| `compile_will` | `Y` if code compiles after refactoring, `N` otherwise |
| `Test_pass` | `Y` if unit tests pass |
| `Easy_to_Debug (if Failed)` | `Y` for small fixable issues, `N` for hard-to-debug failures |
| `Note` | Optional: notes about problems or special cases |
| `before` | Paste PIT stats **before refactoring** |
| `after` | Paste PIT stats **after refactoring** |

💡 The PIT summary should start from:

```

> > Line Coverage: ...
> > Generated ... mutations ...
> > Mutations with no coverage ...
> > Test strength ...

```

---

### 🧪 Step 3: Run Mutation Tests — *After Refactoring*

- Run mutation tests again for **each test class**
- Upload to:

```
mutation-results/<Your Name>/after/\<MCI_id>_<TestClassName>.txt
```

Paste the new stats into the `after` column in your sheet.

---

### 🧹 Step 4: Clean Up

Once you're done:

- Revert the refactored code in test files
- Delete helper methods or classes you added
- Ensure the codebase is clean and unchanged

---

## 🧾 Example Sheet Entry

If you're assigned `cloudstack_MCI_1` with two test classes, only the columns requiring manual input are shown below:

#### Test Case: `cloudstack_Test_1_1`
- **compile_will**: `Y`
- **Test_pass**: `Y`
- **Easy_to_Debug**: *(leave blank if not applicable)*
- **Note**: *(optional)*
- **before**:
    ```
    >> Line Coverage: 9/512...
    ```
- **after**:
    ```
    >> Line Coverage: 9/512...
    ```

---

## ⚠️ Special Case: Class-Level MCI

If the refactoring `Scope` is `class level`:

- You must run and upload reports for **each test class in the MCI**
- Upload `before/` and `after/` PIT results separately for each test class
- Log each test case row individually in your sheet

Example:

```
mutation-results/Xinyi Li/before/cloudstack_MCI_3_LinkDomainToLdapCmdTest.txt
mutation-results/Xinyi Li/before/cloudstack_MCI_3_LinkAccountToLdapCmdTest.txt
mutation-results/Xinyi Li/after/cloudstack_MCI_3_LinkDomainToLdapCmdTest.txt
mutation-results/Xinyi Li/after/cloudstack_MCI_3_LinkAccountToLdapCmdTest.txt
```

---

## ✅ Final Checklist

| Task | Complete? |
|------|-----------|
| Claimed batch in `batch assignment & hunting` | ✅ |
| Pasted tasks from `batch_test_mapping` to personal tab | ✅ |
| Followed refactoring guide | ✅ |
| Uploaded mutation test reports (before & after) | ✅ |
| Completed result columns in personal sheet | ✅ |
| Cleaned up refactoring changes in code | ✅ |

---

