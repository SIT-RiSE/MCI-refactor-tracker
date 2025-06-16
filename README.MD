# Mock Clone Refactoring Task Guide

This guide covers **where to find your tasks, what to upload, and what to log in the spreadsheet**.  


---

## 1  Get Your Task

All tasks are managed in the Google Sheet **`MCI Refactoring Assignment & Verification`**, where you can find your assigned tasks and log your progress:  
<https://docs.google.com/spreadsheets/d/1hkJn9Bx_ZC6QTH_zWtqiKJSOCH4_wKoQ>  

The refactoring guidelines and clone-related reports are available in the GitHub repository:  
<https://github.com/SIT-RiSE/MCI-refactor-tracker>  

### What You Need to Do:
1. **Find Your Task**: Locate your assigned tasks in the shared Google Sheet.  
2. **Follow the Refactoring Guide**: Use the provided refactoring solutions in the repository to complete your tasks.  
3. **Update Your Records**: Log your progress in the Google Sheet and upload mutation test results (TXT or CSV) to the GitHub repository using the specified folder structure.

---

Below is an explanation of the main components of the Google Sheet:


| Sheet | Purpose |
|-------|---------|
| **Progress Overview** | Total MCIs and test cases across all projects |
| **batch assignment & hunting** | Daily batch assignments; self-assign (`hunt`) by writing your name |
| **batch_test_mapping** | Maps each `batch_id` to its list of `MCI_id` + `Test_id` |
| **`<your name>`** | Your personal sheet—paste here all rows you will work on |

**Example**

Here’s an example table showing a batch assigned to Xinyi (just an example):

| Batch ID | Project     | MCI Count | Test Count | Assigned To |
|----------|-------------|-----------|------------|-------------|
| 17       | CloudStack  | 4         | 17         | Xinyi Li    |

After taking batch 17, Xinyi copies every matching row (total 17 tests) from **batch_test_mapping** into her personal sheet:

| batch_id | project | MCI_id | Test_id | Assigned_To |
|----------|---------|--------|---------|-------------|
| 17 | cloudstack | cloudstack_MCI_1 | cloudstack_Test_1_1 | Xinyi Li |
| 17 | cloudstack | cloudstack_MCI_1 | cloudstack_Test_1_2 | Xinyi Li |
| 17 | cloudstack | cloudstack_MCI_2 | cloudstack_Test_2_1 | Xinyi Li |
| … | … | … | … | … |

---

## 2  Work on Each Mock Clone Instance and Test Case

For **each `Test_id` row** in your personal sheet, complete the four steps below.

### Step 1  Baseline Mutation & Unit Tests  
Run mutation tests and unit tests **before**.  
Upload the mutation report to:

```
mutation-results/<Your Name>/before/<MCI_id>_<ClassName>.txt
```
Or, depending on your chosen output format, you can also upload a CSV document.
```
mutation-results/<Your Name>/before/<MCI_id>_<ClassName>.csv
```

### Step 2  Apply the Refactoring  
Follow the refactoring guide:

1. Insert the reusable method/class exactly as shown.  
2. Modify the test according to the suggested diff.

Then update your personal sheet:

| Column | Value |
|--------|-------|
| **compile_will** | `Y` if the code compiles without errors; otherwise `N` |
| **Test_pass** | `Y` if all unit tests pass; otherwise `N` |
| **Easy_to_Debug (if Failed)** | `Y` if failures can be fixed by minor tweaks (order, small syntax); else `N` |
| **Note** | Record any special findings or issues |

### Step 3  Post-Refactor Mutation & Unit Tests  
After the refactoring of each test case in the Mock Clone Instance is completed. Run the mutation tests and unit tests again.  
Upload the new mutation report to:

```
mutation-results/<Your Name>/after/<MCI_id>_<ClassName>.txt
```
Or, depending on your chosen output format, you can also upload a CSV document.
```
mutation-results/<Your Name>/after/<MCI_id>_<ClassName>.csv
```

### Step 4  Clean Up  
Revert all refactoring changes **in the test** (also need to remove added methods/classes and test edits).  

### Example Output  

The following example corresponds to Xinyi's `MCI ID = cloudstack_MCI_1` and its associated test class, `AddVpnUserCmdTest`:

| batch_id | project     | MCI_id              | Test_id               | Assigned_To | compile_will | Test_pass | Easy_to_Debug (if Failed) | Note |
|----------|-------------|---------------------|-----------------------|-------------|--------------|-----------|---------------------------|------|
| 17       | cloudstack  | cloudstack_MCI_1    | cloudstack_Test_1_1   | Xinyi Li    | Y            | Y         |                           |      |
| 17       | cloudstack  | cloudstack_MCI_1    | cloudstack_Test_1_2   | Xinyi Li    | Y            | Y         |                           |      |
|...|


Mutation reports uploaded:

- **Before Refactoring**:  
    `mutation-results/Xinyi Li/before/cloudstack_MCI_1_AddVpnUserCmdTest.txt`

- **After Refactoring**:  
    `mutation-results/Xinyi Li/after/cloudstack_MCI_1_AddVpnUserCmdTest.txt`



**Note**: If a mock clone instance is at the class level and has multiple mutation tests, you need to execute mutation tests for each test class **before** refactoring. After completing the refactoring, execute mutation tests for each test class again and upload every report. For example:

`cloudstack_MCI_3` is at the class level, so you need to upload mutation test reports for two test classes.

- **Before Refactoring**:  
    `mutation-results/Xinyi Li/before/cloudstack_MCI_3_LinkDomainToLdapCmdTest.txt`  
    `mutation-results/Xinyi Li/before/cloudstack_MCI_3_LinkAccountToLdapCmdTest.txt`

- **After Refactoring**:  
    `mutation-results/Xinyi Li/after/cloudstack_MCI_3_LinkDomainToLdapCmdTest.txt`  
    `mutation-results/Xinyi Li/after/cloudstack_MCI_3_LinkAccountToLdapCmdTest.txt`