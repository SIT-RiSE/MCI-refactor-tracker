# How to Read the Refactoring Guideline

This document explains **where to find a refactoring guide, how to read each section, and how to apply the changes** to your test code. We illustrate everything with the example `cloudstack_MCI_1` / `cloudstack_Test_1_1`.

---

## 1  Locate the Guide

Guides are stored in two parallel folders:

```
HTML version Refactoring guidelines/
cloudstack-Refactoring-guide.html
```
or
```
Refactoring guidelines/
cloudstack-Refactoring-guide.md
```


Choose either format and open the file that matches your project (`cloudstack-Refactoring-guide.*` in this case).

---

## 2  Search by `MCI_id`

Search for **`cloudstack_MCI_1`** inside the guide.  
You will see a **Mock Clone Instance** block like this:


| Field | Meaning |
|-------|---------|
| **Scope** | `class level` → clone spans multiple test classes; create a helper **class** in the same package. <br>`method level` → clone local to one test class; insert a helper **method**. |
| **Mocked Class** | The class being mocked. |
| **Test Case Count** | Number of test cases that share this clone. |
| **MO Count** | Number of mock objects to refactor. |
| **Reusable Method / Class** | Code you must copy into the project. |



---

## 3  Search by `Test_id`

Search for **`cloudstack_Test_1_1`** in the same guide.  
The section shows three parts:

| Section              | Description                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| **Suggested Diff**   | The exact code changes to apply.                                           |
| **Original Test Code** | Full context before refactoring. *(Click to expand)*                     |
| **Reusable Method**  | Code to reuse in your project. *(Click to expand)*                         |

---

### Understanding the Diff

```diff
--- original
+++ refactored
@@
- AccountService accountService = Mockito.mock(AccountService.class);
+ AccountService accountService = createMockAccountService(account);
```

| Symbol         | Meaning                                     |
| -------------- | ------------------------------------------- |
| `-` (minus)    | **Delete** the line from the original code. |
| `+` (plus)     | **Add** this new line in its place.         |
| Unmarked lines | Remain unchanged; shown for context.        |

Apply every change exactly as shown.

---

## 4  Apply the Refactoring (Step-by-Step)

1. **Add** the reusable method or class to the project in the correct package.
2. **Modify** each affected test according to the diff.
3. **Compile** the project to ensure no errors.
4. **Run** unit tests; they should still pass (or be fixed).
5. **Run** mutation tests to compare scores before vs. after refactoring.

You are now ready to proceed with the upload and logging steps described in the main task guide.


