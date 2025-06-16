# 🔬 How to Run PIT Mutation Testing

This guide will walk you through **how to run PIT Mutation Testing** in **Eclipse** or **IntelliJ IDEA**, even if you’ve never done it before.
By the end, you’ll know how to run PIT, save the mutation reports, and organize them properly.

---

## ✅ Before You Start

### 🧠 What Is PIT Testing?

PIT Mutation Testing helps you **check how strong your unit tests are**. It creates small changes ("mutations") in your code and sees if your tests catch them.
If a test **fails**, that's good — it means the test detected the change!

---

## 🚀 Step 1: Setup

### 1.1 Choose Your IDE

Pick either:

* **Eclipse**
* **IntelliJ IDEA**

> ⚠️ For each Mock Clone Instance (MCI), you must use the **same IDE** for both *before* and *after* refactoring tests.

---

### 1.2 Install the PIT Plugin

| IDE          | How to Install                                                                                                                                                                                 |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Eclipse**  | Go to `Help → Eclipse Marketplace` → search **“PIT Mutation Testing”** → Install.<br>🔗 [Direct link](https://marketplace.eclipse.org/content/pitclipse)                                       |
| **IntelliJ** | Go to `File → Settings → Plugins → Marketplace`, search **“PIT Mutation Testing”**, and click **Install**.<br>🔗 [Plugin link](https://plugins.jetbrains.com/plugin/7119-pit-mutation-testing) |

---

## 🧪 Step 2: Running PIT

### Option A: Eclipse

1. **Right-click** the test class (or package).
2. Select `Run → PIT Mutation Test`.
3. Wait for PIT to run. Results will show in the **Console** tab.

---

### Option B: IntelliJ IDEA

#### 🔧 First-Time Setup

1. **Install the plugin** if not already done.
2. Go to `Run → Edit Configurations → + → PIT Runner`.

Fill in:

| Field              | What to Put                                                      |
| ------------------ | ---------------------------------------------------------------- |
| **Target Classes** | The package for your production code, e.g. `com.example.myapp.*` |
| **Target Tests**   | The test class, e.g. `com.example.myapp.MyServiceTest`           |

#### ⚙️ Optional Tweaks (Recommended)

* In "Before Launch", remove **Build** to save time.
* In **Other Params**, add: `--skipFailingTests true` (so it still runs even if tests fail)

#### ▶️ Run It

Click **Run**, and PIT will execute. Results appear in the **Run** window.

---

## 🧾 Step 3: Saving Mutation Reports

### 3.1 What to Copy

After PIT finishes, scroll in the **Console** (Eclipse) or **Run Window** (IntelliJ IDEA) to find output like this:

```
========================
- Mutators
========================
...
===================================
- Statistics
===================================
...
>> Mutations with no coverage 12232. Test strength 40%
>> Ran 5 tests (0 tests per mutation)
```

👉 **Copy everything from `- Mutators` down to `>> Ran xxx tests (0 tests per mutation)`**.
This is your mutation report.

---

### 3.2 How Many Reports to Generate?

Depends on your **MCI scope** in **Refactoring guidelines**:

#### 📌 Method-Level MCI

* All the refactory in one test class
* ✅ Only one **Test Class** need the report **before** and **after** refactoring.

#### 📌 Class-Level MCI

* Affects multiple test classes.
* ✅ Run PIT *before* and *after* refactoring for **each test class**, and save each report.

---

### 3.3 How to Name Your Files

Use this format:

```
MCI_{ID}_{TestClassName}.txt
```

Examples:

```
MCI_05_OrderTest.txt
MCI_12_BaseTest.txt
```

Save them under:

```
mutation-results/<YourName>/{before,after}/
```

---

## ✅ Final Checklist

| Step | What You Do                                                                               |
| ---- | ----------------------------------------------------------------------------------------- |
| ✅ 1  | Run PIT **before** refactoring. Save output to `before/`.                                 |
| ✅ 2  | Refactor the test class(es).                                                              |
| ✅ 3  | Run PIT **after** refactoring. Save output to `after/`.                                   |
| ✅ 4  | Upload your `.txt` files to the correct folder. Record results in the shared spreadsheet. |

---

## 💡 Tips & Troubleshooting

### 🔍 **No mutants found?**

> PIT reports something like:
> `INFO : Skipping coverage and analysis as no mutations found`

**What this means**: PIT didn't find any code to mutate. This often happens when the `Target Classes` setting is too narrow.

**What to try**:

* Make sure your production code package is correct. For example:

  * ✅ `com.myapp.service.*` → will detect classes
  * ❌ `com.myapp.service.SomeService` → too narrow, might be ignored
* Try using a **wider pattern**, like:

  * `com.myapp.*` or even just `com.*`

---

### 🧨 **Test crashes or exits immediately?**

> PIT stops and shows an error like:
> `All tests did not pass without mutation when calculating line coverage. Mutation testing requires a green suite.` or nothing runs at all.

**What this means**: Your test class might depend on a failing setup, or another class is throwing an exception.

**Fix**:

* In your **PIT Runner Configuration**, add this to **Other Params**:

  ```
  --skipFailingTests true
  ```
* This tells PIT to **skip broken tests** and still continue analyzing mutations.


