# 📘 How to Use the Refactoring Guide (Step-by-Step for Students)

This guide explains how to use the **Mock Clone Refactoring Guide** for your assigned task.
You'll learn how to:

* Locate your mock clone instance
* Understand each section of the guide
* Apply the refactoring correctly based on its **scope**
* Avoid common mistakes

---

## 🔍 Step 1: Open the Refactoring Guide File

Refactoring guides are saved as either:

```
HTML version → Refactoring guidelines/<project>-Refactoring-guide.html  
Markdown version → Refactoring guidelines/<project>-Refactoring-guide.md
```

For example, if your project is `kiota-java`, open:

```
Refactoring guidelines/kiota-java-Refactoring-guide.md
```

Use any text editor or browser to view the file.

---

## 🧠 Step 2: Locate Your Assigned MCI

Every mock clone instance is labeled by its `MCI ID`, like:

```
kiota-java_MCI_1
```

Use `Ctrl+F` to search for your assigned `MCI ID` in the guide.

You will see a summary block like this:

```text
## Mock Clone Instance #kiota-java_MCI_1
- Scope: method level
- Mocked Class: com.example.MockedService
- Test Case Count: 3
- MO Count: 2
```

| Field               | Meaning                                                                                                                                                     |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Scope**           | Determines the type of refactoring:<br>🔹 `method level`: only one test class is affected.<br>🔹 `class level`: multiple test classes share the mock logic. |
| **Mocked Class**    | The class being mocked (e.g., `SerializationWriter`).                                                                                                       |
| **Test Case Count** | Number of test cases using this clone.                                                                                                                      |
| **MO Count**        | Number of mock objects being refactored.                                                                                                                    |

---

## 🧩 Step 3: Understand the Scope

### 🧪 `method level` (Local Refactoring)

* Only one test class is involved.
* You will:

  1. Insert a **reusable method** inside that class (or a shared `@BeforeEach` mock).
  2. Refactor each target test case inside that class.

✅ You do **not** need to create a new file.

---

### 🧱 `class level` (Cross-Class Refactoring)

* Multiple test classes contain the same mock logic.
* You must:

  1. Create a **new helper class** (e.g., `MockHelper`) in the **same package**.
  2. Move the reusable method(s) into this class.
  3. Let each test class **call that shared method**.

✅ Each affected test class will use this helper class to reduce duplication.

---

## 📄 Step 4: Read the Suggested Refactoring

Under the MCI, you'll see one or more **Test Case** entries like:

```
#### Test Case ID: kiota-java_Test_1_1  
Test Case Name: serializesObject  
File: path/to/TestClass.java  
```

Each test case has three parts:

| Section                | Purpose                                                                   |
| ---------------------- | ------------------------------------------------------------------------- |
| **Suggested Diff**     | Shows exactly what to change (`-` for deleted lines, `+` for added ones). |
| **Original Test Code** | The full code before refactoring (for your reference).                    |
| **Reusable Method**    | The shared logic that should be extracted or reused.                      |

---

### 📘 How to Read the Diff

```diff
- final var mockWriter = mock(Writer.class);
- when(mockWriter.getData()).thenReturn(...);
+ final var mockWriter = createMockWriter(...);
```

| Symbol      | Meaning                                 |
| ----------- | --------------------------------------- |
| `-`         | Delete this line                        |
| `+`         | Add this new line                       |
| (no symbol) | Leave the line unchanged (context only) |

Apply the changes **exactly as shown**. Make sure the variable names and indentation match.

---

## 🛠 Step 5: Apply the Refactoring

Follow this process:

### 🔧 For Method-Level MCI:

1. Add the **reusable method** into the **same test class**.
2. Refactor the target test case(s) by applying the **suggested diff**.
3. Make sure the test class compiles and passes all tests.

### 🧱 For Class-Level MCI:

1. Create a **new helper class** (e.g., `MockWriterHelper`) inside the **same package**.
2. Move the reusable method(s) into that new class.
3. Modify **each test class** to call the shared helper method.
4. Make sure all classes compile and pass unit tests.

---

## ✅ Step 6: Verify and Test

After applying the changes:

1. **Build** the project to check for compile errors.
2. **Run** all affected test cases.
3. **Run PIT mutation testing** and save:

   * The report **before** refactoring
   * The report **after** refactoring

📌 Make sure to save the reports using the correct naming and upload structure (see main guide).

