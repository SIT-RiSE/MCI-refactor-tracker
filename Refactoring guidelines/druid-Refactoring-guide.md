# A guide to Refactoring mock clones in druid

## Mock Clone Instance #druid_MCI_1
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.indexing.common.TaskToolbox`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
private static TaskToolbox createMockTaskToolbox(
    String attemptId,
    DruidNode taskExecutorNode,
    TaskLogPusher taskLogPusher,
    TaskConfig config,
    ObjectMapper objectMapper,
    TaskActionClient taskActionClient
) {
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn(attemptId);
    when(toolbox.getTaskExecutorNode()).thenReturn(taskExecutorNode);
    when(toolbox.getTaskLogPusher()).thenReturn(taskLogPusher);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    return toolbox;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_1_1
#### Test Case Name: `testSetupAndCleanupIsCalledWtihParameter`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\AbstractTaskTest.java`)
#### Mock Object Variable Name: `toolbox`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    // These tests apparently use Mockito.  Mockito is bad as we've seen it rewrite byte code and effectively cause
    // impact to other totally unrelated tests.  Mockito needs to be completely erradicated from the codebase.  This
    // comment is here to either cause me to do it in this commit or just for posterity so that it is clear that it
    // should happen in the future.
-    TaskToolbox toolbox = mock(TaskToolbox.class);
-    when(toolbox.getAttemptId()).thenReturn("1");
+    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
+    TaskLogPusher pusher = mock(TaskLogPusher.class);
+    TaskConfig config = mock(TaskConfig.class);
+    when(config.isEncapsulatedTask()).thenReturn(true);
+    File folder = temporaryFolder.newFolder();
+    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
+    TaskActionClient taskActionClient = mock(TaskActionClient.class);
+    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
+    TaskToolbox toolbox = createMockTaskToolbox(
+        "1",
+        node,
+        pusher,
+        config,
+        objectMapper,
+        taskActionClient
+    );
     DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
-    when(toolbox.getTaskExecutorNode()).thenReturn(node);
     TaskLogPusher pusher = mock(TaskLogPusher.class);
-    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
     TaskConfig config = mock(TaskConfig.class);
     when(config.isEncapsulatedTask()).thenReturn(true);
     File folder = temporaryFolder.newFolder();
     when(config.getTaskDir(eq("myID"))).thenReturn(folder);
-    when(toolbox.getConfig()).thenReturn(config);
-    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
     TaskActionClient taskActionClient = mock(TaskActionClient.class);
     when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
-    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
     AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

         @Nullable
         @Override
         public String setup(TaskToolbox toolbox) throws Exception {
             // create a reports file to test the taskLogPusher pushes task reports
             String result = super.setup(toolbox);
             File attemptDir = Paths.get(folder.getAbsolutePath(), "attempt", toolbox.getAttemptId()).toFile();
             File reportsDir = new File(attemptDir, "report.json");
             File statusDir = new File(attemptDir, "status.json");
             FileUtils.write(reportsDir, "foo", StandardCharsets.UTF_8);
             FileUtils.write(statusDir, "{}", StandardCharsets.UTF_8);
             return result;
         }
     };
     task.run(toolbox);
     // call it 3 times, once to update location in setup, then one for status and location in cleanup
     Mockito.verify(taskActionClient, times(3)).submit(any());
     verify(pusher, times(1)).pushTaskReports(eq("myID"), any());
     verify(pusher, times(1)).pushTaskStatus(eq("myID"), any());
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSetupAndCleanupIsCalledWtihParameter() throws Exception {
    // These tests apparently use Mockito.  Mockito is bad as we've seen it rewrite byte code and effectively cause
    // impact to other totally unrelated tests.  Mockito needs to be completely erradicated from the codebase.  This
    // comment is here to either cause me to do it in this commit or just for posterity so that it is clear that it
    // should happen in the future.
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn("1");
    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
    when(toolbox.getTaskExecutorNode()).thenReturn(node);
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(true);
    File folder = temporaryFolder.newFolder();
    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Nullable
        @Override
        public String setup(TaskToolbox toolbox) throws Exception {
            // create a reports file to test the taskLogPusher pushes task reports
            String result = super.setup(toolbox);
            File attemptDir = Paths.get(folder.getAbsolutePath(), "attempt", toolbox.getAttemptId()).toFile();
            File reportsDir = new File(attemptDir, "report.json");
            File statusDir = new File(attemptDir, "status.json");
            FileUtils.write(reportsDir, "foo", StandardCharsets.UTF_8);
            FileUtils.write(statusDir, "{}", StandardCharsets.UTF_8);
            return result;
        }
    };
    task.run(toolbox);
    // call it 3 times, once to update location in setup, then one for status and location in cleanup
    Mockito.verify(taskActionClient, times(3)).submit(any());
    verify(pusher, times(1)).pushTaskReports(eq("myID"), any());
    verify(pusher, times(1)).pushTaskStatus(eq("myID"), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static TaskToolbox createMockTaskToolbox(
    String attemptId,
    DruidNode taskExecutorNode,
    TaskLogPusher taskLogPusher,
    TaskConfig config,
    ObjectMapper objectMapper,
    TaskActionClient taskActionClient
) {
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn(attemptId);
    when(toolbox.getTaskExecutorNode()).thenReturn(taskExecutorNode);
    when(toolbox.getTaskLogPusher()).thenReturn(taskLogPusher);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    return toolbox;
}
```
</details>

---
#### Test Case ID #druid_Test_1_2
#### Test Case Name: `testWithNoEncapsulatedTask`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\AbstractTaskTest.java`)
#### Mock Object Variable Name: `toolbox`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testWithNoEncapsulatedTask() throws Exception {
-    TaskToolbox toolbox = mock(TaskToolbox.class);
-    when(toolbox.getAttemptId()).thenReturn("1");
     DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
     when(toolbox.getTaskExecutorNode()).thenReturn(node);
     TaskLogPusher pusher = mock(TaskLogPusher.class);
     when(toolbox.getTaskLogPusher()).thenReturn(pusher);
     TaskConfig config = mock(TaskConfig.class);
     when(config.isEncapsulatedTask()).thenReturn(false);
     File folder = temporaryFolder.newFolder();
     when(config.getTaskDir(eq("myID"))).thenReturn(folder);
+    TaskToolbox toolbox = createMockTaskToolbox(
+        "1",
+        node,
+        pusher,
+        config,
+        objectMapper,
+        taskActionClient
+    );
     when(toolbox.getConfig()).thenReturn(config);
     when(toolbox.getJsonMapper()).thenReturn(objectMapper);
     TaskActionClient taskActionClient = mock(TaskActionClient.class);
     when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
     when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
     AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

         @Nullable
         @Override
         public String setup(TaskToolbox toolbox) throws Exception {
             // create a reports file to test the taskLogPusher pushes task reports
             String result = super.setup(toolbox);
             File attemptDir = Paths.get(folder.getAbsolutePath(), "attempt", toolbox.getAttemptId()).toFile();
             File reportsDir = new File(attemptDir, "report.json");
             FileUtils.write(reportsDir, "foo", StandardCharsets.UTF_8);
             return result;
         }
     };
     task.run(toolbox);
     // encapsulated task is set to false, should never get called
     Mockito.verify(taskActionClient, never()).submit(any());
     verify(pusher, never()).pushTaskReports(eq("myID"), any());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testWithNoEncapsulatedTask() throws Exception {
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn("1");
    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
    when(toolbox.getTaskExecutorNode()).thenReturn(node);
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(false);
    File folder = temporaryFolder.newFolder();
    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Nullable
        @Override
        public String setup(TaskToolbox toolbox) throws Exception {
            // create a reports file to test the taskLogPusher pushes task reports
            String result = super.setup(toolbox);
            File attemptDir = Paths.get(folder.getAbsolutePath(), "attempt", toolbox.getAttemptId()).toFile();
            File reportsDir = new File(attemptDir, "report.json");
            FileUtils.write(reportsDir, "foo", StandardCharsets.UTF_8);
            return result;
        }
    };
    task.run(toolbox);
    // encapsulated task is set to false, should never get called
    Mockito.verify(taskActionClient, never()).submit(any());
    verify(pusher, never()).pushTaskReports(eq("myID"), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static TaskToolbox createMockTaskToolbox(
    String attemptId,
    DruidNode taskExecutorNode,
    TaskLogPusher taskLogPusher,
    TaskConfig config,
    ObjectMapper objectMapper,
    TaskActionClient taskActionClient
) {
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn(attemptId);
    when(toolbox.getTaskExecutorNode()).thenReturn(taskExecutorNode);
    when(toolbox.getTaskLogPusher()).thenReturn(taskLogPusher);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    return toolbox;
}
```
</details>

---
#### Test Case ID #druid_Test_1_3
#### Test Case Name: `testTaskFailureWithoutExceptionGetsReportedCorrectly`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\AbstractTaskTest.java`)
#### Mock Object Variable Name: `toolbox`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
-    TaskToolbox toolbox = mock(TaskToolbox.class);
-    when(toolbox.getAttemptId()).thenReturn("1");
     DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
     TaskLogPusher pusher = mock(TaskLogPusher.class);
     when(toolbox.getTaskLogPusher()).thenReturn(pusher);
     TaskConfig config = mock(TaskConfig.class);
     when(config.isEncapsulatedTask()).thenReturn(true);
     File folder = temporaryFolder.newFolder();
     when(config.getTaskDir(eq("myID"))).thenReturn(folder);
+    TaskToolbox toolbox = createMockTaskToolbox(
+        "1",
+        node,
+        pusher,
+        config,
+        objectMapper,
+        taskActionClient
+    );
     when(toolbox.getConfig()).thenReturn(config);
     when(toolbox.getJsonMapper()).thenReturn(objectMapper);
     TaskActionClient taskActionClient = mock(TaskActionClient.class);
     when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
     when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
     AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

         @Override
         public TaskStatus runTask(TaskToolbox toolbox) {
             return TaskStatus.failure("myId", "failed");
         }
     };
     task.run(toolbox);
     UpdateStatusAction action = new UpdateStatusAction("failure");
     verify(taskActionClient).submit(eq(action));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testTaskFailureWithoutExceptionGetsReportedCorrectly() throws Exception {
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn("1");
    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
    when(toolbox.getTaskExecutorNode()).thenReturn(node);
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(true);
    File folder = temporaryFolder.newFolder();
    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Override
        public TaskStatus runTask(TaskToolbox toolbox) {
            return TaskStatus.failure("myId", "failed");
        }
    };
    task.run(toolbox);
    UpdateStatusAction action = new UpdateStatusAction("failure");
    verify(taskActionClient).submit(eq(action));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static TaskToolbox createMockTaskToolbox(
    String attemptId,
    DruidNode taskExecutorNode,
    TaskLogPusher taskLogPusher,
    TaskConfig config,
    ObjectMapper objectMapper,
    TaskActionClient taskActionClient
) {
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn(attemptId);
    when(toolbox.getTaskExecutorNode()).thenReturn(taskExecutorNode);
    when(toolbox.getTaskLogPusher()).thenReturn(taskLogPusher);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    return toolbox;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_2
- **Scope**: class level
- **Mocked Class**: `org.apache.druid.query.filter.ColumnIndexSelector`
- **Test Case Count**: 4
- **MO Count**: 4

### Reusable Method
```java
public class MockColumnIndexSelector {
  public static ColumnIndexSelector createMockColumnIndexSelector(String columnName, ColumnIndexSupplier indexSupplier) {
    ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
    Mockito.when(indexSelector.getIndexSupplier(columnName)).thenReturn(indexSupplier);
    return indexSelector;
  }
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_2_1
#### Test Case Name: `testUsesUtf8SetIndex`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\filter\InDimFilterTest.java`)
#### Mock Object Variable Name: `indexSelector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final Filter inFilter = new InDimFilter("dim0", ImmutableSet.of("v1", "v2")).toFilter();
-    final ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
+    final ColumnIndexSelector indexSelector = MockColumnIndexSelector.createMockColumnIndexSelector("dim0", indexSupplier);
    final ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    final Utf8ValueSetIndexes valueIndexes = Mockito.mock(Utf8ValueSetIndexes.class);
    final BitmapColumnIndex bitmapColumnIndex = Mockito.mock(BitmapColumnIndex.class);
    final InDimFilter.ValuesSet expectedValuesSet = new InDimFilter.ValuesSet();
    expectedValuesSet.addAll(Arrays.asList("v1", "v2"));
-    Mockito.when(indexSelector.getIndexSupplier("dim0")).thenReturn(indexSupplier);
    Mockito.when(indexSupplier.as(Utf8ValueSetIndexes.class)).thenReturn(valueIndexes);
    Mockito.when(valueIndexes.forSortedValuesUtf8(expectedValuesSet.toUtf8())).thenReturn(bitmapColumnIndex);
    final BitmapColumnIndex retVal = inFilter.getBitmapColumnIndex(indexSelector);
    Assert.assertSame("inFilter returns the intended bitmapColumnIndex", bitmapColumnIndex, retVal);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testUsesUtf8SetIndex() {
    // An implementation test.
    // This test confirms that "in" filters use utf8 index lookups when available.
    final Filter inFilter = new InDimFilter("dim0", ImmutableSet.of("v1", "v2")).toFilter();
    final ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
    final ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    final Utf8ValueSetIndexes valueIndexes = Mockito.mock(Utf8ValueSetIndexes.class);
    final BitmapColumnIndex bitmapColumnIndex = Mockito.mock(BitmapColumnIndex.class);
    final InDimFilter.ValuesSet expectedValuesSet = new InDimFilter.ValuesSet();
    expectedValuesSet.addAll(Arrays.asList("v1", "v2"));
    Mockito.when(indexSelector.getIndexSupplier("dim0")).thenReturn(indexSupplier);
    Mockito.when(indexSupplier.as(Utf8ValueSetIndexes.class)).thenReturn(valueIndexes);
    Mockito.when(valueIndexes.forSortedValuesUtf8(expectedValuesSet.toUtf8())).thenReturn(bitmapColumnIndex);
    final BitmapColumnIndex retVal = inFilter.getBitmapColumnIndex(indexSelector);
    Assert.assertSame("inFilter returns the intended bitmapColumnIndex", bitmapColumnIndex, retVal);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnIndexSelector {
  public static ColumnIndexSelector createMockColumnIndexSelector(String columnName, ColumnIndexSupplier indexSupplier) {
    ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
    Mockito.when(indexSelector.getIndexSupplier(columnName)).thenReturn(indexSupplier);
    return indexSelector;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_2_2
#### Test Case Name: `testUsesStringSetIndex`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\filter\InDimFilterTest.java`)
#### Mock Object Variable Name: `indexSelector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final Filter inFilter = new InDimFilter("dim0", ImmutableSet.of("v1", "v2")).toFilter();
-    final ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
+    final ColumnIndexSelector indexSelector = MockColumnIndexSelector.createMockColumnIndexSelector("dim0", indexSupplier);
    final ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    final StringValueSetIndexes valueIndex = Mockito.mock(StringValueSetIndexes.class);
    final BitmapColumnIndex bitmapColumnIndex = Mockito.mock(BitmapColumnIndex.class);
    final InDimFilter.ValuesSet expectedValuesSet = new InDimFilter.ValuesSet();
    expectedValuesSet.addAll(Arrays.asList("v1", "v2"));
-    Mockito.when(indexSelector.getIndexSupplier("dim0")).thenReturn(indexSupplier);
    // Will check for UTF-8 first.
    Mockito.when(indexSupplier.as(Utf8ValueSetIndexes.class)).thenReturn(null);
    Mockito.when(indexSupplier.as(StringValueSetIndexes.class)).thenReturn(valueIndex);
    Mockito.when(valueIndex.forSortedValues(expectedValuesSet)).thenReturn(bitmapColumnIndex);
    final BitmapColumnIndex retVal = inFilter.getBitmapColumnIndex(indexSelector);
    Assert.assertSame("inFilter returns the intended bitmapColumnIndex", bitmapColumnIndex, retVal);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testUsesStringSetIndex() {
    // An implementation test.
    // This test confirms that "in" filters use non-utf8 string index lookups when utf8 indexes are not available.
    final Filter inFilter = new InDimFilter("dim0", ImmutableSet.of("v1", "v2")).toFilter();
    final ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
    final ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    final StringValueSetIndexes valueIndex = Mockito.mock(StringValueSetIndexes.class);
    final BitmapColumnIndex bitmapColumnIndex = Mockito.mock(BitmapColumnIndex.class);
    final InDimFilter.ValuesSet expectedValuesSet = new InDimFilter.ValuesSet();
    expectedValuesSet.addAll(Arrays.asList("v1", "v2"));
    Mockito.when(indexSelector.getIndexSupplier("dim0")).thenReturn(indexSupplier);
    // Will check for UTF-8 first.
    Mockito.when(indexSupplier.as(Utf8ValueSetIndexes.class)).thenReturn(null);
    Mockito.when(indexSupplier.as(StringValueSetIndexes.class)).thenReturn(valueIndex);
    Mockito.when(valueIndex.forSortedValues(expectedValuesSet)).thenReturn(bitmapColumnIndex);
    final BitmapColumnIndex retVal = inFilter.getBitmapColumnIndex(indexSelector);
    Assert.assertSame("inFilter returns the intended bitmapColumnIndex", bitmapColumnIndex, retVal);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnIndexSelector {
  public static ColumnIndexSelector createMockColumnIndexSelector(String columnName, ColumnIndexSupplier indexSupplier) {
    ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
    Mockito.when(indexSelector.getIndexSupplier(columnName)).thenReturn(indexSupplier);
    return indexSelector;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_2_3
#### Test Case Name: `testPrefixMatchUsesRangeIndex`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\filter\LikeDimFilterTest.java`)
#### Mock Object Variable Name: `indexSelector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final Filter likeFilter = new LikeDimFilter("dim0", "f%", null, null, null).toFilter();
-    final ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
+    final ColumnIndexSelector indexSelector = MockColumnIndexSelector.createMockColumnIndexSelector("dim0", indexSupplier);
    final ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    final LexicographicalRangeIndexes rangeIndex = Mockito.mock(LexicographicalRangeIndexes.class);
    final BitmapColumnIndex bitmapColumnIndex = Mockito.mock(BitmapColumnIndex.class);
-    Mockito.when(indexSelector.getIndexSupplier("dim0")).thenReturn(indexSupplier);
    Mockito.when(indexSupplier.as(LexicographicalRangeIndexes.class)).thenReturn(rangeIndex);
    Mockito.when(// Verify that likeFilter uses forRange without a matcher predicate; it's unnecessary and slows things down
    rangeIndex.forRange("f", false, "f" + Character.MAX_VALUE, false)).thenReturn(bitmapColumnIndex);
    final BitmapColumnIndex retVal = likeFilter.getBitmapColumnIndex(indexSelector);
    Assert.assertSame("likeFilter returns the intended bitmapColumnIndex", bitmapColumnIndex, retVal);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testPrefixMatchUsesRangeIndex() {
    // An implementation test.
    // This test confirms that "like" filters with prefix matchers use index-range lookups without matcher predicates.
    final Filter likeFilter = new LikeDimFilter("dim0", "f%", null, null, null).toFilter();
    final ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
    final ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    final LexicographicalRangeIndexes rangeIndex = Mockito.mock(LexicographicalRangeIndexes.class);
    final BitmapColumnIndex bitmapColumnIndex = Mockito.mock(BitmapColumnIndex.class);
    Mockito.when(indexSelector.getIndexSupplier("dim0")).thenReturn(indexSupplier);
    Mockito.when(indexSupplier.as(LexicographicalRangeIndexes.class)).thenReturn(rangeIndex);
    Mockito.when(// Verify that likeFilter uses forRange without a matcher predicate; it's unnecessary and slows things down
    rangeIndex.forRange("f", false, "f" + Character.MAX_VALUE, false)).thenReturn(bitmapColumnIndex);
    final BitmapColumnIndex retVal = likeFilter.getBitmapColumnIndex(indexSelector);
    Assert.assertSame("likeFilter returns the intended bitmapColumnIndex", bitmapColumnIndex, retVal);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnIndexSelector {
  public static ColumnIndexSelector createMockColumnIndexSelector(String columnName, ColumnIndexSupplier indexSupplier) {
    ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
    Mockito.when(indexSelector.getIndexSupplier(columnName)).thenReturn(indexSupplier);
    return indexSelector;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_2_4
#### Test Case Name: `testExactMatchUsesValueIndex`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\filter\LikeDimFilterTest.java`)
#### Mock Object Variable Name: `indexSelector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final Filter likeFilter = new LikeDimFilter("dim0", "f", null, null, null).toFilter();
-    final ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
+    final ColumnIndexSelector indexSelector = MockColumnIndexSelector.createMockColumnIndexSelector("dim0", indexSupplier);
    final ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    final StringValueSetIndexes valueIndex = Mockito.mock(StringValueSetIndexes.class);
    final BitmapColumnIndex bitmapColumnIndex = Mockito.mock(BitmapColumnIndex.class);
-    Mockito.when(indexSelector.getIndexSupplier("dim0")).thenReturn(indexSupplier);
    Mockito.when(indexSupplier.as(StringValueSetIndexes.class)).thenReturn(valueIndex);
    Mockito.when(valueIndex.forValue("f")).thenReturn(bitmapColumnIndex);
    final BitmapColumnIndex retVal = likeFilter.getBitmapColumnIndex(indexSelector);
    Assert.assertSame("likeFilter returns the intended bitmapColumnIndex", bitmapColumnIndex, retVal);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testExactMatchUsesValueIndex() {
    // An implementation test.
    // This test confirms that "like" filters with exact matchers use index lookups.
    final Filter likeFilter = new LikeDimFilter("dim0", "f", null, null, null).toFilter();
    final ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
    final ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    final StringValueSetIndexes valueIndex = Mockito.mock(StringValueSetIndexes.class);
    final BitmapColumnIndex bitmapColumnIndex = Mockito.mock(BitmapColumnIndex.class);
    Mockito.when(indexSelector.getIndexSupplier("dim0")).thenReturn(indexSupplier);
    Mockito.when(indexSupplier.as(StringValueSetIndexes.class)).thenReturn(valueIndex);
    Mockito.when(valueIndex.forValue("f")).thenReturn(bitmapColumnIndex);
    final BitmapColumnIndex retVal = likeFilter.getBitmapColumnIndex(indexSelector);
    Assert.assertSame("likeFilter returns the intended bitmapColumnIndex", bitmapColumnIndex, retVal);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnIndexSelector {
  public static ColumnIndexSelector createMockColumnIndexSelector(String columnName, ColumnIndexSupplier indexSupplier) {
    ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
    Mockito.when(indexSelector.getIndexSupplier(columnName)).thenReturn(indexSupplier);
    return indexSelector;
  }
}
```
</details>

---
## Mock Clone Instance #druid_MCI_3
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.indexing.common.config.TaskConfig`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
private static TaskConfig createMockTaskConfig(boolean isEncapsulatedTaskReturn, File getTaskDirReturn) {
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(isEncapsulatedTaskReturn);
    when(config.getTaskDir(eq("myID"))).thenReturn(getTaskDirReturn);
    return config;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_3_1
#### Test Case Name: `testSetupAndCleanupIsCalledWtihParameter`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\AbstractTaskTest.java`)
#### Mock Object Variable Name: `config`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
-    TaskConfig config = mock(TaskConfig.class);
-    when(config.isEncapsulatedTask()).thenReturn(true);
    File folder = temporaryFolder.newFolder();
-    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
+    TaskConfig config = createMockTaskConfig(true, folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSetupAndCleanupIsCalledWtihParameter() throws Exception {
    // These tests apparently use Mockito.  Mockito is bad as we've seen it rewrite byte code and effectively cause
    // impact to other totally unrelated tests.  Mockito needs to be completely erradicated from the codebase.  This
    // comment is here to either cause me to do it in this commit or just for posterity so that it is clear that it
    // should happen in the future.
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn("1");
    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
    when(toolbox.getTaskExecutorNode()).thenReturn(node);
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(true);
    File folder = temporaryFolder.newFolder();
    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Nullable
        @Override
        public String setup(TaskToolbox toolbox) throws Exception {
            // create a reports file to test the taskLogPusher pushes task reports
            String result = super.setup(toolbox);
            File attemptDir = Paths.get(folder.getAbsolutePath(), "attempt", toolbox.getAttemptId()).toFile();
            File reportsDir = new File(attemptDir, "report.json");
            File statusDir = new File(attemptDir, "status.json");
            FileUtils.write(reportsDir, "foo", StandardCharsets.UTF_8);
            FileUtils.write(statusDir, "{}", StandardCharsets.UTF_8);
            return result;
        }
    };
    task.run(toolbox);
    // call it 3 times, once to update location in setup, then one for status and location in cleanup
    Mockito.verify(taskActionClient, times(3)).submit(any());
    verify(pusher, times(1)).pushTaskReports(eq("myID"), any());
    verify(pusher, times(1)).pushTaskStatus(eq("myID"), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static TaskConfig createMockTaskConfig(boolean isEncapsulatedTaskReturn, File getTaskDirReturn) {
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(isEncapsulatedTaskReturn);
    when(config.getTaskDir(eq("myID"))).thenReturn(getTaskDirReturn);
    return config;
}
```
</details>

---
#### Test Case ID #druid_Test_3_2
#### Test Case Name: `testWithNoEncapsulatedTask`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\AbstractTaskTest.java`)
#### Mock Object Variable Name: `config`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
-    TaskConfig config = mock(TaskConfig.class);
-    when(config.isEncapsulatedTask()).thenReturn(false);
    File folder = temporaryFolder.newFolder();
-    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
+    TaskConfig config = createMockTaskConfig(false, folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testWithNoEncapsulatedTask() throws Exception {
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn("1");
    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
    when(toolbox.getTaskExecutorNode()).thenReturn(node);
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(false);
    File folder = temporaryFolder.newFolder();
    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Nullable
        @Override
        public String setup(TaskToolbox toolbox) throws Exception {
            // create a reports file to test the taskLogPusher pushes task reports
            String result = super.setup(toolbox);
            File attemptDir = Paths.get(folder.getAbsolutePath(), "attempt", toolbox.getAttemptId()).toFile();
            File reportsDir = new File(attemptDir, "report.json");
            FileUtils.write(reportsDir, "foo", StandardCharsets.UTF_8);
            return result;
        }
    };
    task.run(toolbox);
    // encapsulated task is set to false, should never get called
    Mockito.verify(taskActionClient, never()).submit(any());
    verify(pusher, never()).pushTaskReports(eq("myID"), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static TaskConfig createMockTaskConfig(boolean isEncapsulatedTaskReturn, File getTaskDirReturn) {
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(isEncapsulatedTaskReturn);
    when(config.getTaskDir(eq("myID"))).thenReturn(getTaskDirReturn);
    return config;
}
```
</details>

---
#### Test Case ID #druid_Test_3_3
#### Test Case Name: `testTaskFailureWithoutExceptionGetsReportedCorrectly`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\AbstractTaskTest.java`)
#### Mock Object Variable Name: `config`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
-    TaskConfig config = mock(TaskConfig.class);
-    when(config.isEncapsulatedTask()).thenReturn(true);
    File folder = temporaryFolder.newFolder();
-    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
+    TaskConfig config = createMockTaskConfig(true, folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testTaskFailureWithoutExceptionGetsReportedCorrectly() throws Exception {
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn("1");
    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
    when(toolbox.getTaskExecutorNode()).thenReturn(node);
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(true);
    File folder = temporaryFolder.newFolder();
    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Override
        public TaskStatus runTask(TaskToolbox toolbox) {
            return TaskStatus.failure("myId", "failed");
        }
    };
    task.run(toolbox);
    UpdateStatusAction action = new UpdateStatusAction("failure");
    verify(taskActionClient).submit(eq(action));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static TaskConfig createMockTaskConfig(boolean isEncapsulatedTaskReturn, File getTaskDirReturn) {
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(isEncapsulatedTaskReturn);
    when(config.getTaskDir(eq("myID"))).thenReturn(getTaskDirReturn);
    return config;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_4
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.segment.loading.DataSegmentKiller`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
// === Declare in class scope ===
private DataSegmentKiller killer;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    killer = Mockito.mock(DataSegmentKiller.class);
}

// === Replace local variable in test with ===
killer

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_4_1
#### Test Case Name: `testKillSegmentWithType`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentKillerTest.java`)
#### Mock Object Variable Name: `killer`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testKillSegmentWithType() throws SegmentLoadingException {
-    final DataSegmentKiller killer = Mockito.mock(DataSegmentKiller.class);
+    // removed local mock; replaced with global field `killer`
     final DataSegment segment = Mockito.mock(DataSegment.class);
     Mockito.when(segment.isTombstone()).thenReturn(false);
     Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
     final Injector injector = createInjector(killer);
     final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
     segmentKiller.kill(segment);
-    Mockito.verify(killer, Mockito.times(1)).kill(segment);
+    Mockito.verify(killer, Mockito.times(1)).kill(segment);
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testKillSegmentWithType() throws SegmentLoadingException {
    final DataSegmentKiller killer = Mockito.mock(DataSegmentKiller.class);
    final DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.isTombstone()).thenReturn(false);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    final Injector injector = createInjector(killer);
    final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
    segmentKiller.kill(segment);
    Mockito.verify(killer, Mockito.times(1)).kill(segment);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private DataSegmentKiller killer;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    killer = Mockito.mock(DataSegmentKiller.class);
}

// === Replace local variable in test with ===
killer

```
</details>

---
#### Test Case ID #druid_Test_4_2
#### Test Case Name: `testKillMultipleSegmentsWithType`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentKillerTest.java`)
#### Mock Object Variable Name: `killerSane`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testKillMultipleSegmentsWithType() throws SegmentLoadingException {
-    final DataSegmentKiller killerSane = Mockito.mock(DataSegmentKiller.class);
+    // removed local mock; replaced with global field `killer`
     final DataSegmentKiller killerSaneTwo = Mockito.mock(DataSegmentKiller.class);
     final DataSegment segment1 = Mockito.mock(DataSegment.class);
     final DataSegment segment2 = Mockito.mock(DataSegment.class);
     final DataSegment segment3 = Mockito.mock(DataSegment.class);
     Mockito.when(segment1.isTombstone()).thenReturn(false);
     Mockito.when(segment1.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
     Mockito.when(segment2.isTombstone()).thenReturn(false);
     Mockito.when(segment2.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
     Mockito.when(segment3.isTombstone()).thenReturn(false);
     Mockito.when(segment3.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane_2"));
-    final Injector injector = createInjectorFromMap(ImmutableMap.of("sane", killerSane, "sane_2", killerSaneTwo));
+    final Injector injector = createInjectorFromMap(ImmutableMap.of("sane", killer, "sane_2", killerSaneTwo));
     final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
     segmentKiller.kill(ImmutableList.of(segment1, segment2, segment3));
-    Mockito.verify(killerSane, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment1, segment2)));
+    Mockito.verify(killer, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment1, segment2)));
     Mockito.verify(killerSaneTwo, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment3)));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testKillMultipleSegmentsWithType() throws SegmentLoadingException {
    final DataSegmentKiller killerSane = Mockito.mock(DataSegmentKiller.class);
    final DataSegmentKiller killerSaneTwo = Mockito.mock(DataSegmentKiller.class);
    final DataSegment segment1 = Mockito.mock(DataSegment.class);
    final DataSegment segment2 = Mockito.mock(DataSegment.class);
    final DataSegment segment3 = Mockito.mock(DataSegment.class);
    Mockito.when(segment1.isTombstone()).thenReturn(false);
    Mockito.when(segment1.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    Mockito.when(segment2.isTombstone()).thenReturn(false);
    Mockito.when(segment2.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    Mockito.when(segment3.isTombstone()).thenReturn(false);
    Mockito.when(segment3.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane_2"));
    final Injector injector = createInjectorFromMap(ImmutableMap.of("sane", killerSane, "sane_2", killerSaneTwo));
    final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
    segmentKiller.kill(ImmutableList.of(segment1, segment2, segment3));
    Mockito.verify(killerSane, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment1, segment2)));
    Mockito.verify(killerSaneTwo, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment3)));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private DataSegmentKiller killer;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    killer = Mockito.mock(DataSegmentKiller.class);
}

// === Replace local variable in test with ===
killer

```
</details>

---
## Mock Clone Instance #druid_MCI_5
- **Scope**: class level
- **Mocked Class**: `org.apache.druid.segment.ColumnValueSelector`
- **Test Case Count**: 1
- **MO Count**: 3

### Reusable Method
```java
public class MockColumnValueSelector {
  public static ColumnValueSelector createMockColumnValueSelector(Object[] getObjectReturn) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(getObjectReturn);
    return columnValueSelector;
  }
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_5_1
#### Test Case Name: `testAddingInDictionaryWithObjects`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayDoubleGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `columnValueSelector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testAddingInDictionaryWithObjects() {
-    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
-    Mockito.when(columnValueSelector.getObject()).thenReturn(new Object[] { 4.0D, 2.0D });
+    ColumnValueSelector columnValueSelector = MockColumnValueSelector.createMockColumnValueSelector(new Object[] { 4.0D, 2.0D });
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4.0, 2.0)), row.get(0));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testAddingInDictionaryWithObjects() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(new Object[] { 4.0D, 2.0D });
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4.0, 2.0)), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnValueSelector {
  public static ColumnValueSelector createMockColumnValueSelector(Object[] getObjectReturn) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(getObjectReturn);
    return columnValueSelector;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_5_2
#### Test Case Name: `testAddingInDictionaryWithObjects`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayLongGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `columnValueSelector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testAddingInDictionaryWithObjects() {
-    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
-    Mockito.when(columnValueSelector.getObject()).thenReturn(new Object[] { 4L, 2L });
+    ColumnValueSelector columnValueSelector = MockColumnValueSelector.createMockColumnValueSelector(new Object[] { 4L, 2L });
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4L, 2L)), row.get(0));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testAddingInDictionaryWithObjects() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(new Object[] { 4L, 2L });
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4L, 2L)), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnValueSelector {
  public static ColumnValueSelector createMockColumnValueSelector(Object[] getObjectReturn) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(getObjectReturn);
    return columnValueSelector;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_5_3
#### Test Case Name: `testAddingInDictionaryWithObjects`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayStringGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `columnValueSelector`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testAddingInDictionaryWithObjects() {
-    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
-    Mockito.when(columnValueSelector.getObject()).thenReturn(new Object[] { "f", "a" });
+    ColumnValueSelector columnValueSelector = MockColumnValueSelector.createMockColumnValueSelector(new Object[] { "f", "a" });
     Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
     GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
     Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
     ResultRow row = ResultRow.create(1);
     buffer1.putInt(3);
     strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
     Assert.assertEquals(ComparableStringArray.of("f", "a"), row.get(0));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testAddingInDictionaryWithObjects() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(new Object[] { "f", "a" });
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(ComparableStringArray.of("f", "a"), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnValueSelector {
  public static ColumnValueSelector createMockColumnValueSelector(Object[] getObjectReturn) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(getObjectReturn);
    return columnValueSelector;
  }
}
```
</details>

---
## Mock Clone Instance #druid_MCI_6
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.segment.ColumnValueSelector`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
private static ColumnValueSelector createMockColumnValueSelector(List<Double> objectReturn) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.copyOf(objectReturn));
    return columnValueSelector;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_6_1
#### Test Case Name: `testSanity`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayDoubleGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `columnValueSelector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    @Test
    public void testSanity() {
-    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
-    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of(1.0, 2.0));
+    ColumnValueSelector columnValueSelector = createMockColumnValueSelector(ImmutableList.of(1.0, 2.0));
    Assert.assertEquals(0, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(0);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(1.0, 2.0)), row.get(0));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSanity() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of(1.0, 2.0));
    Assert.assertEquals(0, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(0);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(1.0, 2.0)), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ColumnValueSelector createMockColumnValueSelector(List<Double> objectReturn) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.copyOf(objectReturn));
    return columnValueSelector;
}
```
</details>

---
#### Test Case ID #druid_Test_6_2
#### Test Case Name: `testAddingInDictionary`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayDoubleGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `columnValueSelector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testAddingInDictionary() {
-    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
-    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of(4.0, 2.0));
+    ColumnValueSelector columnValueSelector = createMockColumnValueSelector(ImmutableList.of(4.0, 2.0));
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4.0, 2.0)), row.get(0));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testAddingInDictionary() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of(4.0, 2.0));
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4.0, 2.0)), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ColumnValueSelector createMockColumnValueSelector(List<Double> objectReturn) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.copyOf(objectReturn));
    return columnValueSelector;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_7
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.segment.ColumnValueSelector`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
private static ColumnValueSelector createMockColumnValueSelector(List<Long> objectReturnList) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.copyOf(objectReturnList));
    return columnValueSelector;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_7_1
#### Test Case Name: `testSanity`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayLongGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `columnValueSelector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testSanity() {
-    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
-    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of(1L, 2L));
+    ColumnValueSelector columnValueSelector = createMockColumnValueSelector(ImmutableList.of(1L, 2L));
    Assert.assertEquals(0, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(0);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(1L, 2L)), row.get(0));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSanity() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of(1L, 2L));
    Assert.assertEquals(0, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(0);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(1L, 2L)), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ColumnValueSelector createMockColumnValueSelector(List<Long> objectReturnList) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.copyOf(objectReturnList));
    return columnValueSelector;
}
```
</details>

---
#### Test Case ID #druid_Test_7_2
#### Test Case Name: `testAddingInDictionary`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayLongGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `columnValueSelector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testAddingInDictionary() {
-    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
-    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of(4L, 2L));
+    ColumnValueSelector columnValueSelector = createMockColumnValueSelector(ImmutableList.of(4L, 2L));
     Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
     GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
     Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
     ResultRow row = ResultRow.create(1);
     buffer1.putInt(3);
     strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
     Assert.assertEquals(new ComparableList<>(ImmutableList.of(4L, 2L)), row.get(0));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testAddingInDictionary() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of(4L, 2L));
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4L, 2L)), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ColumnValueSelector createMockColumnValueSelector(List<Long> objectReturnList) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.copyOf(objectReturnList));
    return columnValueSelector;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_8
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.segment.ColumnValueSelector`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
private static ColumnValueSelector createMockColumnValueSelector(ImmutableList<String> objectReturn) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(objectReturn);
    return columnValueSelector;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_8_1
#### Test Case Name: `testSanity`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayStringGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `columnValueSelector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testSanity() {
-    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
-    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of("a", "b"));
+    ColumnValueSelector columnValueSelector = createMockColumnValueSelector(ImmutableList.of("a", "b"));
     Assert.assertEquals(0, strategy.computeDictionaryId(columnValueSelector));
     GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
     Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
     ResultRow row = ResultRow.create(1);
     buffer1.putInt(0);
     strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
     Assert.assertEquals(ComparableStringArray.of("a", "b"), row.get(0));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSanity() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of("a", "b"));
    Assert.assertEquals(0, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(0);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(ComparableStringArray.of("a", "b"), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ColumnValueSelector createMockColumnValueSelector(ImmutableList<String> objectReturn) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(objectReturn);
    return columnValueSelector;
}
```
</details>

---
#### Test Case ID #druid_Test_8_2
#### Test Case Name: `testAddingInDictionary`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayStringGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `columnValueSelector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testAddingInDictionary() {
-    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
-    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of("f", "a"));
+    ColumnValueSelector columnValueSelector = createMockColumnValueSelector(ImmutableList.of("f", "a"));
     Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
     GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
     Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
     ResultRow row = ResultRow.create(1);
     buffer1.putInt(3);
     strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
     Assert.assertEquals(ComparableStringArray.of("f", "a"), row.get(0));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testAddingInDictionary() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of("f", "a"));
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(ComparableStringArray.of("f", "a"), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ColumnValueSelector createMockColumnValueSelector(ImmutableList<String> objectReturn) {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(objectReturn);
    return columnValueSelector;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_9
- **Scope**: class level
- **Mocked Class**: `org.apache.druid.server.coordinator.DruidCoordinatorRuntimeParams`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
public class MockDruidCoordinatorRuntimeParams {
  public static DruidCoordinatorRuntimeParams createMockDruidCoordinatorRuntimeParams(CoordinatorRunStats coordinatorRunStats) {
    DruidCoordinatorRuntimeParams mockDruidCoordinatorRuntimeParams = Mockito.mock(DruidCoordinatorRuntimeParams.class);
    Mockito.when(mockDruidCoordinatorRuntimeParams.getCoordinatorStats()).thenReturn(coordinatorRunStats);
    return mockDruidCoordinatorRuntimeParams;
  }
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_9_1
#### Test Case Name: `testRunNotSkipIfLastRunMoreThanPeriod`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\KillDatasourceMetadataTest.java`)
#### Mock Object Variable Name: `mockDruidCoordinatorRuntimeParams`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testRunNotSkipIfLastRunMoreThanPeriod() {
-    Mockito.when(mockDruidCoordinatorRuntimeParams.getCoordinatorStats()).thenReturn(runStats);
+    mockDruidCoordinatorRuntimeParams = MockDruidCoordinatorRuntimeParams.createMockDruidCoordinatorRuntimeParams(runStats);
     TestDruidCoordinatorConfig druidCoordinatorConfig = new TestDruidCoordinatorConfig.Builder().withMetadataStoreManagementPeriod(new Duration("PT5S")).withCoordinatorDatasourceKillPeriod(new Duration("PT6S")).withCoordinatorDatasourceKillDurationToRetain(new Duration("PT1S")).withCoordinatorKillMaxSegments(10).withCoordinatorKillIgnoreDurationToRetain(false).build();
     killDatasourceMetadata = new KillDatasourceMetadata(druidCoordinatorConfig, mockIndexerMetadataStorageCoordinator, mockMetadataSupervisorManager);
     killDatasourceMetadata.run(mockDruidCoordinatorRuntimeParams);
     Mockito.verify(mockIndexerMetadataStorageCoordinator).removeDataSourceMetadataOlderThan(ArgumentMatchers.anyLong(), ArgumentMatchers.anySet());
     Assert.assertTrue(runStats.hasStat(Stats.Kill.DATASOURCES));
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void testRunNotSkipIfLastRunMoreThanPeriod() {
    Mockito.when(mockDruidCoordinatorRuntimeParams.getCoordinatorStats()).thenReturn(runStats);
    TestDruidCoordinatorConfig druidCoordinatorConfig = new TestDruidCoordinatorConfig.Builder().withMetadataStoreManagementPeriod(new Duration("PT5S")).withCoordinatorDatasourceKillPeriod(new Duration("PT6S")).withCoordinatorDatasourceKillDurationToRetain(new Duration("PT1S")).withCoordinatorKillMaxSegments(10).withCoordinatorKillIgnoreDurationToRetain(false).build();
    killDatasourceMetadata = new KillDatasourceMetadata(druidCoordinatorConfig, mockIndexerMetadataStorageCoordinator, mockMetadataSupervisorManager);
    killDatasourceMetadata.run(mockDruidCoordinatorRuntimeParams);
    Mockito.verify(mockIndexerMetadataStorageCoordinator).removeDataSourceMetadataOlderThan(ArgumentMatchers.anyLong(), ArgumentMatchers.anySet());
    Assert.assertTrue(runStats.hasStat(Stats.Kill.DATASOURCES));
}
@Before
public void setup() {
    runStats = new CoordinatorRunStats();
    Mockito.when(mockDruidCoordinatorRuntimeParams.getCoordinatorStats()).thenReturn(runStats);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDruidCoordinatorRuntimeParams {
  public static DruidCoordinatorRuntimeParams createMockDruidCoordinatorRuntimeParams(CoordinatorRunStats coordinatorRunStats) {
    DruidCoordinatorRuntimeParams mockDruidCoordinatorRuntimeParams = Mockito.mock(DruidCoordinatorRuntimeParams.class);
    Mockito.when(mockDruidCoordinatorRuntimeParams.getCoordinatorStats()).thenReturn(coordinatorRunStats);
    return mockDruidCoordinatorRuntimeParams;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_9_2
#### Test Case Name: `testRun`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\KillSupervisorsCustomDutyTest.java`)
#### Mock Object Variable Name: `mockDruidCoordinatorRuntimeParams`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testRun() {
     final CoordinatorRunStats runStats = new CoordinatorRunStats();
-    Mockito.when(mockDruidCoordinatorRuntimeParams.getCoordinatorStats()).thenReturn(runStats);
+    mockDruidCoordinatorRuntimeParams = MockDruidCoordinatorRuntimeParams.createMockDruidCoordinatorRuntimeParams(runStats);
     killSupervisors = new KillSupervisorsCustomDuty(new Duration("PT1S"), mockMetadataSupervisorManager, coordinatorConfig);
     killSupervisors.run(mockDruidCoordinatorRuntimeParams);
     Mockito.verify(mockMetadataSupervisorManager).removeTerminatedSupervisorsOlderThan(ArgumentMatchers.anyLong());
     Assert.assertTrue(runStats.hasStat(Stats.Kill.SUPERVISOR_SPECS));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void testRun() {
    final CoordinatorRunStats runStats = new CoordinatorRunStats();
    Mockito.when(mockDruidCoordinatorRuntimeParams.getCoordinatorStats()).thenReturn(runStats);
    killSupervisors = new KillSupervisorsCustomDuty(new Duration("PT1S"), mockMetadataSupervisorManager, coordinatorConfig);
    killSupervisors.run(mockDruidCoordinatorRuntimeParams);
    Mockito.verify(mockMetadataSupervisorManager).removeTerminatedSupervisorsOlderThan(ArgumentMatchers.anyLong());
    Assert.assertTrue(runStats.hasStat(Stats.Kill.SUPERVISOR_SPECS));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDruidCoordinatorRuntimeParams {
  public static DruidCoordinatorRuntimeParams createMockDruidCoordinatorRuntimeParams(CoordinatorRunStats coordinatorRunStats) {
    DruidCoordinatorRuntimeParams mockDruidCoordinatorRuntimeParams = Mockito.mock(DruidCoordinatorRuntimeParams.class);
    Mockito.when(mockDruidCoordinatorRuntimeParams.getCoordinatorStats()).thenReturn(coordinatorRunStats);
    return mockDruidCoordinatorRuntimeParams;
  }
}
```
</details>

---
## Mock Clone Instance #druid_MCI_10
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.data.input.impl.CloudObjectInputSource`
- **Test Case Count**: 4
- **MO Count**: 4

### Reusable Method
```java
private static CloudObjectInputSource createMockCloudObjectInputSource(String scheme, List<URI> uris, List<CloudObjectLocation> objects, List<CloudObjectLocation> objectsBeforeGlob, String objectGlob, MockSplitWidget splitWidget) {
    CloudObjectInputSource inputSource = Mockito.mock(
        CloudObjectInputSource.class,
        Mockito.withSettings()
            .useConstructor(scheme, uris, null, objectsBeforeGlob, objectGlob)
            .defaultAnswer(Mockito.CALLS_REAL_METHODS)
    );
    Mockito.when(inputSource.getSplitWidget()).thenReturn(splitWidget);
    return inputSource;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_10_1
#### Test Case Name: `testWithUrisFilter`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\impl\CloudObjectInputSourceTest.java`)
#### Mock Object Variable Name: `inputSource`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testWithUrisFilter() {
-    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS2, null, null, "**.csv").defaultAnswer(Mockito.CALLS_REAL_METHODS));
-    Mockito.when(inputSource.getSplitWidget()).thenReturn(new MockSplitWidget());
+    CloudObjectInputSource inputSource = createMockCloudObjectInputSource(SCHEME, URIS2, null, null, "**.csv", new MockSplitWidget());
    Stream<InputSplit<List<CloudObjectLocation>>> splits = inputSource.createSplits(new JsonInputFormat(JSONPathSpec.DEFAULT, null, null, null, null), new MaxSizeSplitHintSpec(null, 1));
    List<CloudObjectLocation> returnedLocations = splits.map(InputSplit::get).collect(Collectors.toList()).get(0);
    List<URI> returnedLocationUris = returnedLocations.stream().map(object -> object.toUri(SCHEME)).collect(Collectors.toList());
    Assert.assertEquals("**.csv", inputSource.getObjectGlob());
    Assert.assertEquals(URIS, returnedLocationUris);
    final List<InputEntity> entities = Lists.newArrayList(inputSource.getInputEntities(new JsonInputFormat(null, null, null, null, null)));
    Assert.assertEquals(URIS.size(), entities.size());
}
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testWithUrisFilter() {
    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS2, null, null, "**.csv").defaultAnswer(Mockito.CALLS_REAL_METHODS));
    Mockito.when(inputSource.getSplitWidget()).thenReturn(new MockSplitWidget());
    Stream<InputSplit<List<CloudObjectLocation>>> splits = inputSource.createSplits(new JsonInputFormat(JSONPathSpec.DEFAULT, null, null, null, null), new MaxSizeSplitHintSpec(null, 1));
    List<CloudObjectLocation> returnedLocations = splits.map(InputSplit::get).collect(Collectors.toList()).get(0);
    List<URI> returnedLocationUris = returnedLocations.stream().map(object -> object.toUri(SCHEME)).collect(Collectors.toList());
    Assert.assertEquals("**.csv", inputSource.getObjectGlob());
    Assert.assertEquals(URIS, returnedLocationUris);
    final List<InputEntity> entities = Lists.newArrayList(inputSource.getInputEntities(new JsonInputFormat(null, null, null, null, null)));
    Assert.assertEquals(URIS.size(), entities.size());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static CloudObjectInputSource createMockCloudObjectInputSource(String scheme, List<URI> uris, List<CloudObjectLocation> objects, List<CloudObjectLocation> objectsBeforeGlob, String objectGlob, MockSplitWidget splitWidget) {
    CloudObjectInputSource inputSource = Mockito.mock(
        CloudObjectInputSource.class,
        Mockito.withSettings()
            .useConstructor(scheme, uris, null, objectsBeforeGlob, objectGlob)
            .defaultAnswer(Mockito.CALLS_REAL_METHODS)
    );
    Mockito.when(inputSource.getSplitWidget()).thenReturn(splitWidget);
    return inputSource;
}
```
</details>

---
#### Test Case ID #druid_Test_10_2
#### Test Case Name: `testWithUris`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\impl\CloudObjectInputSourceTest.java`)
#### Mock Object Variable Name: `inputSource`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testWithUris() {
-    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
-    Mockito.when(inputSource.getSplitWidget()).thenReturn(new MockSplitWidget());
+    CloudObjectInputSource inputSource = createMockCloudObjectInputSource(SCHEME, URIS, null, null, null, new MockSplitWidget());
     Stream<InputSplit<List<CloudObjectLocation>>> splits = inputSource.createSplits(new JsonInputFormat(JSONPathSpec.DEFAULT, null, null, null, null), new MaxSizeSplitHintSpec(null, 1));
     List<CloudObjectLocation> returnedLocations = splits.map(InputSplit::get).collect(Collectors.toList()).get(0);
     List<URI> returnedLocationUris = returnedLocations.stream().map(object -> object.toUri(SCHEME)).collect(Collectors.toList());
     Assert.assertEquals(null, inputSource.getObjectGlob());
     Assert.assertEquals(URIS, returnedLocationUris);
     final List<InputEntity> entities = Lists.newArrayList(inputSource.getInputEntities(new JsonInputFormat(null, null, null, null, null)));
     Assert.assertEquals(URIS.size(), entities.size());
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testWithUris() {
    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
    Mockito.when(inputSource.getSplitWidget()).thenReturn(new MockSplitWidget());
    Stream<InputSplit<List<CloudObjectLocation>>> splits = inputSource.createSplits(new JsonInputFormat(JSONPathSpec.DEFAULT, null, null, null, null), new MaxSizeSplitHintSpec(null, 1));
    List<CloudObjectLocation> returnedLocations = splits.map(InputSplit::get).collect(Collectors.toList()).get(0);
    List<URI> returnedLocationUris = returnedLocations.stream().map(object -> object.toUri(SCHEME)).collect(Collectors.toList());
    Assert.assertEquals(null, inputSource.getObjectGlob());
    Assert.assertEquals(URIS, returnedLocationUris);
    final List<InputEntity> entities = Lists.newArrayList(inputSource.getInputEntities(new JsonInputFormat(null, null, null, null, null)));
    Assert.assertEquals(URIS.size(), entities.size());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static CloudObjectInputSource createMockCloudObjectInputSource(String scheme, List<URI> uris, List<CloudObjectLocation> objects, List<CloudObjectLocation> objectsBeforeGlob, String objectGlob, MockSplitWidget splitWidget) {
    CloudObjectInputSource inputSource = Mockito.mock(
        CloudObjectInputSource.class,
        Mockito.withSettings()
            .useConstructor(scheme, uris, null, objectsBeforeGlob, objectGlob)
            .defaultAnswer(Mockito.CALLS_REAL_METHODS)
    );
    Mockito.when(inputSource.getSplitWidget()).thenReturn(splitWidget);
    return inputSource;
}
```
</details>

---
#### Test Case ID #druid_Test_10_3
#### Test Case Name: `testWithObjectsFilter`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\impl\CloudObjectInputSourceTest.java`)
#### Mock Object Variable Name: `inputSource`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testWithObjectsFilter() {
-    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, null, null, OBJECTS_BEFORE_GLOB, "**.csv").defaultAnswer(Mockito.CALLS_REAL_METHODS));
-    Mockito.when(inputSource.getSplitWidget()).thenReturn(new MockSplitWidget());
+    CloudObjectInputSource inputSource = createMockCloudObjectInputSource(SCHEME, null, OBJECTS, OBJECTS_BEFORE_GLOB, "**.csv", new MockSplitWidget());
    Stream<InputSplit<List<CloudObjectLocation>>> splits = inputSource.createSplits(new JsonInputFormat(JSONPathSpec.DEFAULT, null, null, null, null), new MaxSizeSplitHintSpec(null, 1));
    List<CloudObjectLocation> returnedLocations = splits.map(InputSplit::get).collect(Collectors.toList()).get(0);
    List<URI> returnedLocationUris = returnedLocations.stream().map(object -> object.toUri(SCHEME)).collect(Collectors.toList());
    Assert.assertEquals("**.csv", inputSource.getObjectGlob());
    Assert.assertEquals(URIS, returnedLocationUris);
    final List<InputEntity> entities = Lists.newArrayList(inputSource.getInputEntities(new JsonInputFormat(null, null, null, null, null)));
    Assert.assertEquals(OBJECTS.size(), entities.size());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testWithObjectsFilter() {
    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, null, null, OBJECTS_BEFORE_GLOB, "**.csv").defaultAnswer(Mockito.CALLS_REAL_METHODS));
    Mockito.when(inputSource.getSplitWidget()).thenReturn(new MockSplitWidget());
    Stream<InputSplit<List<CloudObjectLocation>>> splits = inputSource.createSplits(new JsonInputFormat(JSONPathSpec.DEFAULT, null, null, null, null), new MaxSizeSplitHintSpec(null, 1));
    List<CloudObjectLocation> returnedLocations = splits.map(InputSplit::get).collect(Collectors.toList()).get(0);
    List<URI> returnedLocationUris = returnedLocations.stream().map(object -> object.toUri(SCHEME)).collect(Collectors.toList());
    Assert.assertEquals("**.csv", inputSource.getObjectGlob());
    Assert.assertEquals(URIS, returnedLocationUris);
    final List<InputEntity> entities = Lists.newArrayList(inputSource.getInputEntities(new JsonInputFormat(null, null, null, null, null)));
    Assert.assertEquals(OBJECTS.size(), entities.size());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static CloudObjectInputSource createMockCloudObjectInputSource(String scheme, List<URI> uris, List<CloudObjectLocation> objects, List<CloudObjectLocation> objectsBeforeGlob, String objectGlob, MockSplitWidget splitWidget) {
    CloudObjectInputSource inputSource = Mockito.mock(
        CloudObjectInputSource.class,
        Mockito.withSettings()
            .useConstructor(scheme, uris, null, objectsBeforeGlob, objectGlob)
            .defaultAnswer(Mockito.CALLS_REAL_METHODS)
    );
    Mockito.when(inputSource.getSplitWidget()).thenReturn(splitWidget);
    return inputSource;
}
```
</details>

---
#### Test Case ID #druid_Test_10_4
#### Test Case Name: `testWithObjects`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\impl\CloudObjectInputSourceTest.java`)
#### Mock Object Variable Name: `inputSource`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testWithObjects() {
-    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, null, null, OBJECTS, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
-    Mockito.when(inputSource.getSplitWidget()).thenReturn(new MockSplitWidget());
+    CloudObjectInputSource inputSource = createMockCloudObjectInputSource(SCHEME, null, OBJECTS, null, null, new MockSplitWidget());
    Stream<InputSplit<List<CloudObjectLocation>>> splits = inputSource.createSplits(new JsonInputFormat(JSONPathSpec.DEFAULT, null, null, null, null), new MaxSizeSplitHintSpec(null, 1));
    List<CloudObjectLocation> returnedLocations = splits.map(InputSplit::get).collect(Collectors.toList()).get(0);
    Assert.assertNull(inputSource.getObjectGlob());
    Assert.assertEquals(OBJECTS, returnedLocations);
    final List<InputEntity> entities = Lists.newArrayList(inputSource.getInputEntities(new JsonInputFormat(null, null, null, null, null)));
    Assert.assertEquals(OBJECTS.size(), entities.size());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testWithObjects() {
    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, null, null, OBJECTS, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
    Mockito.when(inputSource.getSplitWidget()).thenReturn(new MockSplitWidget());
    Stream<InputSplit<List<CloudObjectLocation>>> splits = inputSource.createSplits(new JsonInputFormat(JSONPathSpec.DEFAULT, null, null, null, null), new MaxSizeSplitHintSpec(null, 1));
    List<CloudObjectLocation> returnedLocations = splits.map(InputSplit::get).collect(Collectors.toList()).get(0);
    Assert.assertNull(inputSource.getObjectGlob());
    Assert.assertEquals(OBJECTS, returnedLocations);
    final List<InputEntity> entities = Lists.newArrayList(inputSource.getInputEntities(new JsonInputFormat(null, null, null, null, null)));
    Assert.assertEquals(OBJECTS.size(), entities.size());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static CloudObjectInputSource createMockCloudObjectInputSource(String scheme, List<URI> uris, List<CloudObjectLocation> objects, List<CloudObjectLocation> objectsBeforeGlob, String objectGlob, MockSplitWidget splitWidget) {
    CloudObjectInputSource inputSource = Mockito.mock(
        CloudObjectInputSource.class,
        Mockito.withSettings()
            .useConstructor(scheme, uris, null, objectsBeforeGlob, objectGlob)
            .defaultAnswer(Mockito.CALLS_REAL_METHODS)
    );
    Mockito.when(inputSource.getSplitWidget()).thenReturn(splitWidget);
    return inputSource;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_11
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.data.input.impl.CloudObjectInputSource`
- **Test Case Count**: 4
- **MO Count**: 4

### Reusable Method
```java
// === Declare in class scope ===
private CloudObjectInputSource inputSource;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
}

// === Replace local variable in test with ===
inputSource;

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_11_1
#### Test Case Name: `testGetUris`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\impl\CloudObjectInputSourceTest.java`)
#### Mock Object Variable Name: `inputSource`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testGetUris() {
-    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
+    // removed local mock; replaced with global field `inputSource`
     Assert.assertEquals(URIS, inputSource.getUris());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testGetUris() {
    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
    Assert.assertEquals(URIS, inputSource.getUris());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private CloudObjectInputSource inputSource;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
}

// === Replace local variable in test with ===
inputSource;

```
</details>

---
#### Test Case ID #druid_Test_11_2
#### Test Case Name: `testGetPrefixes`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\impl\CloudObjectInputSourceTest.java`)
#### Mock Object Variable Name: `inputSource`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testGetPrefixes() {
-    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, null, PREFIXES, null, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
+    // removed local mock; replaced with global field `inputSource`
     Assert.assertEquals(PREFIXES, inputSource.getPrefixes());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testGetPrefixes() {
    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, null, PREFIXES, null, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
    Assert.assertEquals(PREFIXES, inputSource.getPrefixes());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private CloudObjectInputSource inputSource;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
}

// === Replace local variable in test with ===
inputSource;

```
</details>

---
#### Test Case ID #druid_Test_11_3
#### Test Case Name: `testGetObjectGlob`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\impl\CloudObjectInputSourceTest.java`)
#### Mock Object Variable Name: `inputSource`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testGetObjectGlob() {
-    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, "**.parquet").defaultAnswer(Mockito.CALLS_REAL_METHODS));
+    // removed local mock; replaced with global field `inputSource`
     Assert.assertEquals("**.parquet", inputSource.getObjectGlob());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testGetObjectGlob() {
    CloudObjectInputSource inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, "**.parquet").defaultAnswer(Mockito.CALLS_REAL_METHODS));
    Assert.assertEquals("**.parquet", inputSource.getObjectGlob());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private CloudObjectInputSource inputSource;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
}

// === Replace local variable in test with ===
inputSource;

```
</details>

---
#### Test Case ID #druid_Test_11_4
#### Test Case Name: `testInequality`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\impl\CloudObjectInputSourceTest.java`)
#### Mock Object Variable Name: `inputSource1`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testInequality() {
-    CloudObjectInputSource inputSource1 = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, "**.parquet").defaultAnswer(Mockito.CALLS_REAL_METHODS));
+    // removed local mock; replaced with global field `inputSource`
     CloudObjectInputSource inputSource2 = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, "**.csv").defaultAnswer(Mockito.CALLS_REAL_METHODS));
@@
-    Assert.assertEquals("**.parquet", inputSource1.getObjectGlob());
+    Assert.assertEquals("**.parquet", inputSource.getObjectGlob());
     Assert.assertEquals("**.csv", inputSource2.getObjectGlob());
-    Assert.assertFalse(inputSource2.equals(inputSource1));
+    Assert.assertFalse(inputSource2.equals(inputSource));
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testInequality() {
    CloudObjectInputSource inputSource1 = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, "**.parquet").defaultAnswer(Mockito.CALLS_REAL_METHODS));
    CloudObjectInputSource inputSource2 = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, "**.csv").defaultAnswer(Mockito.CALLS_REAL_METHODS));
    Assert.assertEquals("**.parquet", inputSource1.getObjectGlob());
    Assert.assertEquals("**.csv", inputSource2.getObjectGlob());
    Assert.assertFalse(inputSource2.equals(inputSource1));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private CloudObjectInputSource inputSource;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    inputSource = Mockito.mock(CloudObjectInputSource.class, Mockito.withSettings().useConstructor(SCHEME, URIS, null, null, null).defaultAnswer(Mockito.CALLS_REAL_METHODS));
}

// === Replace local variable in test with ===
inputSource;

```
</details>

---
## Mock Clone Instance #druid_MCI_12
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.query.lookup.namespace.ExtractionNamespace`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
// === Declare in class scope ===
private ExtractionNamespace en1, en2;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    en1 = mock(ExtractionNamespace.class);
en2 = mock(ExtractionNamespace.class);
}

// === Replace local variable in test with ===
en1, en2

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_12_1
#### Test Case Name: `testReplaces`(File: `C:\Java_projects\Apache\druid\extensions-core\lookups-cached-global\src\test\java\org\apache\druid\query\lookup\NamespaceLookupExtractorFactoryTest.java`)
#### Mock Object Variable Name: `en1`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testReplaces() {
+    // Cannot refactor mock `en1`: declared together with `en2` in a single line; split required before refactoring
     final ExtractionNamespace en1 = mock(ExtractionNamespace.class), en2 = mock(ExtractionNamespace.class);
     final NamespaceLookupExtractorFactory f1 = new NamespaceLookupExtractorFactory(en1, scheduler);
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testReplaces() {
    final ExtractionNamespace en1 = mock(ExtractionNamespace.class), en2 = mock(ExtractionNamespace.class);
    final NamespaceLookupExtractorFactory f1 = new NamespaceLookupExtractorFactory(en1, scheduler);
    final NamespaceLookupExtractorFactory f2 = new NamespaceLookupExtractorFactory(en2, scheduler);
    final NamespaceLookupExtractorFactory f1b = new NamespaceLookupExtractorFactory(en1, scheduler);
    Assert.assertTrue(f1.replaces(f2));
    Assert.assertTrue(f2.replaces(f1));
    Assert.assertFalse(f1.replaces(f1b));
    Assert.assertFalse(f1b.replaces(f1));
    Assert.assertFalse(f1.replaces(f1));
    Assert.assertTrue(f1.replaces(mock(LookupExtractorFactory.class)));
    verifyNoInteractions(en1, en2);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ExtractionNamespace en1, en2;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    en1 = mock(ExtractionNamespace.class);
en2 = mock(ExtractionNamespace.class);
}

// === Replace local variable in test with ===
en1, en2

```
</details>

---
#### Test Case ID #druid_Test_12_2
#### Test Case Name: `testExceptionalIntrospectionHandler`(File: `C:\Java_projects\Apache\druid\extensions-core\lookups-cached-global\src\test\java\org\apache\druid\query\lookup\NamespaceLookupExtractorFactoryTest.java`)
#### Mock Object Variable Name: `extractionNamespace`
<summary>Suggested Diff</summary>

```diff
@@
+    // Cannot refactor mock `extractionNamespace`: ambiguous mapping to multiple global mocks ('en1, en2')
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testExceptionalIntrospectionHandler() throws Exception {
    final ExtractionNamespace extractionNamespace = mock(ExtractionNamespace.class);
    when(scheduler.scheduleAndWait(eq(extractionNamespace), anyLong())).thenReturn(entry);
    final LookupExtractorFactory lookupExtractorFactory = new NamespaceLookupExtractorFactory(extractionNamespace, scheduler);
    Assert.assertTrue(lookupExtractorFactory.start());
    final LookupIntrospectHandler handler = lookupExtractorFactory.getIntrospectHandler();
    Assert.assertNotNull(handler);
    final Class<? extends LookupIntrospectHandler> clazz = handler.getClass();
    verify(scheduler).scheduleAndWait(eq(extractionNamespace), anyLong());
    verifyNoMoreInteractions(scheduler, entry, versionedCache);
    reset(scheduler, entry, versionedCache);
    when(entry.getCacheState()).thenReturn(CacheScheduler.NoCache.CACHE_NOT_INITIALIZED);
    final Response response = (Response) clazz.getMethod("getVersion").invoke(handler);
    Assert.assertEquals(404, response.getStatus());
    verify(entry).getCacheState();
    validateNotFound("getKeys", handler, clazz);
    validateNotFound("getValues", handler, clazz);
    validateNotFound("getMap", handler, clazz);
    verifyNoMoreInteractions(scheduler, entry, versionedCache);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ExtractionNamespace en1, en2;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    en1 = mock(ExtractionNamespace.class);
en2 = mock(ExtractionNamespace.class);
}

// === Replace local variable in test with ===
en1, en2

```
</details>

---
## Mock Clone Instance #druid_MCI_13
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.indexing.common.task.TaskResource`
- **Test Case Count**: 1
- **MO Count**: 3

### Reusable Method
```java
private static TaskResource createMockTaskResource(int requiredCapacity) {
    TaskResource taskResource = mock(TaskResource.class);
    when(taskResource.getRequiredCapacity()).thenReturn(requiredCapacity);
    return taskResource;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_13_1
#### Test Case Name: `test_canRunTask`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\overlord\ImmutableWorkerInfoTest.java`)
#### Mock Object Variable Name: `taskResource0`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
     ImmutableWorkerInfo workerInfo = new ImmutableWorkerInfo(new Worker("http", "testWorker2", "192.0.0.1", 10, "v1", WorkerConfig.DEFAULT_CATEGORY), 6, 0, ImmutableSet.of("grp1", "grp2"), ImmutableSet.of("task1", "task2"), DateTimes.of("2015-01-01T01:01:02Z"));
     // Parallel index task
-    TaskResource taskResource0 = mock(TaskResource.class);
-    when(taskResource0.getRequiredCapacity()).thenReturn(3);
+    TaskResource taskResource0 = createMockTaskResource(3);
     Task parallelIndexTask = mock(ParallelIndexSupervisorTask.class);
     when(parallelIndexTask.getType()).thenReturn(ParallelIndexSupervisorTask.TYPE);
     when(parallelIndexTask.getTaskResource()).thenReturn(taskResource0);
     // Since task satisifies parallel and total slot constraints, can run
     Assert.assertTrue(workerInfo.canRunTask(parallelIndexTask, 0.5));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void test_canRunTask() {
    ImmutableWorkerInfo workerInfo = new ImmutableWorkerInfo(new Worker("http", "testWorker2", "192.0.0.1", 10, "v1", WorkerConfig.DEFAULT_CATEGORY), 6, 0, ImmutableSet.of("grp1", "grp2"), ImmutableSet.of("task1", "task2"), DateTimes.of("2015-01-01T01:01:02Z"));
    // Parallel index task
    TaskResource taskResource0 = mock(TaskResource.class);
    when(taskResource0.getRequiredCapacity()).thenReturn(3);
    Task parallelIndexTask = mock(ParallelIndexSupervisorTask.class);
    when(parallelIndexTask.getType()).thenReturn(ParallelIndexSupervisorTask.TYPE);
    when(parallelIndexTask.getTaskResource()).thenReturn(taskResource0);
    // Since task satisifies parallel and total slot constraints, can run
    Assert.assertTrue(workerInfo.canRunTask(parallelIndexTask, 0.5));
    // Since task fails the parallel slot constraint, it cannot run (3 > 1)
    Assert.assertFalse(workerInfo.canRunTask(parallelIndexTask, 0.1));
    // Some other indexing task
    TaskResource taskResource1 = mock(TaskResource.class);
    when(taskResource1.getRequiredCapacity()).thenReturn(5);
    Task anyOtherTask = mock(IndexTask.class);
    when(anyOtherTask.getType()).thenReturn("index");
    when(anyOtherTask.getTaskResource()).thenReturn(taskResource1);
    // Not a parallel index task ->  satisfies parallel index constraint
    // But does not satisfy the total slot constraint and cannot run (11 > 10)
    Assert.assertFalse(workerInfo.canRunTask(anyOtherTask, 0.5));
    // Task has an availability conflict ("grp1")
    TaskResource taskResource2 = mock(TaskResource.class);
    when(taskResource2.getRequiredCapacity()).thenReturn(1);
    when(taskResource2.getAvailabilityGroup()).thenReturn("grp1");
    Task grp1Task = mock(IndexTask.class);
    when(grp1Task.getType()).thenReturn("blah");
    when(grp1Task.getTaskResource()).thenReturn(taskResource2);
    // Satisifies parallel index and total index slot constraints but cannot run due availability
    Assert.assertFalse(workerInfo.canRunTask(grp1Task, 0.3));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static TaskResource createMockTaskResource(int requiredCapacity) {
    TaskResource taskResource = mock(TaskResource.class);
    when(taskResource.getRequiredCapacity()).thenReturn(requiredCapacity);
    return taskResource;
}
```
</details>

---
#### Test Case ID #druid_Test_13_2
#### Test Case Name: `test_canRunTask`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\overlord\ImmutableWorkerInfoTest.java`)
#### Mock Object Variable Name: `taskResource1`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    // Since task fails the parallel slot constraint, it cannot run (3 > 1)
    Assert.assertFalse(workerInfo.canRunTask(parallelIndexTask, 0.1));
    // Some other indexing task
-    TaskResource taskResource1 = mock(TaskResource.class);
-    when(taskResource1.getRequiredCapacity()).thenReturn(5);
+    TaskResource taskResource1 = createMockTaskResource(5);
    Task anyOtherTask = mock(IndexTask.class);
    when(anyOtherTask.getType()).thenReturn("index");
    when(anyOtherTask.getTaskResource()).thenReturn(taskResource1);
    // Not a parallel index task ->  satisfies parallel index constraint
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void test_canRunTask() {
    ImmutableWorkerInfo workerInfo = new ImmutableWorkerInfo(new Worker("http", "testWorker2", "192.0.0.1", 10, "v1", WorkerConfig.DEFAULT_CATEGORY), 6, 0, ImmutableSet.of("grp1", "grp2"), ImmutableSet.of("task1", "task2"), DateTimes.of("2015-01-01T01:01:02Z"));
    // Parallel index task
    TaskResource taskResource0 = mock(TaskResource.class);
    when(taskResource0.getRequiredCapacity()).thenReturn(3);
    Task parallelIndexTask = mock(ParallelIndexSupervisorTask.class);
    when(parallelIndexTask.getType()).thenReturn(ParallelIndexSupervisorTask.TYPE);
    when(parallelIndexTask.getTaskResource()).thenReturn(taskResource0);
    // Since task satisifies parallel and total slot constraints, can run
    Assert.assertTrue(workerInfo.canRunTask(parallelIndexTask, 0.5));
    // Since task fails the parallel slot constraint, it cannot run (3 > 1)
    Assert.assertFalse(workerInfo.canRunTask(parallelIndexTask, 0.1));
    // Some other indexing task
    TaskResource taskResource1 = mock(TaskResource.class);
    when(taskResource1.getRequiredCapacity()).thenReturn(5);
    Task anyOtherTask = mock(IndexTask.class);
    when(anyOtherTask.getType()).thenReturn("index");
    when(anyOtherTask.getTaskResource()).thenReturn(taskResource1);
    // Not a parallel index task ->  satisfies parallel index constraint
    // But does not satisfy the total slot constraint and cannot run (11 > 10)
    Assert.assertFalse(workerInfo.canRunTask(anyOtherTask, 0.5));
    // Task has an availability conflict ("grp1")
    TaskResource taskResource2 = mock(TaskResource.class);
    when(taskResource2.getRequiredCapacity()).thenReturn(1);
    when(taskResource2.getAvailabilityGroup()).thenReturn("grp1");
    Task grp1Task = mock(IndexTask.class);
    when(grp1Task.getType()).thenReturn("blah");
    when(grp1Task.getTaskResource()).thenReturn(taskResource2);
    // Satisifies parallel index and total index slot constraints but cannot run due availability
    Assert.assertFalse(workerInfo.canRunTask(grp1Task, 0.3));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static TaskResource createMockTaskResource(int requiredCapacity) {
    TaskResource taskResource = mock(TaskResource.class);
    when(taskResource.getRequiredCapacity()).thenReturn(requiredCapacity);
    return taskResource;
}
```
</details>

---
#### Test Case ID #druid_Test_13_3
#### Test Case Name: `test_canRunTask`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\overlord\ImmutableWorkerInfoTest.java`)
#### Mock Object Variable Name: `taskResource2`
<summary>Suggested Diff</summary>

```diff
@@
     // Task has an availability conflict ("grp1")
-    TaskResource taskResource2 = mock(TaskResource.class);
-    when(taskResource2.getRequiredCapacity()).thenReturn(1);
+    TaskResource taskResource2 = createMockTaskResource(1);
     when(taskResource2.getAvailabilityGroup()).thenReturn("grp1");
     Task grp1Task = mock(IndexTask.class);
     when(grp1Task.getType()).thenReturn("blah");
     when(grp1Task.getTaskResource()).thenReturn(taskResource2);
     // Satisifies parallel index and total index slot constraints but cannot run due availability
     Assert.assertFalse(workerInfo.canRunTask(grp1Task, 0.3));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void test_canRunTask() {
    ImmutableWorkerInfo workerInfo = new ImmutableWorkerInfo(new Worker("http", "testWorker2", "192.0.0.1", 10, "v1", WorkerConfig.DEFAULT_CATEGORY), 6, 0, ImmutableSet.of("grp1", "grp2"), ImmutableSet.of("task1", "task2"), DateTimes.of("2015-01-01T01:01:02Z"));
    // Parallel index task
    TaskResource taskResource0 = mock(TaskResource.class);
    when(taskResource0.getRequiredCapacity()).thenReturn(3);
    Task parallelIndexTask = mock(ParallelIndexSupervisorTask.class);
    when(parallelIndexTask.getType()).thenReturn(ParallelIndexSupervisorTask.TYPE);
    when(parallelIndexTask.getTaskResource()).thenReturn(taskResource0);
    // Since task satisifies parallel and total slot constraints, can run
    Assert.assertTrue(workerInfo.canRunTask(parallelIndexTask, 0.5));
    // Since task fails the parallel slot constraint, it cannot run (3 > 1)
    Assert.assertFalse(workerInfo.canRunTask(parallelIndexTask, 0.1));
    // Some other indexing task
    TaskResource taskResource1 = mock(TaskResource.class);
    when(taskResource1.getRequiredCapacity()).thenReturn(5);
    Task anyOtherTask = mock(IndexTask.class);
    when(anyOtherTask.getType()).thenReturn("index");
    when(anyOtherTask.getTaskResource()).thenReturn(taskResource1);
    // Not a parallel index task ->  satisfies parallel index constraint
    // But does not satisfy the total slot constraint and cannot run (11 > 10)
    Assert.assertFalse(workerInfo.canRunTask(anyOtherTask, 0.5));
    // Task has an availability conflict ("grp1")
    TaskResource taskResource2 = mock(TaskResource.class);
    when(taskResource2.getRequiredCapacity()).thenReturn(1);
    when(taskResource2.getAvailabilityGroup()).thenReturn("grp1");
    Task grp1Task = mock(IndexTask.class);
    when(grp1Task.getType()).thenReturn("blah");
    when(grp1Task.getTaskResource()).thenReturn(taskResource2);
    // Satisifies parallel index and total index slot constraints but cannot run due availability
    Assert.assertFalse(workerInfo.canRunTask(grp1Task, 0.3));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static TaskResource createMockTaskResource(int requiredCapacity) {
    TaskResource taskResource = mock(TaskResource.class);
    when(taskResource.getRequiredCapacity()).thenReturn(requiredCapacity);
    return taskResource;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_14
- **Scope**: class level
- **Mocked Class**: `org.apache.druid.indexing.common.actions.TaskActionToolbox`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
public class MockTaskActionToolbox {
  public static TaskActionToolbox createMockTaskActionToolbox(TaskRunner runner) {
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
    return toolbox;
  }
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_14_1
#### Test Case Name: `testFlow`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\actions\UpdateLocationActionTest.java`)
#### Mock Object Variable Name: `toolbox`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    Task task = NoopTask.create();
-    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    TaskRunner runner = mock(TaskRunner.class);
-    when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
+    TaskActionToolbox toolbox = MockTaskActionToolbox.createMockTaskActionToolbox(runner);
    action.perform(task, toolbox);
    verify(runner, times(1)).updateLocation(eq(task), eq(myLocation));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testFlow() throws UnknownHostException {
    // get my task location
    InetAddress hostName = InetAddress.getLocalHost();
    TaskLocation myLocation = TaskLocation.create(hostName.getHostAddress(), 1, 2);
    UpdateLocationAction action = new UpdateLocationAction(myLocation);
    Task task = NoopTask.create();
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    TaskRunner runner = mock(TaskRunner.class);
    when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
    action.perform(task, toolbox);
    verify(runner, times(1)).updateLocation(eq(task), eq(myLocation));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockTaskActionToolbox {
  public static TaskActionToolbox createMockTaskActionToolbox(TaskRunner runner) {
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
    return toolbox;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_14_2
#### Test Case Name: `testActionCallsTaskRunner`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\actions\UpdateStatusActionTest.java`)
#### Mock Object Variable Name: `toolbox`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    UpdateStatusAction action = new UpdateStatusAction("successful");
    Task task = NoopTask.create();
-    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    TaskRunner runner = mock(TaskRunner.class);
-    when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
+    TaskActionToolbox toolbox = MockTaskActionToolbox.createMockTaskActionToolbox(runner);
    action.perform(task, toolbox);
    verify(runner, times(1)).updateStatus(eq(task), eq(TaskStatus.success(task.getId())));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testActionCallsTaskRunner() {
    UpdateStatusAction action = new UpdateStatusAction("successful");
    Task task = NoopTask.create();
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    TaskRunner runner = mock(TaskRunner.class);
    when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
    action.perform(task, toolbox);
    verify(runner, times(1)).updateStatus(eq(task), eq(TaskStatus.success(task.getId())));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockTaskActionToolbox {
  public static TaskActionToolbox createMockTaskActionToolbox(TaskRunner runner) {
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
    return toolbox;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_14_3
#### Test Case Name: `testFailureScenario`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\actions\UpdateStatusActionTest.java`)
#### Mock Object Variable Name: `toolbox`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    UpdateStatusAction action = new UpdateStatusAction("failure");
    Task task = NoopTask.create();
-    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    TaskRunner runner = mock(TaskRunner.class);
-    when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
+    TaskActionToolbox toolbox = MockTaskActionToolbox.createMockTaskActionToolbox(runner);
    action.perform(task, toolbox);
    verify(runner, times(1)).updateStatus(eq(task), eq(TaskStatus.failure(task.getId(), "Error with task")));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testFailureScenario() {
    UpdateStatusAction action = new UpdateStatusAction("failure");
    Task task = NoopTask.create();
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    TaskRunner runner = mock(TaskRunner.class);
    when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
    action.perform(task, toolbox);
    verify(runner, times(1)).updateStatus(eq(task), eq(TaskStatus.failure(task.getId(), "Error with task")));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockTaskActionToolbox {
  public static TaskActionToolbox createMockTaskActionToolbox(TaskRunner runner) {
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
    return toolbox;
  }
}
```
</details>

---
## Mock Clone Instance #druid_MCI_15
- **Scope**: class level
- **Mocked Class**: `org.apache.druid.indexing.common.actions.TaskActionToolbox`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
public class MockTaskActionToolbox {
  public static TaskActionToolbox createMockTaskActionToolbox(com.google.common.base.Optional<?> taskRunnerReturn) {
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    when(toolbox.getTaskRunner()).thenReturn(taskRunnerReturn);
    return toolbox;
  }
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_15_1
#### Test Case Name: `testWithNoTaskRunner`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\actions\UpdateLocationActionTest.java`)
#### Mock Object Variable Name: `toolbox`
<summary>Suggested Diff</summary>

```diff
@@
    UpdateLocationAction action = new UpdateLocationAction(myLocation);
    Task task = NoopTask.create();
-    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
-    TaskRunner runner = mock(TaskRunner.class);
-    when(toolbox.getTaskRunner()).thenReturn(Optional.absent());
+    TaskActionToolbox toolbox = MockTaskActionToolbox.createMockTaskActionToolbox(Optional.absent());
+    TaskRunner runner = mock(TaskRunner.class);
    action.perform(task, toolbox);
    verify(runner, never()).updateStatus(any(), any());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testWithNoTaskRunner() throws UnknownHostException {
    // get my task location
    InetAddress hostName = InetAddress.getLocalHost();
    TaskLocation myLocation = TaskLocation.create(hostName.getHostAddress(), 1, 2);
    UpdateLocationAction action = new UpdateLocationAction(myLocation);
    Task task = NoopTask.create();
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    TaskRunner runner = mock(TaskRunner.class);
    when(toolbox.getTaskRunner()).thenReturn(Optional.absent());
    action.perform(task, toolbox);
    verify(runner, never()).updateStatus(any(), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockTaskActionToolbox {
  public static TaskActionToolbox createMockTaskActionToolbox(com.google.common.base.Optional<?> taskRunnerReturn) {
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    when(toolbox.getTaskRunner()).thenReturn(taskRunnerReturn);
    return toolbox;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_15_2
#### Test Case Name: `testNoTaskRunner`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\actions\UpdateStatusActionTest.java`)
#### Mock Object Variable Name: `toolbox`
<summary>Suggested Diff</summary>

```diff
@@
     UpdateStatusAction action = new UpdateStatusAction("successful");
     Task task = NoopTask.create();
-    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
-    TaskRunner runner = mock(TaskRunner.class);
-    when(toolbox.getTaskRunner()).thenReturn(Optional.absent());
+    TaskActionToolbox toolbox = MockTaskActionToolbox.createMockTaskActionToolbox(Optional.absent());
+    TaskRunner runner = mock(TaskRunner.class);
     action.perform(task, toolbox);
     verify(runner, never()).updateStatus(any(), any());
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testNoTaskRunner() {
    UpdateStatusAction action = new UpdateStatusAction("successful");
    Task task = NoopTask.create();
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    TaskRunner runner = mock(TaskRunner.class);
    when(toolbox.getTaskRunner()).thenReturn(Optional.absent());
    action.perform(task, toolbox);
    verify(runner, never()).updateStatus(any(), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockTaskActionToolbox {
  public static TaskActionToolbox createMockTaskActionToolbox(com.google.common.base.Optional<?> taskRunnerReturn) {
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    when(toolbox.getTaskRunner()).thenReturn(taskRunnerReturn);
    return toolbox;
  }
}
```
</details>

---
## Mock Clone Instance #druid_MCI_16
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.indexing.overlord.autoscaling.ProvisioningSchedulerConfig`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
private static ProvisioningSchedulerConfig createMockProvisioningSchedulerConfig(boolean doAutoscaleReturn) {
    ProvisioningSchedulerConfig provisioningSchedulerConfig = Mockito.mock(ProvisioningSchedulerConfig.class);
    Mockito.when(provisioningSchedulerConfig.isDoAutoscale()).thenReturn(doAutoscaleReturn);
    return provisioningSchedulerConfig;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_16_1
#### Test Case Name: `testBuildWithAutoScale`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\overlord\RemoteTaskRunnerFactoryTest.java`)
#### Mock Object Variable Name: `provisioningSchedulerConfig`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testBuildWithAutoScale() {
-    ProvisioningSchedulerConfig provisioningSchedulerConfig = Mockito.mock(ProvisioningSchedulerConfig.class);
-    Mockito.when(provisioningSchedulerConfig.isDoAutoscale()).thenReturn(true);
+    ProvisioningSchedulerConfig provisioningSchedulerConfig = createMockProvisioningSchedulerConfig(true);
     RemoteTaskRunnerFactory remoteTaskRunnerFactory = getTestRemoteTaskRunnerFactory(provisioningSchedulerConfig);
     Assert.assertNull(remoteTaskRunnerFactory.build().getProvisioningStrategy());
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testBuildWithAutoScale() {
    ProvisioningSchedulerConfig provisioningSchedulerConfig = Mockito.mock(ProvisioningSchedulerConfig.class);
    Mockito.when(provisioningSchedulerConfig.isDoAutoscale()).thenReturn(true);
    RemoteTaskRunnerFactory remoteTaskRunnerFactory = getTestRemoteTaskRunnerFactory(provisioningSchedulerConfig);
    Assert.assertNull(remoteTaskRunnerFactory.build().getProvisioningStrategy());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ProvisioningSchedulerConfig createMockProvisioningSchedulerConfig(boolean doAutoscaleReturn) {
    ProvisioningSchedulerConfig provisioningSchedulerConfig = Mockito.mock(ProvisioningSchedulerConfig.class);
    Mockito.when(provisioningSchedulerConfig.isDoAutoscale()).thenReturn(doAutoscaleReturn);
    return provisioningSchedulerConfig;
}
```
</details>

---
#### Test Case ID #druid_Test_16_2
#### Test Case Name: `testBuildWithoutAutoScale`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\overlord\RemoteTaskRunnerFactoryTest.java`)
#### Mock Object Variable Name: `provisioningSchedulerConfig`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testBuildWithoutAutoScale() {
-    ProvisioningSchedulerConfig provisioningSchedulerConfig = Mockito.mock(ProvisioningSchedulerConfig.class);
-    Mockito.when(provisioningSchedulerConfig.isDoAutoscale()).thenReturn(false);
+    ProvisioningSchedulerConfig provisioningSchedulerConfig = createMockProvisioningSchedulerConfig(false);
     RemoteTaskRunnerFactory remoteTaskRunnerFactory = getTestRemoteTaskRunnerFactory(provisioningSchedulerConfig);
     Assert.assertTrue(remoteTaskRunnerFactory.build().getProvisioningStrategy() instanceof NoopProvisioningStrategy);
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testBuildWithoutAutoScale() {
    ProvisioningSchedulerConfig provisioningSchedulerConfig = Mockito.mock(ProvisioningSchedulerConfig.class);
    Mockito.when(provisioningSchedulerConfig.isDoAutoscale()).thenReturn(false);
    RemoteTaskRunnerFactory remoteTaskRunnerFactory = getTestRemoteTaskRunnerFactory(provisioningSchedulerConfig);
    Assert.assertTrue(remoteTaskRunnerFactory.build().getProvisioningStrategy() instanceof NoopProvisioningStrategy);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ProvisioningSchedulerConfig createMockProvisioningSchedulerConfig(boolean doAutoscaleReturn) {
    ProvisioningSchedulerConfig provisioningSchedulerConfig = Mockito.mock(ProvisioningSchedulerConfig.class);
    Mockito.when(provisioningSchedulerConfig.isDoAutoscale()).thenReturn(doAutoscaleReturn);
    return provisioningSchedulerConfig;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_17
- **Scope**: method level
- **Mocked Class**: `org.mockito.MockedStatic<org.apache.druid.utils.ConnectionUriUtils>`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
// === Declare in class scope ===
private MockedStatic<ConnectionUriUtils> utils;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    utils = Mockito.mockStatic(ConnectionUriUtils.class);
}

// === Replace local variable in test with ===
utils

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_17_1
#### Test Case Name: `testMySqlFallbackMySqlMaria2x`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\utils\ConnectionUriUtilsTest.java`)
#### Mock Object Variable Name: `utils`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testMySqlFallbackMySqlMaria2x() {
-    MockedStatic<ConnectionUriUtils> utils = Mockito.mockStatic(ConnectionUriUtils.class);
+    // removed local mock; replaced with global field `utils`
     utils.when(() -> ConnectionUriUtils.tryParseJdbcUriParameters(MYSQL_URI, false)).thenCallRealMethod();
     utils.when(() -> ConnectionUriUtils.tryParseMySqlConnectionUri(MYSQL_URI)).thenThrow(ClassNotFoundException.class);
     utils.when(() -> ConnectionUriUtils.tryParseMariaDb2xConnectionUri(MYSQL_URI)).thenCallRealMethod();
     Set<String> props = ConnectionUriUtils.tryParseJdbcUriParameters(MYSQL_URI, false);
     // this would be 9 if didn't fall back to mariadb
     Assert.assertEquals(4, props.size());
     utils.close();
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testMySqlFallbackMySqlMaria2x() {
    MockedStatic<ConnectionUriUtils> utils = Mockito.mockStatic(ConnectionUriUtils.class);
    utils.when(() -> ConnectionUriUtils.tryParseJdbcUriParameters(MYSQL_URI, false)).thenCallRealMethod();
    utils.when(() -> ConnectionUriUtils.tryParseMySqlConnectionUri(MYSQL_URI)).thenThrow(ClassNotFoundException.class);
    utils.when(() -> ConnectionUriUtils.tryParseMariaDb2xConnectionUri(MYSQL_URI)).thenCallRealMethod();
    Set<String> props = ConnectionUriUtils.tryParseJdbcUriParameters(MYSQL_URI, false);
    // this would be 9 if didn't fall back to mariadb
    Assert.assertEquals(4, props.size());
    utils.close();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private MockedStatic<ConnectionUriUtils> utils;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    utils = Mockito.mockStatic(ConnectionUriUtils.class);
}

// === Replace local variable in test with ===
utils

```
</details>

---
#### Test Case ID #druid_Test_17_2
#### Test Case Name: `testMariaFallbackMaria3x`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\utils\ConnectionUriUtilsTest.java`)
#### Mock Object Variable Name: `utils`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testMariaFallbackMaria3x() {
-    MockedStatic<ConnectionUriUtils> utils = Mockito.mockStatic(ConnectionUriUtils.class);
+    // removed local mock; replaced with global field `utils`
     utils.when(() -> ConnectionUriUtils.tryParseJdbcUriParameters(MARIA_URI, false)).thenCallRealMethod();
     utils.when(() -> ConnectionUriUtils.tryParseMariaDb2xConnectionUri(MARIA_URI)).thenThrow(ClassNotFoundException.class);
     utils.when(() -> ConnectionUriUtils.tryParseMariaDb3xConnectionUri(MARIA_URI)).thenCallRealMethod();
     try {
         Set<String> props = ConnectionUriUtils.tryParseJdbcUriParameters(MARIA_URI, false);
         // this would be 4 if didn't fall back to mariadb 3x
         Assert.assertEquals(8, props.size());
     } catch (RuntimeException e) {
         Assert.assertTrue(e.getMessage().contains("Failed to find MariaDB driver class"));
     }
     utils.close();
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testMariaFallbackMaria3x() {
    MockedStatic<ConnectionUriUtils> utils = Mockito.mockStatic(ConnectionUriUtils.class);
    utils.when(() -> ConnectionUriUtils.tryParseJdbcUriParameters(MARIA_URI, false)).thenCallRealMethod();
    utils.when(() -> ConnectionUriUtils.tryParseMariaDb2xConnectionUri(MARIA_URI)).thenThrow(ClassNotFoundException.class);
    utils.when(() -> ConnectionUriUtils.tryParseMariaDb3xConnectionUri(MARIA_URI)).thenCallRealMethod();
    try {
        Set<String> props = ConnectionUriUtils.tryParseJdbcUriParameters(MARIA_URI, false);
        // this would be 4 if didn't fall back to mariadb 3x
        Assert.assertEquals(8, props.size());
    } catch (RuntimeException e) {
        Assert.assertTrue(e.getMessage().contains("Failed to find MariaDB driver class"));
    }
    utils.close();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private MockedStatic<ConnectionUriUtils> utils;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    utils = Mockito.mockStatic(ConnectionUriUtils.class);
}

// === Replace local variable in test with ===
utils

```
</details>

---
#### Test Case ID #druid_Test_17_3
#### Test Case Name: `testMySqlFallbackMySqlNoDrivers`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\utils\ConnectionUriUtilsTest.java`)
#### Mock Object Variable Name: `utils`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testMySqlFallbackMySqlNoDrivers() {
-    MockedStatic<ConnectionUriUtils> utils = Mockito.mockStatic(ConnectionUriUtils.class);
+    // removed local mock; replaced with global field `utils`
     utils.when(() -> ConnectionUriUtils.tryParseJdbcUriParameters(MYSQL_URI, false)).thenCallRealMethod();
     utils.when(() -> ConnectionUriUtils.tryParseMySqlConnectionUri(MYSQL_URI)).thenThrow(ClassNotFoundException.class);
     utils.when(() -> ConnectionUriUtils.tryParseMariaDb2xConnectionUri(MYSQL_URI)).thenThrow(ClassNotFoundException.class);
     try {
         ConnectionUriUtils.tryParseJdbcUriParameters(MYSQL_URI, false);
     } catch (RuntimeException e) {
         Assert.assertTrue(e.getMessage().contains("Failed to find MySQL driver class"));
     }
     utils.close();
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testMySqlFallbackMySqlNoDrivers() {
    MockedStatic<ConnectionUriUtils> utils = Mockito.mockStatic(ConnectionUriUtils.class);
    utils.when(() -> ConnectionUriUtils.tryParseJdbcUriParameters(MYSQL_URI, false)).thenCallRealMethod();
    utils.when(() -> ConnectionUriUtils.tryParseMySqlConnectionUri(MYSQL_URI)).thenThrow(ClassNotFoundException.class);
    utils.when(() -> ConnectionUriUtils.tryParseMariaDb2xConnectionUri(MYSQL_URI)).thenThrow(ClassNotFoundException.class);
    try {
        ConnectionUriUtils.tryParseJdbcUriParameters(MYSQL_URI, false);
    } catch (RuntimeException e) {
        Assert.assertTrue(e.getMessage().contains("Failed to find MySQL driver class"));
    }
    utils.close();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private MockedStatic<ConnectionUriUtils> utils;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    utils = Mockito.mockStatic(ConnectionUriUtils.class);
}

// === Replace local variable in test with ===
utils

```
</details>

---
## Mock Clone Instance #druid_MCI_18
- **Scope**: method level
- **Mocked Class**: `javax.servlet.http.HttpServletRequest`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
// === Declare in class scope ===
private HttpServletRequest request;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    request = Mockito.mock(HttpServletRequest.class);
}

// === Replace local variable in test with ===
request;

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_18_1
#### Test Case Name: `testHandleQueryParseExceptionWithFilterDisabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `request`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleQueryParseExceptionWithFilterDisabled() throws Exception {
     String errorMessage = "test exception message";
     ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
-    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
+    // removed local mock; replaced with global field `request`
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
     ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
     IOException testException = new IOException(errorMessage);
     servlet.handleQueryParseException(request, response, mockMapper, testException, false);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
     Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertEquals(errorMessage, captor.getValue().getMessage());
     Assert.assertEquals(IOException.class.getName(), ((QueryException) captor.getValue()).getErrorClass());
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleQueryParseExceptionWithFilterDisabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertEquals(IOException.class.getName(), ((QueryException) captor.getValue()).getErrorClass());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private HttpServletRequest request;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    request = Mockito.mock(HttpServletRequest.class);
}

// === Replace local variable in test with ===
request;

```
</details>

---
#### Test Case ID #druid_Test_18_2
#### Test Case Name: `testHandleQueryParseExceptionWithFilterEnabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `request`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleQueryParseExceptionWithFilterEnabled() throws Exception {
     String errorMessage = "test exception message";
     ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
-    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
+    // removed local mock; replaced with global field `request`
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
     ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

         @Override
         public boolean isShowDetailedJettyErrors() {
             return true;
         }

         @Override
         public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
             return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
         }
     });
     IOException testException = new IOException(errorMessage);
     servlet.handleQueryParseException(request, response, mockMapper, testException, false);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
     Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertNull(captor.getValue().getMessage());
     Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
     Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleQueryParseExceptionWithFilterEnabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
        }
    });
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertNull(captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private HttpServletRequest request;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    request = Mockito.mock(HttpServletRequest.class);
}

// === Replace local variable in test with ===
request;

```
</details>

---
#### Test Case ID #druid_Test_18_3
#### Test Case Name: `testHandleQueryParseExceptionWithFilterEnabledButMessageMatchAllowedRegex`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `request`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleQueryParseExceptionWithFilterEnabledButMessageMatchAllowedRegex() throws Exception {
     String errorMessage = "test exception message";
     ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
-    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
+    // removed local mock; replaced with global field `request`
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
     ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

         @Override
         public boolean isShowDetailedJettyErrors() {
             return true;
         }

         @Override
         public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
             return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of("test .*"));
         }
     });
     IOException testException = new IOException(errorMessage);
-    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
+    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
     Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertEquals(errorMessage, captor.getValue().getMessage());
     Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
     Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleQueryParseExceptionWithFilterEnabledButMessageMatchAllowedRegex() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of("test .*"));
        }
    });
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private HttpServletRequest request;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    request = Mockito.mock(HttpServletRequest.class);
}

// === Replace local variable in test with ===
request;

```
</details>

---
## Mock Clone Instance #druid_MCI_19
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.indexing.common.actions.TaskActionClient`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
private static TaskActionClient createMockTaskActionClient() {
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    return taskActionClient;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_19_1
#### Test Case Name: `testSetupAndCleanupIsCalledWtihParameter`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\AbstractTaskTest.java`)
#### Mock Object Variable Name: `taskActionClient`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
-    TaskActionClient taskActionClient = mock(TaskActionClient.class);
-    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
+    TaskActionClient taskActionClient = createMockTaskActionClient();
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSetupAndCleanupIsCalledWtihParameter() throws Exception {
    // These tests apparently use Mockito.  Mockito is bad as we've seen it rewrite byte code and effectively cause
    // impact to other totally unrelated tests.  Mockito needs to be completely erradicated from the codebase.  This
    // comment is here to either cause me to do it in this commit or just for posterity so that it is clear that it
    // should happen in the future.
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn("1");
    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
    when(toolbox.getTaskExecutorNode()).thenReturn(node);
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(true);
    File folder = temporaryFolder.newFolder();
    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Nullable
        @Override
        public String setup(TaskToolbox toolbox) throws Exception {
            // create a reports file to test the taskLogPusher pushes task reports
            String result = super.setup(toolbox);
            File attemptDir = Paths.get(folder.getAbsolutePath(), "attempt", toolbox.getAttemptId()).toFile();
            File reportsDir = new File(attemptDir, "report.json");
            File statusDir = new File(attemptDir, "status.json");
            FileUtils.write(reportsDir, "foo", StandardCharsets.UTF_8);
            FileUtils.write(statusDir, "{}", StandardCharsets.UTF_8);
            return result;
        }
    };
    task.run(toolbox);
    // call it 3 times, once to update location in setup, then one for status and location in cleanup
    Mockito.verify(taskActionClient, times(3)).submit(any());
    verify(pusher, times(1)).pushTaskReports(eq("myID"), any());
    verify(pusher, times(1)).pushTaskStatus(eq("myID"), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static TaskActionClient createMockTaskActionClient() {
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    return taskActionClient;
}
```
</details>

---
#### Test Case ID #druid_Test_19_2
#### Test Case Name: `testWithNoEncapsulatedTask`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\AbstractTaskTest.java`)
#### Mock Object Variable Name: `taskActionClient`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
-    TaskActionClient taskActionClient = mock(TaskActionClient.class);
-    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
+    TaskActionClient taskActionClient = createMockTaskActionClient();
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testWithNoEncapsulatedTask() throws Exception {
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn("1");
    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
    when(toolbox.getTaskExecutorNode()).thenReturn(node);
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(false);
    File folder = temporaryFolder.newFolder();
    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Nullable
        @Override
        public String setup(TaskToolbox toolbox) throws Exception {
            // create a reports file to test the taskLogPusher pushes task reports
            String result = super.setup(toolbox);
            File attemptDir = Paths.get(folder.getAbsolutePath(), "attempt", toolbox.getAttemptId()).toFile();
            File reportsDir = new File(attemptDir, "report.json");
            FileUtils.write(reportsDir, "foo", StandardCharsets.UTF_8);
            return result;
        }
    };
    task.run(toolbox);
    // encapsulated task is set to false, should never get called
    Mockito.verify(taskActionClient, never()).submit(any());
    verify(pusher, never()).pushTaskReports(eq("myID"), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static TaskActionClient createMockTaskActionClient() {
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    return taskActionClient;
}
```
</details>

---
#### Test Case ID #druid_Test_19_3
#### Test Case Name: `testTaskFailureWithoutExceptionGetsReportedCorrectly`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\AbstractTaskTest.java`)
#### Mock Object Variable Name: `taskActionClient`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
-    TaskActionClient taskActionClient = mock(TaskActionClient.class);
-    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
+    TaskActionClient taskActionClient = createMockTaskActionClient();
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Override
        public TaskStatus runTask(TaskToolbox toolbox) {
            return TaskStatus.failure("myId", "failed");
        }
    };
    task.run(toolbox);
    UpdateStatusAction action = new UpdateStatusAction("failure");
    verify(taskActionClient).submit(eq(action));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testTaskFailureWithoutExceptionGetsReportedCorrectly() throws Exception {
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn("1");
    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
    when(toolbox.getTaskExecutorNode()).thenReturn(node);
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(true);
    File folder = temporaryFolder.newFolder();
    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Override
        public TaskStatus runTask(TaskToolbox toolbox) {
            return TaskStatus.failure("myId", "failed");
        }
    };
    task.run(toolbox);
    UpdateStatusAction action = new UpdateStatusAction("failure");
    verify(taskActionClient).submit(eq(action));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static TaskActionClient createMockTaskActionClient() {
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    return taskActionClient;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_20
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.rpc.indexing.OverlordClient`
- **Test Case Count**: 14
- **MO Count**: 14

### Reusable Method
```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_20_1
#### Test Case Name: `testCompactWithoutGranularitySpec`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testCompactWithoutGranularitySpec() {
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
     final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
     final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
     final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
     final String dataSource = DATA_SOURCE_PREFIX + 0;
     compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
     new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, null, null));
     doCompactSegments(compactSegments, compactionConfigs);
     ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
     Assert.assertEquals(Intervals.of("2017-01-09T12:00:00.000Z/2017-01-10T00:00:00.000Z"), taskPayload.getIoConfig().getInputSpec().getInterval());
     Assert.assertNull(taskPayload.getGranularitySpec().getSegmentGranularity());
     Assert.assertNull(taskPayload.getGranularitySpec().getQueryGranularity());
     Assert.assertNull(taskPayload.getGranularitySpec().isRollup());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCompactWithoutGranularitySpec() {
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertEquals(Intervals.of("2017-01-09T12:00:00.000Z/2017-01-10T00:00:00.000Z"), taskPayload.getIoConfig().getInputSpec().getInterval());
    Assert.assertNull(taskPayload.getGranularitySpec().getSegmentGranularity());
    Assert.assertNull(taskPayload.getGranularitySpec().getQueryGranularity());
    Assert.assertNull(taskPayload.getGranularitySpec().isRollup());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_2
#### Test Case Name: `testCompactWithNotNullIOConfig`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
@Test
public void testCompactWithNotNullIOConfig() {
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, new UserCompactionTaskIOConfig(true), null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertTrue(taskPayload.getIoConfig().isDropExisting());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCompactWithNotNullIOConfig() {
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, new UserCompactionTaskIOConfig(true), null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertTrue(taskPayload.getIoConfig().isDropExisting());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_3
#### Test Case Name: `testCompactWithNullIOConfig`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
@Test
public void testCompactWithNullIOConfig() {
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertEquals(BatchIOConfig.DEFAULT_DROP_EXISTING, taskPayload.getIoConfig().isDropExisting());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCompactWithNullIOConfig() {
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertEquals(BatchIOConfig.DEFAULT_DROP_EXISTING, taskPayload.getIoConfig().isDropExisting());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_4
#### Test Case Name: `testCompactWithGranularitySpec`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
@Test
public void testCompactWithGranularitySpec() {
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), new UserCompactionTaskGranularityConfig(Granularities.YEAR, null, null), null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    // All segments is compact at the same time since we changed the segment granularity to YEAR and all segment
    // are within the same year
    Assert.assertEquals(ClientCompactionIntervalSpec.fromSegments(datasourceToSegments.get(dataSource), Granularities.YEAR), taskPayload.getIoConfig().getInputSpec());
    ClientCompactionTaskGranularitySpec expectedGranularitySpec = new ClientCompactionTaskGranularitySpec(Granularities.YEAR, null, null);
    Assert.assertEquals(expectedGranularitySpec, taskPayload.getGranularitySpec());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCompactWithGranularitySpec() {
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), new UserCompactionTaskGranularityConfig(Granularities.YEAR, null, null), null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    // All segments is compact at the same time since we changed the segment granularity to YEAR and all segment
    // are within the same year
    Assert.assertEquals(ClientCompactionIntervalSpec.fromSegments(datasourceToSegments.get(dataSource), Granularities.YEAR), taskPayload.getIoConfig().getInputSpec());
    ClientCompactionTaskGranularitySpec expectedGranularitySpec = new ClientCompactionTaskGranularitySpec(Granularities.YEAR, null, null);
    Assert.assertEquals(expectedGranularitySpec, taskPayload.getGranularitySpec());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_5
#### Test Case Name: `testCompactWithDimensionSpec`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testCompactWithDimensionSpec() {
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
     final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
     final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
     final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
     final String dataSource = DATA_SOURCE_PREFIX + 0;
     compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
     new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, new UserCompactionTaskDimensionsConfig(DimensionsSpec.getDefaultSchemas(ImmutableList.of("bar", "foo"))), null, null, null, null));
     doCompactSegments(compactSegments, compactionConfigs);
     ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
     Assert.assertEquals(DimensionsSpec.getDefaultSchemas(ImmutableList.of("bar", "foo")), taskPayload.getDimensionsSpec().getDimensions());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCompactWithDimensionSpec() {
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, new UserCompactionTaskDimensionsConfig(DimensionsSpec.getDefaultSchemas(ImmutableList.of("bar", "foo"))), null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertEquals(DimensionsSpec.getDefaultSchemas(ImmutableList.of("bar", "foo")), taskPayload.getDimensionsSpec().getDimensions());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_6
#### Test Case Name: `testCompactWithoutDimensionSpec`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testCompactWithoutDimensionSpec() {
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
     final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
     final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
     final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
     final String dataSource = DATA_SOURCE_PREFIX + 0;
     compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
     new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, null, null));
     doCompactSegments(compactSegments, compactionConfigs);
     ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
     Assert.assertNull(taskPayload.getDimensionsSpec());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCompactWithoutDimensionSpec() {
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertNull(taskPayload.getDimensionsSpec());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_7
#### Test Case Name: `testCompactWithRollupInGranularitySpec`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
@Test
public void testCompactWithRollupInGranularitySpec() {
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), new UserCompactionTaskGranularityConfig(Granularities.YEAR, null, true), null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    // All segments is compact at the same time since we changed the segment granularity to YEAR and all segment
    // are within the same year
    Assert.assertEquals(ClientCompactionIntervalSpec.fromSegments(datasourceToSegments.get(dataSource), Granularities.YEAR), taskPayload.getIoConfig().getInputSpec());
    ClientCompactionTaskGranularitySpec expectedGranularitySpec = new ClientCompactionTaskGranularitySpec(Granularities.YEAR, null, true);
    Assert.assertEquals(expectedGranularitySpec, taskPayload.getGranularitySpec());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCompactWithRollupInGranularitySpec() {
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), new UserCompactionTaskGranularityConfig(Granularities.YEAR, null, true), null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    // All segments is compact at the same time since we changed the segment granularity to YEAR and all segment
    // are within the same year
    Assert.assertEquals(ClientCompactionIntervalSpec.fromSegments(datasourceToSegments.get(dataSource), Granularities.YEAR), taskPayload.getIoConfig().getInputSpec());
    ClientCompactionTaskGranularitySpec expectedGranularitySpec = new ClientCompactionTaskGranularitySpec(Granularities.YEAR, null, true);
    Assert.assertEquals(expectedGranularitySpec, taskPayload.getGranularitySpec());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_8
#### Test Case Name: `testCompactWithTransformSpec`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testCompactWithTransformSpec() {
     NullHandling.initializeForTests();
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
     final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
     final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
     final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
     final String dataSource = DATA_SOURCE_PREFIX + 0;
     compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
     new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, new UserCompactionTaskTransformConfig(new SelectorDimFilter("dim1", "foo", null)), null, null));
     doCompactSegments(compactSegments, compactionConfigs);
     ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
     Assert.assertNotNull(taskPayload.getTransformSpec());
     Assert.assertEquals(new SelectorDimFilter("dim1", "foo", null), taskPayload.getTransformSpec().getFilter());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCompactWithTransformSpec() {
    NullHandling.initializeForTests();
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, new UserCompactionTaskTransformConfig(new SelectorDimFilter("dim1", "foo", null)), null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertNotNull(taskPayload.getTransformSpec());
    Assert.assertEquals(new SelectorDimFilter("dim1", "foo", null), taskPayload.getTransformSpec().getFilter());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_9
#### Test Case Name: `testCompactWithoutCustomSpecs`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
@Test
public void testCompactWithoutCustomSpecs() {
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertNull(taskPayload.getTransformSpec());
    Assert.assertNull(taskPayload.getMetricsSpec());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCompactWithoutCustomSpecs() {
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertNull(taskPayload.getTransformSpec());
    Assert.assertNull(taskPayload.getMetricsSpec());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_10
#### Test Case Name: `testCompactWithMetricsSpec`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testCompactWithMetricsSpec() {
     NullHandling.initializeForTests();
     AggregatorFactory[] aggregatorFactories = new AggregatorFactory[] { new CountAggregatorFactory("cnt") };
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
     final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
     final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
     final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
     final String dataSource = DATA_SOURCE_PREFIX + 0;
     compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
     new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, aggregatorFactories, null, null, null));
     doCompactSegments(compactSegments, compactionConfigs);
     ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
     AggregatorFactory[] actual = taskPayload.getMetricsSpec();
     Assert.assertNotNull(actual);
     Assert.assertArrayEquals(aggregatorFactories, actual);
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCompactWithMetricsSpec() {
    NullHandling.initializeForTests();
    AggregatorFactory[] aggregatorFactories = new AggregatorFactory[] { new CountAggregatorFactory("cnt") };
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, aggregatorFactories, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    AggregatorFactory[] actual = taskPayload.getMetricsSpec();
    Assert.assertNotNull(actual);
    Assert.assertArrayEquals(aggregatorFactories, actual);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_11
#### Test Case Name: `testDetermineSegmentGranularityFromSegmentsToCompact`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
     dataSources = DataSourcesSnapshot.fromUsedSegments(segments, ImmutableMap.of());
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
     final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
     final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
     final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testDetermineSegmentGranularityFromSegmentsToCompact() {
    String dataSourceName = DATA_SOURCE_PREFIX + 1;
    List<DataSegment> segments = new ArrayList<>();
    segments.add(new DataSegment(dataSourceName, Intervals.of("2017-01-01T00:00:00/2017-01-02T00:00:00"), "1", null, ImmutableList.of(), ImmutableList.of(), shardSpecFactory.apply(0, 2), 0, 10L));
    segments.add(new DataSegment(dataSourceName, Intervals.of("2017-01-01T00:00:00/2017-01-02T00:00:00"), "1", null, ImmutableList.of(), ImmutableList.of(), shardSpecFactory.apply(1, 2), 0, 10L));
    dataSources = DataSourcesSnapshot.fromUsedSegments(segments, ImmutableMap.of());
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    compactionConfigs.add(new DataSourceCompactionConfig(dataSourceName, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertEquals(ClientCompactionIntervalSpec.fromSegments(segments, Granularities.DAY), taskPayload.getIoConfig().getInputSpec());
    ClientCompactionTaskGranularitySpec expectedGranularitySpec = new ClientCompactionTaskGranularitySpec(Granularities.DAY, null, null);
    Assert.assertEquals(expectedGranularitySpec, taskPayload.getGranularitySpec());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_12
#### Test Case Name: `testDetermineSegmentGranularityFromSegmentGranularityInCompactionConfig`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
     dataSources = DataSourcesSnapshot.fromUsedSegments(segments, ImmutableMap.of());
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
     final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
     final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
     final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
     compactionConfigs.add(new DataSourceCompactionConfig(dataSourceName, 0, 500L, null, // smaller than segment interval
     new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), new UserCompactionTaskGranularityConfig(Granularities.YEAR, null, null), null, null, null, null, null));
     doCompactSegments(compactSegments, compactionConfigs);
     ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
     Assert.assertEquals(ClientCompactionIntervalSpec.fromSegments(segments, Granularities.YEAR), taskPayload.getIoConfig().getInputSpec());
     ClientCompactionTaskGranularitySpec expectedGranularitySpec = new ClientCompactionTaskGranularitySpec(Granularities.YEAR, null, null);
     Assert.assertEquals(expectedGranularitySpec, taskPayload.getGranularitySpec());
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testDetermineSegmentGranularityFromSegmentGranularityInCompactionConfig() {
    String dataSourceName = DATA_SOURCE_PREFIX + 1;
    List<DataSegment> segments = new ArrayList<>();
    segments.add(new DataSegment(dataSourceName, Intervals.of("2017-01-01T00:00:00/2017-01-02T00:00:00"), "1", null, ImmutableList.of(), ImmutableList.of(), shardSpecFactory.apply(0, 2), 0, 10L));
    segments.add(new DataSegment(dataSourceName, Intervals.of("2017-01-01T00:00:00/2017-01-02T00:00:00"), "1", null, ImmutableList.of(), ImmutableList.of(), shardSpecFactory.apply(1, 2), 0, 10L));
    dataSources = DataSourcesSnapshot.fromUsedSegments(segments, ImmutableMap.of());
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    compactionConfigs.add(new DataSourceCompactionConfig(dataSourceName, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), new UserCompactionTaskGranularityConfig(Granularities.YEAR, null, null), null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertEquals(ClientCompactionIntervalSpec.fromSegments(segments, Granularities.YEAR), taskPayload.getIoConfig().getInputSpec());
    ClientCompactionTaskGranularitySpec expectedGranularitySpec = new ClientCompactionTaskGranularitySpec(Granularities.YEAR, null, null);
    Assert.assertEquals(expectedGranularitySpec, taskPayload.getGranularitySpec());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_13
#### Test Case Name: `testCompactWithMetricsSpecShouldSetPreserveExistingMetricsTrue`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testCompactWithMetricsSpecShouldSetPreserveExistingMetricsTrue() {
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
     final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
     final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
     final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCompactWithMetricsSpecShouldSetPreserveExistingMetricsTrue() {
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, new AggregatorFactory[] { new CountAggregatorFactory("cnt") }, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertNotNull(taskPayload.getTuningConfig());
    Assert.assertNotNull(taskPayload.getTuningConfig().getAppendableIndexSpec());
    Assert.assertTrue(((OnheapIncrementalIndex.Spec) taskPayload.getTuningConfig().getAppendableIndexSpec()).isPreserveExistingMetrics());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
#### Test Case ID #druid_Test_20_14
#### Test Case Name: `testCompactWithoutMetricsSpecShouldSetPreserveExistingMetricsFalse`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `mockClient`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testCompactWithoutMetricsSpecShouldSetPreserveExistingMetricsFalse() {
-    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
+    // removed local mock; replaced with global field `mockClient`
     final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
     final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
     final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
     final String dataSource = DATA_SOURCE_PREFIX + 0;
     compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
     new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, null, null));
     doCompactSegments(compactSegments, compactionConfigs);
     ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
     Assert.assertNotNull(taskPayload.getTuningConfig());
     Assert.assertNotNull(taskPayload.getTuningConfig().getAppendableIndexSpec());
     Assert.assertFalse(((OnheapIncrementalIndex.Spec) taskPayload.getTuningConfig().getAppendableIndexSpec()).isPreserveExistingMetrics());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCompactWithoutMetricsSpecShouldSetPreserveExistingMetricsFalse() {
    final OverlordClient mockClient = Mockito.mock(OverlordClient.class);
    final ArgumentCaptor<Object> payloadCaptor = setUpMockClient(mockClient);
    final CompactSegments compactSegments = new CompactSegments(SEARCH_POLICY, mockClient);
    final List<DataSourceCompactionConfig> compactionConfigs = new ArrayList<>();
    final String dataSource = DATA_SOURCE_PREFIX + 0;
    compactionConfigs.add(new DataSourceCompactionConfig(dataSource, 0, 500L, null, // smaller than segment interval
    new Period("PT0H"), new UserCompactionTaskQueryTuningConfig(null, null, null, null, null, partitionsSpec, null, null, null, null, null, 3, null, null, null, null, null, null, null), null, null, null, null, null, null));
    doCompactSegments(compactSegments, compactionConfigs);
    ClientCompactionTaskQuery taskPayload = (ClientCompactionTaskQuery) payloadCaptor.getValue();
    Assert.assertNotNull(taskPayload.getTuningConfig());
    Assert.assertNotNull(taskPayload.getTuningConfig().getAppendableIndexSpec());
    Assert.assertFalse(((OnheapIncrementalIndex.Spec) taskPayload.getTuningConfig().getAppendableIndexSpec()).isPreserveExistingMetrics());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private OverlordClient mockClient;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockClient = Mockito.mock(OverlordClient.class);
}

// === Replace local variable in test with ===
mockClient

```
</details>

---
## Mock Clone Instance #druid_MCI_21
- **Scope**: method level
- **Mocked Class**: `org.apache.datasketches.hll.Union`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
private static Union createMockUnion(double estimateReturn) {
    Union union = mock(Union.class);
    Mockito.when(union.getEstimate()).thenReturn(estimateReturn);
    return union;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_21_1
#### Test Case Name: `testSupervisorDetermineNegativeNumShardsFromCardinalityReport`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\batch\parallel\DimensionCardinalityReportTest.java`)
#### Mock Object Variable Name: `negativeUnion`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    logger.clearLogEvents();
-    Union negativeUnion = mock(Union.class);
-    Mockito.when(negativeUnion.getEstimate()).thenReturn(-1.0);
+    Union negativeUnion = createMockUnion(-1.0);
    Interval interval = Intervals.of("2001-01-01/P1D");
    Map<Interval, Union> intervalToUnion = ImmutableMap.of(interval, negativeUnion);
    Map<Interval, Integer> intervalToNumShards = ParallelIndexSupervisorTask.computeIntervalToNumShards(10, intervalToUnion);
    Assert.assertEquals(new Integer(7), intervalToNumShards.get(interval));
    List<LogEvent> loggingEvents = logger.getLogEvents();
    String expectedLogMessage = "Estimated cardinality for union of estimates is zero or less: -1.00, setting num shards to 7";
    Assert.assertTrue("Logging events: " + loggingEvents, loggingEvents.stream().anyMatch(l -> l.getLevel().equals(Level.WARN) && l.getMessage().getFormattedMessage().equals(expectedLogMessage)));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSupervisorDetermineNegativeNumShardsFromCardinalityReport() {
    logger.clearLogEvents();
    Union negativeUnion = mock(Union.class);
    Mockito.when(negativeUnion.getEstimate()).thenReturn(-1.0);
    Interval interval = Intervals.of("2001-01-01/P1D");
    Map<Interval, Union> intervalToUnion = ImmutableMap.of(interval, negativeUnion);
    Map<Interval, Integer> intervalToNumShards = ParallelIndexSupervisorTask.computeIntervalToNumShards(10, intervalToUnion);
    Assert.assertEquals(new Integer(7), intervalToNumShards.get(interval));
    List<LogEvent> loggingEvents = logger.getLogEvents();
    String expectedLogMessage = "Estimated cardinality for union of estimates is zero or less: -1.00, setting num shards to 7";
    Assert.assertTrue("Logging events: " + loggingEvents, loggingEvents.stream().anyMatch(l -> l.getLevel().equals(Level.WARN) && l.getMessage().getFormattedMessage().equals(expectedLogMessage)));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static Union createMockUnion(double estimateReturn) {
    Union union = mock(Union.class);
    Mockito.when(union.getEstimate()).thenReturn(estimateReturn);
    return union;
}
```
</details>

---
#### Test Case ID #druid_Test_21_2
#### Test Case Name: `testSupervisorDeterminePositiveNumShardsFromCardinalityReport`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\batch\parallel\DimensionCardinalityReportTest.java`)
#### Mock Object Variable Name: `union`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testSupervisorDeterminePositiveNumShardsFromCardinalityReport() {
-    Union union = mock(Union.class);
-    Mockito.when(union.getEstimate()).thenReturn(24.0);
+    Union union = createMockUnion(24.0);
    Interval interval = Intervals.of("2001-01-01/P1D");
    Map<Interval, Union> intervalToUnion = ImmutableMap.of(interval, union);
    Map<Interval, Integer> intervalToNumShards = ParallelIndexSupervisorTask.computeIntervalToNumShards(6, intervalToUnion);
    Assert.assertEquals(new Integer(4), intervalToNumShards.get(interval));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSupervisorDeterminePositiveNumShardsFromCardinalityReport() {
    Union union = mock(Union.class);
    Mockito.when(union.getEstimate()).thenReturn(24.0);
    Interval interval = Intervals.of("2001-01-01/P1D");
    Map<Interval, Union> intervalToUnion = ImmutableMap.of(interval, union);
    Map<Interval, Integer> intervalToNumShards = ParallelIndexSupervisorTask.computeIntervalToNumShards(6, intervalToUnion);
    Assert.assertEquals(new Integer(4), intervalToNumShards.get(interval));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static Union createMockUnion(double estimateReturn) {
    Union union = mock(Union.class);
    Mockito.when(union.getEstimate()).thenReturn(estimateReturn);
    return union;
}
```
</details>

---
#### Test Case ID #druid_Test_21_3
#### Test Case Name: `testSupervisorDeterminePositiveOneShardFromCardinalityReport`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\batch\parallel\DimensionCardinalityReportTest.java`)
#### Mock Object Variable Name: `union`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
     logger.clearLogEvents();
-    Union union = mock(Union.class);
-    Mockito.when(union.getEstimate()).thenReturn(24.0);
+    Union union = createMockUnion(24.0);
     Interval interval = Intervals.of("2001-01-01/P1D");
     Map<Interval, Union> intervalToUnion = ImmutableMap.of(interval, union);
     Map<Interval, Integer> intervalToNumShards = ParallelIndexSupervisorTask.computeIntervalToNumShards(24, intervalToUnion);
     Assert.assertEquals(new Integer(1), intervalToNumShards.get(interval));
     List<LogEvent> loggingEvents = logger.getLogEvents();
     String expectedLogMessage = "estimatedNumShards is ONE (1) given estimated cardinality 24.00 and maxRowsPerSegment 24";
     Assert.assertTrue("Logging events: " + loggingEvents, loggingEvents.stream().anyMatch(l -> l.getLevel().equals(Level.INFO) && l.getMessage().getFormattedMessage().equals(expectedLogMessage)));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSupervisorDeterminePositiveOneShardFromCardinalityReport() {
    logger.clearLogEvents();
    Union union = mock(Union.class);
    Mockito.when(union.getEstimate()).thenReturn(24.0);
    Interval interval = Intervals.of("2001-01-01/P1D");
    Map<Interval, Union> intervalToUnion = ImmutableMap.of(interval, union);
    Map<Interval, Integer> intervalToNumShards = ParallelIndexSupervisorTask.computeIntervalToNumShards(24, intervalToUnion);
    Assert.assertEquals(new Integer(1), intervalToNumShards.get(interval));
    List<LogEvent> loggingEvents = logger.getLogEvents();
    String expectedLogMessage = "estimatedNumShards is ONE (1) given estimated cardinality 24.00 and maxRowsPerSegment 24";
    Assert.assertTrue("Logging events: " + loggingEvents, loggingEvents.stream().anyMatch(l -> l.getLevel().equals(Level.INFO) && l.getMessage().getFormattedMessage().equals(expectedLogMessage)));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static Union createMockUnion(double estimateReturn) {
    Union union = mock(Union.class);
    Mockito.when(union.getEstimate()).thenReturn(estimateReturn);
    return union;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_22
- **Scope**: method level
- **Mocked Class**: `javax.servlet.ServletOutputStream`
- **Test Case Count**: 6
- **MO Count**: 6

### Reusable Method
```java
// === Declare in class scope ===
private ServletOutputStream outputStream;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    outputStream = Mockito.mock(ServletOutputStream.class);
}

// === Replace local variable in test with ===
outputStream

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_22_1
#### Test Case Name: `testHandleExceptionWithFilterDisabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `outputStream`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleExceptionWithFilterDisabled() throws Exception {
     String errorMessage = "test exception message";
     ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
-    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
+    // removed local mock; replaced with global field `outputStream`
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
     Exception testException = new IllegalStateException(errorMessage);
     servlet.handleException(response, mockMapper, testException);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
-    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
+    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertEquals(errorMessage, captor.getValue().getMessage());
     Assert.assertEquals(IllegalStateException.class.getName(), ((QueryException) captor.getValue()).getErrorClass());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleExceptionWithFilterDisabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
    Exception testException = new IllegalStateException(errorMessage);
    servlet.handleException(response, mockMapper, testException);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertEquals(IllegalStateException.class.getName(), ((QueryException) captor.getValue()).getErrorClass());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ServletOutputStream outputStream;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    outputStream = Mockito.mock(ServletOutputStream.class);
}

// === Replace local variable in test with ===
outputStream

```
</details>

---
#### Test Case ID #druid_Test_22_2
#### Test Case Name: `testHandleExceptionWithFilterEnabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `outputStream`
<summary>Suggested Diff</summary>

```diff
@@
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
-    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
+    // removed local mock; replaced with global field `outputStream`
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {
@@
    servlet.handleException(response, mockMapper, testException);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
-    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
+    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleExceptionWithFilterEnabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
        }
    });
    Exception testException = new IllegalStateException(errorMessage);
    servlet.handleException(response, mockMapper, testException);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertNull(captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ServletOutputStream outputStream;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    outputStream = Mockito.mock(ServletOutputStream.class);
}

// === Replace local variable in test with ===
outputStream

```
</details>

---
#### Test Case ID #druid_Test_22_3
#### Test Case Name: `testHandleExceptionWithFilterEnabledButMessageMatchAllowedRegex`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `outputStream`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleExceptionWithFilterEnabledButMessageMatchAllowedRegex() throws Exception {
     String errorMessage = "test exception message";
     ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
-    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
+    // removed local mock; replaced with global field `outputStream`
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

         @Override
         public boolean isShowDetailedJettyErrors() {
             return true;
         }

         @Override
         public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
             return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of("test .*"));
         }
     });
     Exception testException = new IllegalStateException(errorMessage);
     servlet.handleException(response, mockMapper, testException);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
-    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
+    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertEquals(errorMessage, captor.getValue().getMessage());
     Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
     Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleExceptionWithFilterEnabledButMessageMatchAllowedRegex() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of("test .*"));
        }
    });
    Exception testException = new IllegalStateException(errorMessage);
    servlet.handleException(response, mockMapper, testException);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ServletOutputStream outputStream;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    outputStream = Mockito.mock(ServletOutputStream.class);
}

// === Replace local variable in test with ===
outputStream

```
</details>

---
#### Test Case ID #druid_Test_22_4
#### Test Case Name: `testHandleQueryParseExceptionWithFilterDisabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `outputStream`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleQueryParseExceptionWithFilterDisabled() throws Exception {
     String errorMessage = "test exception message";
     ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
     HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
-    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
+    // removed local mock; replaced with global field `outputStream`
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
     IOException testException = new IOException(errorMessage);
     servlet.handleQueryParseException(request, response, mockMapper, testException, false);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
     Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertEquals(errorMessage, captor.getValue().getMessage());
     Assert.assertEquals(IOException.class.getName(), ((QueryException) captor.getValue()).getErrorClass());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleQueryParseExceptionWithFilterDisabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertEquals(IOException.class.getName(), ((QueryException) captor.getValue()).getErrorClass());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ServletOutputStream outputStream;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    outputStream = Mockito.mock(ServletOutputStream.class);
}

// === Replace local variable in test with ===
outputStream

```
</details>

---
#### Test Case ID #druid_Test_22_5
#### Test Case Name: `testHandleQueryParseExceptionWithFilterEnabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `outputStream`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleQueryParseExceptionWithFilterEnabled() throws Exception {
     String errorMessage = "test exception message";
     ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
     HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
-    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
+    // removed local mock; replaced with global field `outputStream`
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

         @Override
         public boolean isShowDetailedJettyErrors() {
             return true;
         }

         @Override
         public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
             return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
         }
     });
     IOException testException = new IOException(errorMessage);
     servlet.handleQueryParseException(request, response, mockMapper, testException, false);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
-    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
+    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertNull(captor.getValue().getMessage());
     Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
     Assert.assertNull(((QueryException) captor.getValue()).getHost());
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleQueryParseExceptionWithFilterEnabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
        }
    });
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertNull(captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ServletOutputStream outputStream;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    outputStream = Mockito.mock(ServletOutputStream.class);
}

// === Replace local variable in test with ===
outputStream

```
</details>

---
#### Test Case ID #druid_Test_22_6
#### Test Case Name: `testHandleQueryParseExceptionWithFilterEnabledButMessageMatchAllowedRegex`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `outputStream`
<summary>Suggested Diff</summary>

```diff
@@
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
-    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
+    // removed local mock; replaced with global field `outputStream`
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {
@@
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
-    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
+    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleQueryParseExceptionWithFilterEnabledButMessageMatchAllowedRegex() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of("test .*"));
        }
    });
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ServletOutputStream outputStream;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    outputStream = Mockito.mock(ServletOutputStream.class);
}

// === Replace local variable in test with ===
outputStream

```
</details>

---
## Mock Clone Instance #druid_MCI_23
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.indexing.common.task.Task`
- **Test Case Count**: 1
- **MO Count**: 3

### Reusable Method
```java
private static Task createMockTask(TaskResource taskResource, String type) {
    Task mockTask = mock(Task.class);
    when(mockTask.getTaskResource()).thenReturn(taskResource);
    when(mockTask.getType()).thenReturn(type);
    return mockTask;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_23_1
#### Test Case Name: `test_canRunTask`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\overlord\ImmutableWorkerInfoTest.java`)
#### Mock Object Variable Name: `parallelIndexTask`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    TaskResource taskResource0 = mock(TaskResource.class);
    when(taskResource0.getRequiredCapacity()).thenReturn(3);
-    Task parallelIndexTask = mock(ParallelIndexSupervisorTask.class);
-    when(parallelIndexTask.getType()).thenReturn(ParallelIndexSupervisorTask.TYPE);
-    when(parallelIndexTask.getTaskResource()).thenReturn(taskResource0);
+    Task parallelIndexTask = createMockTask(taskResource0, ParallelIndexSupervisorTask.TYPE);
    // Since task satisifies parallel and total slot constraints, can run
    Assert.assertTrue(workerInfo.canRunTask(parallelIndexTask, 0.5));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void test_canRunTask() {
    ImmutableWorkerInfo workerInfo = new ImmutableWorkerInfo(new Worker("http", "testWorker2", "192.0.0.1", 10, "v1", WorkerConfig.DEFAULT_CATEGORY), 6, 0, ImmutableSet.of("grp1", "grp2"), ImmutableSet.of("task1", "task2"), DateTimes.of("2015-01-01T01:01:02Z"));
    // Parallel index task
    TaskResource taskResource0 = mock(TaskResource.class);
    when(taskResource0.getRequiredCapacity()).thenReturn(3);
    Task parallelIndexTask = mock(ParallelIndexSupervisorTask.class);
    when(parallelIndexTask.getType()).thenReturn(ParallelIndexSupervisorTask.TYPE);
    when(parallelIndexTask.getTaskResource()).thenReturn(taskResource0);
    // Since task satisifies parallel and total slot constraints, can run
    Assert.assertTrue(workerInfo.canRunTask(parallelIndexTask, 0.5));
    // Since task fails the parallel slot constraint, it cannot run (3 > 1)
    Assert.assertFalse(workerInfo.canRunTask(parallelIndexTask, 0.1));
    // Some other indexing task
    TaskResource taskResource1 = mock(TaskResource.class);
    when(taskResource1.getRequiredCapacity()).thenReturn(5);
    Task anyOtherTask = mock(IndexTask.class);
    when(anyOtherTask.getType()).thenReturn("index");
    when(anyOtherTask.getTaskResource()).thenReturn(taskResource1);
    // Not a parallel index task ->  satisfies parallel index constraint
    // But does not satisfy the total slot constraint and cannot run (11 > 10)
    Assert.assertFalse(workerInfo.canRunTask(anyOtherTask, 0.5));
    // Task has an availability conflict ("grp1")
    TaskResource taskResource2 = mock(TaskResource.class);
    when(taskResource2.getRequiredCapacity()).thenReturn(1);
    when(taskResource2.getAvailabilityGroup()).thenReturn("grp1");
    Task grp1Task = mock(IndexTask.class);
    when(grp1Task.getType()).thenReturn("blah");
    when(grp1Task.getTaskResource()).thenReturn(taskResource2);
    // Satisifies parallel index and total index slot constraints but cannot run due availability
    Assert.assertFalse(workerInfo.canRunTask(grp1Task, 0.3));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static Task createMockTask(TaskResource taskResource, String type) {
    Task mockTask = mock(Task.class);
    when(mockTask.getTaskResource()).thenReturn(taskResource);
    when(mockTask.getType()).thenReturn(type);
    return mockTask;
}
```
</details>

---
#### Test Case ID #druid_Test_23_2
#### Test Case Name: `test_canRunTask`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\overlord\ImmutableWorkerInfoTest.java`)
#### Mock Object Variable Name: `anyOtherTask`
<summary>Suggested Diff</summary>

```diff
@@
     TaskResource taskResource1 = mock(TaskResource.class);
     when(taskResource1.getRequiredCapacity()).thenReturn(5);
-    Task anyOtherTask = mock(IndexTask.class);
-    when(anyOtherTask.getType()).thenReturn("index");
-    when(anyOtherTask.getTaskResource()).thenReturn(taskResource1);
+    Task anyOtherTask = createMockTask(taskResource1, "index");
     // Not a parallel index task ->  satisfies parallel index constraint
     // But does not satisfy the total slot constraint and cannot run (11 > 10)
     Assert.assertFalse(workerInfo.canRunTask(anyOtherTask, 0.5));
     // Task has an availability conflict ("grp1")
     TaskResource taskResource2 = mock(TaskResource.class);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void test_canRunTask() {
    ImmutableWorkerInfo workerInfo = new ImmutableWorkerInfo(new Worker("http", "testWorker2", "192.0.0.1", 10, "v1", WorkerConfig.DEFAULT_CATEGORY), 6, 0, ImmutableSet.of("grp1", "grp2"), ImmutableSet.of("task1", "task2"), DateTimes.of("2015-01-01T01:01:02Z"));
    // Parallel index task
    TaskResource taskResource0 = mock(TaskResource.class);
    when(taskResource0.getRequiredCapacity()).thenReturn(3);
    Task parallelIndexTask = mock(ParallelIndexSupervisorTask.class);
    when(parallelIndexTask.getType()).thenReturn(ParallelIndexSupervisorTask.TYPE);
    when(parallelIndexTask.getTaskResource()).thenReturn(taskResource0);
    // Since task satisifies parallel and total slot constraints, can run
    Assert.assertTrue(workerInfo.canRunTask(parallelIndexTask, 0.5));
    // Since task fails the parallel slot constraint, it cannot run (3 > 1)
    Assert.assertFalse(workerInfo.canRunTask(parallelIndexTask, 0.1));
    // Some other indexing task
    TaskResource taskResource1 = mock(TaskResource.class);
    when(taskResource1.getRequiredCapacity()).thenReturn(5);
    Task anyOtherTask = mock(IndexTask.class);
    when(anyOtherTask.getType()).thenReturn("index");
    when(anyOtherTask.getTaskResource()).thenReturn(taskResource1);
    // Not a parallel index task ->  satisfies parallel index constraint
    // But does not satisfy the total slot constraint and cannot run (11 > 10)
    Assert.assertFalse(workerInfo.canRunTask(anyOtherTask, 0.5));
    // Task has an availability conflict ("grp1")
    TaskResource taskResource2 = mock(TaskResource.class);
    when(taskResource2.getRequiredCapacity()).thenReturn(1);
    when(taskResource2.getAvailabilityGroup()).thenReturn("grp1");
    Task grp1Task = mock(IndexTask.class);
    when(grp1Task.getType()).thenReturn("blah");
    when(grp1Task.getTaskResource()).thenReturn(taskResource2);
    // Satisifies parallel index and total index slot constraints but cannot run due availability
    Assert.assertFalse(workerInfo.canRunTask(grp1Task, 0.3));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static Task createMockTask(TaskResource taskResource, String type) {
    Task mockTask = mock(Task.class);
    when(mockTask.getTaskResource()).thenReturn(taskResource);
    when(mockTask.getType()).thenReturn(type);
    return mockTask;
}
```
</details>

---
#### Test Case ID #druid_Test_23_3
#### Test Case Name: `test_canRunTask`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\overlord\ImmutableWorkerInfoTest.java`)
#### Mock Object Variable Name: `grp1Task`
<summary>Suggested Diff</summary>

```diff
@@
     when(taskResource2.getRequiredCapacity()).thenReturn(1);
     when(taskResource2.getAvailabilityGroup()).thenReturn("grp1");
-    Task grp1Task = mock(IndexTask.class);
-    when(grp1Task.getType()).thenReturn("blah");
-    when(grp1Task.getTaskResource()).thenReturn(taskResource2);
+    Task grp1Task = createMockTask(taskResource2, "blah");
     // Satisifies parallel index and total index slot constraints but cannot run due availability
     Assert.assertFalse(workerInfo.canRunTask(grp1Task, 0.3));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void test_canRunTask() {
    ImmutableWorkerInfo workerInfo = new ImmutableWorkerInfo(new Worker("http", "testWorker2", "192.0.0.1", 10, "v1", WorkerConfig.DEFAULT_CATEGORY), 6, 0, ImmutableSet.of("grp1", "grp2"), ImmutableSet.of("task1", "task2"), DateTimes.of("2015-01-01T01:01:02Z"));
    // Parallel index task
    TaskResource taskResource0 = mock(TaskResource.class);
    when(taskResource0.getRequiredCapacity()).thenReturn(3);
    Task parallelIndexTask = mock(ParallelIndexSupervisorTask.class);
    when(parallelIndexTask.getType()).thenReturn(ParallelIndexSupervisorTask.TYPE);
    when(parallelIndexTask.getTaskResource()).thenReturn(taskResource0);
    // Since task satisifies parallel and total slot constraints, can run
    Assert.assertTrue(workerInfo.canRunTask(parallelIndexTask, 0.5));
    // Since task fails the parallel slot constraint, it cannot run (3 > 1)
    Assert.assertFalse(workerInfo.canRunTask(parallelIndexTask, 0.1));
    // Some other indexing task
    TaskResource taskResource1 = mock(TaskResource.class);
    when(taskResource1.getRequiredCapacity()).thenReturn(5);
    Task anyOtherTask = mock(IndexTask.class);
    when(anyOtherTask.getType()).thenReturn("index");
    when(anyOtherTask.getTaskResource()).thenReturn(taskResource1);
    // Not a parallel index task ->  satisfies parallel index constraint
    // But does not satisfy the total slot constraint and cannot run (11 > 10)
    Assert.assertFalse(workerInfo.canRunTask(anyOtherTask, 0.5));
    // Task has an availability conflict ("grp1")
    TaskResource taskResource2 = mock(TaskResource.class);
    when(taskResource2.getRequiredCapacity()).thenReturn(1);
    when(taskResource2.getAvailabilityGroup()).thenReturn("grp1");
    Task grp1Task = mock(IndexTask.class);
    when(grp1Task.getType()).thenReturn("blah");
    when(grp1Task.getTaskResource()).thenReturn(taskResource2);
    // Satisifies parallel index and total index slot constraints but cannot run due availability
    Assert.assertFalse(workerInfo.canRunTask(grp1Task, 0.3));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static Task createMockTask(TaskResource taskResource, String type) {
    Task mockTask = mock(Task.class);
    when(mockTask.getTaskResource()).thenReturn(taskResource);
    when(mockTask.getType()).thenReturn(type);
    return mockTask;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_24
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.indexing.overlord.TaskRunner`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
// === Declare in class scope ===
private TaskRunner runner;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    runner = mock(TaskRunner.class);
}

// === Replace local variable in test with ===
runner;

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_24_1
#### Test Case Name: `testActionCallsTaskRunner`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\actions\UpdateStatusActionTest.java`)
#### Mock Object Variable Name: `runner`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testActionCallsTaskRunner() {
     UpdateStatusAction action = new UpdateStatusAction("successful");
     Task task = NoopTask.create();
     TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
-    TaskRunner runner = mock(TaskRunner.class);
+    // removed local mock; replaced with global field `runner`
     when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
     action.perform(task, toolbox);
     verify(runner, times(1)).updateStatus(eq(task), eq(TaskStatus.success(task.getId())));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testActionCallsTaskRunner() {
    UpdateStatusAction action = new UpdateStatusAction("successful");
    Task task = NoopTask.create();
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    TaskRunner runner = mock(TaskRunner.class);
    when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
    action.perform(task, toolbox);
    verify(runner, times(1)).updateStatus(eq(task), eq(TaskStatus.success(task.getId())));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private TaskRunner runner;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    runner = mock(TaskRunner.class);
}

// === Replace local variable in test with ===
runner;

```
</details>

---
#### Test Case ID #druid_Test_24_2
#### Test Case Name: `testFailureScenario`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\actions\UpdateStatusActionTest.java`)
#### Mock Object Variable Name: `runner`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testFailureScenario() {
     UpdateStatusAction action = new UpdateStatusAction("failure");
     Task task = NoopTask.create();
     TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
-    TaskRunner runner = mock(TaskRunner.class);
+    // removed local mock; replaced with global field `runner`
     when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
     action.perform(task, toolbox);
     verify(runner, times(1)).updateStatus(eq(task), eq(TaskStatus.failure(task.getId(), "Error with task")));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testFailureScenario() {
    UpdateStatusAction action = new UpdateStatusAction("failure");
    Task task = NoopTask.create();
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    TaskRunner runner = mock(TaskRunner.class);
    when(toolbox.getTaskRunner()).thenReturn(Optional.of(runner));
    action.perform(task, toolbox);
    verify(runner, times(1)).updateStatus(eq(task), eq(TaskStatus.failure(task.getId(), "Error with task")));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private TaskRunner runner;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    runner = mock(TaskRunner.class);
}

// === Replace local variable in test with ===
runner;

```
</details>

---
#### Test Case ID #druid_Test_24_3
#### Test Case Name: `testNoTaskRunner`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\actions\UpdateStatusActionTest.java`)
#### Mock Object Variable Name: `runner`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testNoTaskRunner() {
     UpdateStatusAction action = new UpdateStatusAction("successful");
     Task task = NoopTask.create();
     TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
-    TaskRunner runner = mock(TaskRunner.class);
+    // removed local mock; replaced with global field `runner`
     when(toolbox.getTaskRunner()).thenReturn(Optional.absent());
     action.perform(task, toolbox);
-    verify(runner, never()).updateStatus(any(), any());
+    verify(runner, never()).updateStatus(any(), any());
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testNoTaskRunner() {
    UpdateStatusAction action = new UpdateStatusAction("successful");
    Task task = NoopTask.create();
    TaskActionToolbox toolbox = mock(TaskActionToolbox.class);
    TaskRunner runner = mock(TaskRunner.class);
    when(toolbox.getTaskRunner()).thenReturn(Optional.absent());
    action.perform(task, toolbox);
    verify(runner, never()).updateStatus(any(), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private TaskRunner runner;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    runner = mock(TaskRunner.class);
}

// === Replace local variable in test with ===
runner;

```
</details>

---
## Mock Clone Instance #druid_MCI_25
- **Scope**: method level
- **Mocked Class**: `java.util.concurrent.ExecutorService`
- **Test Case Count**: 5
- **MO Count**: 5

### Reusable Method
```java
// === Declare in class scope ===
private ExecutorService executor;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    executor = Mockito.mock(ExecutorService.class);
}

// === Replace local variable in test with ===
executor;

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_25_1
#### Test Case Name: `testFindMonitor`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\ClockDriftSafeMonitorSchedulerTest.java`)
#### Mock Object Variable Name: `executor`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testFindMonitor() {
     class Monitor1 extends NoopMonitor {
     }
     class Monitor2 extends NoopMonitor {
     }
     class Monitor3 extends NoopMonitor {
     }
     final Monitor1 monitor1 = new Monitor1();
     final Monitor2 monitor2 = new Monitor2();
-    ExecutorService executor = Mockito.mock(ExecutorService.class);
+    // removed local mock; replaced with global field `executor`
     final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(Mockito.mock(DruidMonitorSchedulerConfig.class), Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor1, monitor2), CronScheduler.newBuilder(Duration.ofSeconds(1L)).setThreadName("monitor-scheduler-test").build(), executor);
     final Optional<Monitor1> maybeFound1 = scheduler.findMonitor(Monitor1.class);
     final Optional<Monitor2> maybeFound2 = scheduler.findMonitor(Monitor2.class);
     Assert.assertTrue(maybeFound1.isPresent());
     Assert.assertTrue(maybeFound2.isPresent());
     Assert.assertSame(monitor1, maybeFound1.get());
     Assert.assertSame(monitor2, maybeFound2.get());
     Assert.assertFalse(scheduler.findMonitor(Monitor3.class).isPresent());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testFindMonitor() {
    class Monitor1 extends NoopMonitor {
    }
    class Monitor2 extends NoopMonitor {
    }
    class Monitor3 extends NoopMonitor {
    }
    final Monitor1 monitor1 = new Monitor1();
    final Monitor2 monitor2 = new Monitor2();
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(Mockito.mock(DruidMonitorSchedulerConfig.class), Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor1, monitor2), CronScheduler.newBuilder(Duration.ofSeconds(1L)).setThreadName("monitor-scheduler-test").build(), executor);
    final Optional<Monitor1> maybeFound1 = scheduler.findMonitor(Monitor1.class);
    final Optional<Monitor2> maybeFound2 = scheduler.findMonitor(Monitor2.class);
    Assert.assertTrue(maybeFound1.isPresent());
    Assert.assertTrue(maybeFound2.isPresent());
    Assert.assertSame(monitor1, maybeFound1.get());
    Assert.assertSame(monitor2, maybeFound2.get());
    Assert.assertFalse(scheduler.findMonitor(Monitor3.class).isPresent());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ExecutorService executor;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    executor = Mockito.mock(ExecutorService.class);
}

// === Replace local variable in test with ===
executor;

```
</details>

---
#### Test Case ID #druid_Test_25_2
#### Test Case Name: `testStart_RepeatScheduling`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\ClockDriftSafeMonitorSchedulerTest.java`)
#### Mock Object Variable Name: `executor`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testStart_RepeatScheduling() throws InterruptedException {
-    ExecutorService executor = Mockito.mock(ExecutorService.class);
+    // removed local mock; replaced with global field `executor`
     CountDownLatch latch = new CountDownLatch(1);
     Mockito.doAnswer(new Answer<Future<?>>() {

         private int scheduleCount = 0;

         @SuppressWarnings("unchecked")
         @Override
         public Future<?> answer(InvocationOnMock invocation) {
             final Object originalArgument = (invocation.getArguments())[3];
             CronTask task = ((CronTask) originalArgument);
             Mockito.doAnswer(new Answer<Future<?>>() {

                 @Override
                 public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                     final Object originalArgument = (invocation.getArguments())[0];
                     ((Callable<Boolean>) originalArgument).call();
                     return CompletableFuture.completedFuture(Boolean.TRUE);
                 }
             }).when(executor).submit(ArgumentMatchers.any(Callable.class));
             cronTaskRunner.submit(() -> {
                 while (scheduleCount < 2) {
                     scheduleCount++;
                     task.run(123L);
                 }
                 latch.countDown();
                 return null;
             });
             return createDummyFuture();
         }
     }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
     Monitor monitor = Mockito.mock(Monitor.class);
     DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
     Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
     final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
     scheduler.start();
     latch.await(5, TimeUnit.SECONDS);
     Mockito.verify(monitor, Mockito.times(1)).start();
     Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
-    Mockito.verify(executor, Mockito.times(2)).submit(ArgumentMatchers.any(Callable.class));
+    Mockito.verify(executor, Mockito.times(2)).submit(ArgumentMatchers.any(Callable.class));
     Mockito.verify(monitor, Mockito.times(2)).monitor(ArgumentMatchers.any());
     scheduler.stop();
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStart_RepeatScheduling() throws InterruptedException {
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    CountDownLatch latch = new CountDownLatch(1);
    Mockito.doAnswer(new Answer<Future<?>>() {

        private int scheduleCount = 0;

        @SuppressWarnings("unchecked")
        @Override
        public Future<?> answer(InvocationOnMock invocation) {
            final Object originalArgument = (invocation.getArguments())[3];
            CronTask task = ((CronTask) originalArgument);
            Mockito.doAnswer(new Answer<Future<?>>() {

                @Override
                public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                    final Object originalArgument = (invocation.getArguments())[0];
                    ((Callable<Boolean>) originalArgument).call();
                    return CompletableFuture.completedFuture(Boolean.TRUE);
                }
            }).when(executor).submit(ArgumentMatchers.any(Callable.class));
            cronTaskRunner.submit(() -> {
                while (scheduleCount < 2) {
                    scheduleCount++;
                    task.run(123L);
                }
                latch.countDown();
                return null;
            });
            return createDummyFuture();
        }
    }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Monitor monitor = Mockito.mock(Monitor.class);
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
    latch.await(5, TimeUnit.SECONDS);
    Mockito.verify(monitor, Mockito.times(1)).start();
    Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Mockito.verify(executor, Mockito.times(2)).submit(ArgumentMatchers.any(Callable.class));
    Mockito.verify(monitor, Mockito.times(2)).monitor(ArgumentMatchers.any());
    scheduler.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ExecutorService executor;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    executor = Mockito.mock(ExecutorService.class);
}

// === Replace local variable in test with ===
executor;

```
</details>

---
#### Test Case ID #druid_Test_25_3
#### Test Case Name: `testStart_RepeatAndStopScheduling`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\ClockDriftSafeMonitorSchedulerTest.java`)
#### Mock Object Variable Name: `executor`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testStart_RepeatAndStopScheduling() throws InterruptedException {
-    ExecutorService executor = Mockito.mock(ExecutorService.class);
+    // removed local mock; replaced with global field `executor`
     CountDownLatch latch = new CountDownLatch(1);
     Mockito.doAnswer(new Answer<Future<?>>() {

         private int scheduleCount = 0;

         @SuppressWarnings("unchecked")
         @Override
         public Future<?> answer(InvocationOnMock invocation) {
             final Object originalArgument = (invocation.getArguments())[3];
             CronTask task = ((CronTask) originalArgument);
             Mockito.doAnswer(new Answer<Future<?>>() {

                 @Override
                 public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                     final Object originalArgument = (invocation.getArguments())[0];
                     ((Callable<Boolean>) originalArgument).call();
                     return CompletableFuture.completedFuture(Boolean.FALSE);
                 }
             }).when(executor).submit(ArgumentMatchers.any(Callable.class));
             cronTaskRunner.submit(() -> {
                 while (scheduleCount < 2) {
                     scheduleCount++;
                     task.run(123L);
                 }
                 latch.countDown();
                 return null;
             });
             return createDummyFuture();
         }
     }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
     Monitor monitor = Mockito.mock(Monitor.class);
     DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
     Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
     final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
     scheduler.start();
     latch.await(5, TimeUnit.SECONDS);
     Mockito.verify(monitor, Mockito.times(1)).start();
     Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
-    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
+    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
     Mockito.verify(monitor, Mockito.times(2)).monitor(ArgumentMatchers.any());
     Mockito.verify(monitor, Mockito.times(1)).stop();
     scheduler.stop();
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStart_RepeatAndStopScheduling() throws InterruptedException {
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    CountDownLatch latch = new CountDownLatch(1);
    Mockito.doAnswer(new Answer<Future<?>>() {

        private int scheduleCount = 0;

        @SuppressWarnings("unchecked")
        @Override
        public Future<?> answer(InvocationOnMock invocation) {
            final Object originalArgument = (invocation.getArguments())[3];
            CronTask task = ((CronTask) originalArgument);
            Mockito.doAnswer(new Answer<Future<?>>() {

                @Override
                public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                    final Object originalArgument = (invocation.getArguments())[0];
                    ((Callable<Boolean>) originalArgument).call();
                    return CompletableFuture.completedFuture(Boolean.FALSE);
                }
            }).when(executor).submit(ArgumentMatchers.any(Callable.class));
            cronTaskRunner.submit(() -> {
                while (scheduleCount < 2) {
                    scheduleCount++;
                    task.run(123L);
                }
                latch.countDown();
                return null;
            });
            return createDummyFuture();
        }
    }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Monitor monitor = Mockito.mock(Monitor.class);
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
    latch.await(5, TimeUnit.SECONDS);
    Mockito.verify(monitor, Mockito.times(1)).start();
    Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
    Mockito.verify(monitor, Mockito.times(2)).monitor(ArgumentMatchers.any());
    Mockito.verify(monitor, Mockito.times(1)).stop();
    scheduler.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ExecutorService executor;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    executor = Mockito.mock(ExecutorService.class);
}

// === Replace local variable in test with ===
executor;

```
</details>

---
#### Test Case ID #druid_Test_25_4
#### Test Case Name: `testStart_UnexpectedExceptionWhileMonitoring`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\ClockDriftSafeMonitorSchedulerTest.java`)
#### Mock Object Variable Name: `executor`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testStart_UnexpectedExceptionWhileMonitoring() throws InterruptedException {
-    ExecutorService executor = Mockito.mock(ExecutorService.class);
+    // removed local mock; replaced with global field `executor`
     Monitor monitor = Mockito.mock(Monitor.class);
     Mockito.when(monitor.monitor(ArgumentMatchers.any(ServiceEmitter.class))).thenThrow(new RuntimeException("Test throwing exception while monitoring"));
     DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
     Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
     CountDownLatch latch = new CountDownLatch(1);
     AtomicBoolean monitorResultHolder = new AtomicBoolean(false);
     Mockito.doAnswer(new Answer<Future<?>>() {

         @SuppressWarnings("unchecked")
         @Override
         public Future<?> answer(InvocationOnMock invocation) {
             final Object originalArgument = (invocation.getArguments())[3];
             CronTask task = ((CronTask) originalArgument);
             Mockito.doAnswer(new Answer<Future<?>>() {

                 @Override
                 public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                     final Object originalArgument = (invocation.getArguments())[0];
                     final boolean continueMonitor = ((Callable<Boolean>) originalArgument).call();
                     monitorResultHolder.set(continueMonitor);
                     return CompletableFuture.completedFuture(continueMonitor);
                 }
             }).when(executor).submit(ArgumentMatchers.any(Callable.class));
             cronTaskRunner.submit(() -> {
                 task.run(123L);
                 latch.countDown();
                 return null;
             });
             return createDummyFuture();
         }
     }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
     final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
     scheduler.start();
     latch.await(5, TimeUnit.SECONDS);
     Mockito.verify(monitor, Mockito.times(1)).start();
     Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
-    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
+    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
     Mockito.verify(monitor, Mockito.times(1)).monitor(ArgumentMatchers.any());
     Assert.assertTrue(monitorResultHolder.get());
     scheduler.stop();
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStart_UnexpectedExceptionWhileMonitoring() throws InterruptedException {
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    Monitor monitor = Mockito.mock(Monitor.class);
    Mockito.when(monitor.monitor(ArgumentMatchers.any(ServiceEmitter.class))).thenThrow(new RuntimeException("Test throwing exception while monitoring"));
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
    CountDownLatch latch = new CountDownLatch(1);
    AtomicBoolean monitorResultHolder = new AtomicBoolean(false);
    Mockito.doAnswer(new Answer<Future<?>>() {

        @SuppressWarnings("unchecked")
        @Override
        public Future<?> answer(InvocationOnMock invocation) {
            final Object originalArgument = (invocation.getArguments())[3];
            CronTask task = ((CronTask) originalArgument);
            Mockito.doAnswer(new Answer<Future<?>>() {

                @Override
                public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                    final Object originalArgument = (invocation.getArguments())[0];
                    final boolean continueMonitor = ((Callable<Boolean>) originalArgument).call();
                    monitorResultHolder.set(continueMonitor);
                    return CompletableFuture.completedFuture(continueMonitor);
                }
            }).when(executor).submit(ArgumentMatchers.any(Callable.class));
            cronTaskRunner.submit(() -> {
                task.run(123L);
                latch.countDown();
                return null;
            });
            return createDummyFuture();
        }
    }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
    latch.await(5, TimeUnit.SECONDS);
    Mockito.verify(monitor, Mockito.times(1)).start();
    Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
    Mockito.verify(monitor, Mockito.times(1)).monitor(ArgumentMatchers.any());
    Assert.assertTrue(monitorResultHolder.get());
    scheduler.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ExecutorService executor;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    executor = Mockito.mock(ExecutorService.class);
}

// === Replace local variable in test with ===
executor;

```
</details>

---
#### Test Case ID #druid_Test_25_5
#### Test Case Name: `testStart_UnexpectedExceptionWhileScheduling`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\ClockDriftSafeMonitorSchedulerTest.java`)
#### Mock Object Variable Name: `executor`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testStart_UnexpectedExceptionWhileScheduling() throws InterruptedException {
-    ExecutorService executor = Mockito.mock(ExecutorService.class);
+    // removed local mock; replaced with global field `executor`
     Monitor monitor = Mockito.mock(Monitor.class);
     DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
     Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
     CountDownLatch latch = new CountDownLatch(1);
     Mockito.doAnswer(new Answer<Future<?>>() {

         @SuppressWarnings("unchecked")
         @Override
         public Future<?> answer(InvocationOnMock invocation) {
             final Object originalArgument = (invocation.getArguments())[3];
             CronTask task = ((CronTask) originalArgument);
-            Mockito.when(executor.submit(ArgumentMatchers.any(Callable.class))).thenThrow(new RuntimeException("Test throwing exception while scheduling"));
+            Mockito.when(executor.submit(ArgumentMatchers.any(Callable.class))).thenThrow(new RuntimeException("Test throwing exception while scheduling"));
             cronTaskRunner.submit(() -> {
                 task.run(123L);
                 latch.countDown();
                 return null;
             });
             return createDummyFuture();
         }
     }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
     final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
     scheduler.start();
     latch.await(5, TimeUnit.SECONDS);
     Mockito.verify(monitor, Mockito.times(1)).start();
     Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
-    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
+    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
     scheduler.stop();
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStart_UnexpectedExceptionWhileScheduling() throws InterruptedException {
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    Monitor monitor = Mockito.mock(Monitor.class);
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
    CountDownLatch latch = new CountDownLatch(1);
    Mockito.doAnswer(new Answer<Future<?>>() {

        @SuppressWarnings("unchecked")
        @Override
        public Future<?> answer(InvocationOnMock invocation) {
            final Object originalArgument = (invocation.getArguments())[3];
            CronTask task = ((CronTask) originalArgument);
            Mockito.when(executor.submit(ArgumentMatchers.any(Callable.class))).thenThrow(new RuntimeException("Test throwing exception while scheduling"));
            cronTaskRunner.submit(() -> {
                task.run(123L);
                latch.countDown();
                return null;
            });
            return createDummyFuture();
        }
    }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
    latch.await(5, TimeUnit.SECONDS);
    Mockito.verify(monitor, Mockito.times(1)).start();
    Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
    scheduler.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ExecutorService executor;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    executor = Mockito.mock(ExecutorService.class);
}

// === Replace local variable in test with ===
executor;

```
</details>

---
## Mock Clone Instance #druid_MCI_26
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.indexer.HadoopDruidIndexerConfig`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
private static HadoopDruidIndexerConfig createMockHadoopDruidIndexerConfig(Set<Interval> segmentGranularIntervals) {
    HadoopDruidIndexerConfig config = Mockito.mock(HadoopDruidIndexerConfig.class);
    Mockito.doNothing().when(config).setShardSpecs(Mockito.anyMap());
    Mockito.when(config.getSegmentGranularIntervals()).thenReturn(segmentGranularIntervals);
    Mockito.when(config.isDeterminingPartitions()).thenReturn(false);
    return config;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_26_1
#### Test Case Name: `testRunWithHashedPartitionsSpecCreateHashBasedNumberedShardSpecWithHashPartitionFunction`(File: `C:\Java_projects\Apache\druid\indexing-hadoop\src\test\java\org\apache\druid\indexer\HadoopDruidDetermineConfigurationJobTest.java`)
#### Mock Object Variable Name: `config`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final Set<Interval> intervals = ImmutableSet.of(Intervals.of("2020-01-01/P1D"), Intervals.of("2020-01-02/P1D"), Intervals.of("2020-01-03/P1D"));
    final HashedPartitionsSpec partitionsSpec = new HashedPartitionsSpec(null, 2, null, HashPartitionFunction.MURMUR3_32_ABS, null, null);
-    final HadoopDruidIndexerConfig config = Mockito.mock(HadoopDruidIndexerConfig.class);
-    Mockito.when(config.isDeterminingPartitions()).thenReturn(false);
-    Mockito.when(config.getPartitionsSpec()).thenReturn(partitionsSpec);
-    Mockito.when(config.getSegmentGranularIntervals()).thenReturn(intervals);
    final ArgumentCaptor<Map<Long, List<HadoopyShardSpec>>> resultCaptor = ArgumentCaptor.forClass(Map.class);
-    Mockito.doNothing().when(config).setShardSpecs(resultCaptor.capture());
+    final HadoopDruidIndexerConfig config = createMockHadoopDruidIndexerConfig(intervals);
+    Mockito.when(config.getPartitionsSpec()).thenReturn(partitionsSpec);
+    Mockito.doNothing().when(config).setShardSpecs(resultCaptor.capture());
    final HadoopDruidDetermineConfigurationJob job = new HadoopDruidDetermineConfigurationJob(config);
    Assert.assertTrue(job.run());
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testRunWithHashedPartitionsSpecCreateHashBasedNumberedShardSpecWithHashPartitionFunction() {
    final Set<Interval> intervals = ImmutableSet.of(Intervals.of("2020-01-01/P1D"), Intervals.of("2020-01-02/P1D"), Intervals.of("2020-01-03/P1D"));
    final HashedPartitionsSpec partitionsSpec = new HashedPartitionsSpec(null, 2, null, HashPartitionFunction.MURMUR3_32_ABS, null, null);
    final HadoopDruidIndexerConfig config = Mockito.mock(HadoopDruidIndexerConfig.class);
    Mockito.when(config.isDeterminingPartitions()).thenReturn(false);
    Mockito.when(config.getPartitionsSpec()).thenReturn(partitionsSpec);
    Mockito.when(config.getSegmentGranularIntervals()).thenReturn(intervals);
    final ArgumentCaptor<Map<Long, List<HadoopyShardSpec>>> resultCaptor = ArgumentCaptor.forClass(Map.class);
    Mockito.doNothing().when(config).setShardSpecs(resultCaptor.capture());
    final HadoopDruidDetermineConfigurationJob job = new HadoopDruidDetermineConfigurationJob(config);
    Assert.assertTrue(job.run());
    final Map<Long, List<HadoopyShardSpec>> shardSpecs = resultCaptor.getValue();
    Assert.assertEquals(3, shardSpecs.size());
    for (Interval interval : intervals) {
        final List<HadoopyShardSpec> shardSpecsPerInterval = shardSpecs.get(interval.getStartMillis());
        Assert.assertEquals(2, shardSpecsPerInterval.size());
        for (int i = 0; i < shardSpecsPerInterval.size(); i++) {
            Assert.assertEquals(new HashBasedNumberedShardSpec(i, shardSpecsPerInterval.size(), i, shardSpecsPerInterval.size(), null, HashPartitionFunction.MURMUR3_32_ABS, new ObjectMapper()), shardSpecsPerInterval.get(i).getActualSpec());
        }
    }
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HadoopDruidIndexerConfig createMockHadoopDruidIndexerConfig(Set<Interval> segmentGranularIntervals) {
    HadoopDruidIndexerConfig config = Mockito.mock(HadoopDruidIndexerConfig.class);
    Mockito.doNothing().when(config).setShardSpecs(Mockito.anyMap());
    Mockito.when(config.getSegmentGranularIntervals()).thenReturn(segmentGranularIntervals);
    Mockito.when(config.isDeterminingPartitions()).thenReturn(false);
    return config;
}
```
</details>

---
#### Test Case ID #druid_Test_26_2
#### Test Case Name: `testRunWithSingleDimensionPartitionsSpecCreateHashBasedNumberedShardSpecWithoutHashPartitionFunction`(File: `C:\Java_projects\Apache\druid\indexing-hadoop\src\test\java\org\apache\druid\indexer\HadoopDruidDetermineConfigurationJobTest.java`)
#### Mock Object Variable Name: `config`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final Set<Interval> intervals = ImmutableSet.of(Intervals.of("2020-01-01/P1D"), Intervals.of("2020-01-02/P1D"), Intervals.of("2020-01-03/P1D"));
    final SingleDimensionPartitionsSpec partitionsSpec = new SingleDimensionPartitionsSpec(1000, null, "dim", false);
-    final HadoopDruidIndexerConfig config = Mockito.mock(HadoopDruidIndexerConfig.class);
-    Mockito.when(config.isDeterminingPartitions()).thenReturn(false);
-    Mockito.when(config.getPartitionsSpec()).thenReturn(partitionsSpec);
-    Mockito.when(config.getSegmentGranularIntervals()).thenReturn(intervals);
+    final HadoopDruidIndexerConfig config = createMockHadoopDruidIndexerConfig(intervals);
+    Mockito.when(config.getPartitionsSpec()).thenReturn(partitionsSpec);
    final ArgumentCaptor<Map<Long, List<HadoopyShardSpec>>> resultCaptor = ArgumentCaptor.forClass(Map.class);
-    Mockito.doNothing().when(config).setShardSpecs(resultCaptor.capture());
+    Mockito.doNothing().when(config).setShardSpecs(resultCaptor.capture());
    final HadoopDruidDetermineConfigurationJob job = new HadoopDruidDetermineConfigurationJob(config);
    Assert.assertTrue(job.run());
    final Map<Long, List<HadoopyShardSpec>> shardSpecs = resultCaptor.getValue();
    Assert.assertEquals(3, shardSpecs.size());
    for (Interval interval : intervals) {
        final List<HadoopyShardSpec> shardSpecsPerInterval = shardSpecs.get(interval.getStartMillis());
        Assert.assertEquals(1, shardSpecsPerInterval.size());
        Assert.assertEquals(new HashBasedNumberedShardSpec(0, shardSpecsPerInterval.size(), 0, shardSpecsPerInterval.size(), ImmutableList.of("dim"), null, new ObjectMapper()), shardSpecsPerInterval.get(0).getActualSpec());
    }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testRunWithSingleDimensionPartitionsSpecCreateHashBasedNumberedShardSpecWithoutHashPartitionFunction() {
    final Set<Interval> intervals = ImmutableSet.of(Intervals.of("2020-01-01/P1D"), Intervals.of("2020-01-02/P1D"), Intervals.of("2020-01-03/P1D"));
    final SingleDimensionPartitionsSpec partitionsSpec = new SingleDimensionPartitionsSpec(1000, null, "dim", false);
    final HadoopDruidIndexerConfig config = Mockito.mock(HadoopDruidIndexerConfig.class);
    Mockito.when(config.isDeterminingPartitions()).thenReturn(false);
    Mockito.when(config.getPartitionsSpec()).thenReturn(partitionsSpec);
    Mockito.when(config.getSegmentGranularIntervals()).thenReturn(intervals);
    final ArgumentCaptor<Map<Long, List<HadoopyShardSpec>>> resultCaptor = ArgumentCaptor.forClass(Map.class);
    Mockito.doNothing().when(config).setShardSpecs(resultCaptor.capture());
    final HadoopDruidDetermineConfigurationJob job = new HadoopDruidDetermineConfigurationJob(config);
    Assert.assertTrue(job.run());
    final Map<Long, List<HadoopyShardSpec>> shardSpecs = resultCaptor.getValue();
    Assert.assertEquals(3, shardSpecs.size());
    for (Interval interval : intervals) {
        final List<HadoopyShardSpec> shardSpecsPerInterval = shardSpecs.get(interval.getStartMillis());
        Assert.assertEquals(1, shardSpecsPerInterval.size());
        Assert.assertEquals(new HashBasedNumberedShardSpec(0, shardSpecsPerInterval.size(), 0, shardSpecsPerInterval.size(), ImmutableList.of("dim"), null, new ObjectMapper()), shardSpecsPerInterval.get(0).getActualSpec());
    }
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HadoopDruidIndexerConfig createMockHadoopDruidIndexerConfig(Set<Interval> segmentGranularIntervals) {
    HadoopDruidIndexerConfig config = Mockito.mock(HadoopDruidIndexerConfig.class);
    Mockito.doNothing().when(config).setShardSpecs(Mockito.anyMap());
    Mockito.when(config.getSegmentGranularIntervals()).thenReturn(segmentGranularIntervals);
    Mockito.when(config.isDeterminingPartitions()).thenReturn(false);
    return config;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_27
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.server.initialization.ServerConfig`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
private static ServerConfig createMockServerConfig(AllowedRegexErrorResponseTransformStrategy errorResponseTransformStrategy) {
    ServerConfig serverConfig = Mockito.mock(ServerConfig.class);
    Mockito.when(serverConfig.getErrorResponseTransformStrategy()).thenReturn(errorResponseTransformStrategy);
    return serverConfig;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_27_1
#### Test Case Name: `testErrorHandlerSanitizesErrorAsExpected`(File: `C:\Java_projects\Apache\druid\sql\src\test\java\org\apache\druid\sql\avatica\ErrorHandlerTest.java`)
#### Mock Object Variable Name: `serverConfig`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testErrorHandlerSanitizesErrorAsExpected() {
-    ServerConfig serverConfig = Mockito.mock(ServerConfig.class);
    AllowedRegexErrorResponseTransformStrategy emptyAllowedRegexErrorResponseTransformStrategy = new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
-    Mockito.when(serverConfig.getErrorResponseTransformStrategy()).thenReturn(emptyAllowedRegexErrorResponseTransformStrategy);
+    ServerConfig serverConfig = createMockServerConfig(emptyAllowedRegexErrorResponseTransformStrategy);
    ErrorHandler errorHandler = new ErrorHandler(serverConfig);
    QueryException input = new QueryException("error", "error message", "error class", "host");
    RuntimeException output = errorHandler.sanitize(input);
    Assert.assertNull(output.getMessage());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testErrorHandlerSanitizesErrorAsExpected() {
    ServerConfig serverConfig = Mockito.mock(ServerConfig.class);
    AllowedRegexErrorResponseTransformStrategy emptyAllowedRegexErrorResponseTransformStrategy = new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
    Mockito.when(serverConfig.getErrorResponseTransformStrategy()).thenReturn(emptyAllowedRegexErrorResponseTransformStrategy);
    ErrorHandler errorHandler = new ErrorHandler(serverConfig);
    QueryException input = new QueryException("error", "error message", "error class", "host");
    RuntimeException output = errorHandler.sanitize(input);
    Assert.assertNull(output.getMessage());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ServerConfig createMockServerConfig(AllowedRegexErrorResponseTransformStrategy errorResponseTransformStrategy) {
    ServerConfig serverConfig = Mockito.mock(ServerConfig.class);
    Mockito.when(serverConfig.getErrorResponseTransformStrategy()).thenReturn(errorResponseTransformStrategy);
    return serverConfig;
}
```
</details>

---
#### Test Case ID #druid_Test_27_2
#### Test Case Name: `testErrorHandlerHasAffectingErrorResponseTransformStrategyReturnsTrueWhenNotUsingNoErrorResponseTransformStrategy`(File: `C:\Java_projects\Apache\druid\sql\src\test\java\org\apache\druid\sql\avatica\ErrorHandlerTest.java`)
#### Mock Object Variable Name: `serverConfig`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testErrorHandlerHasAffectingErrorResponseTransformStrategyReturnsTrueWhenNotUsingNoErrorResponseTransformStrategy() {
-    ServerConfig serverConfig = Mockito.mock(ServerConfig.class);
     AllowedRegexErrorResponseTransformStrategy emptyAllowedRegexErrorResponseTransformStrategy = new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
-    Mockito.when(serverConfig.getErrorResponseTransformStrategy()).thenReturn(emptyAllowedRegexErrorResponseTransformStrategy);
+    ServerConfig serverConfig = createMockServerConfig(emptyAllowedRegexErrorResponseTransformStrategy);
     ErrorHandler errorHandler = new ErrorHandler(serverConfig);
     Assert.assertTrue(errorHandler.hasAffectingErrorResponseTransformStrategy());
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testErrorHandlerHasAffectingErrorResponseTransformStrategyReturnsTrueWhenNotUsingNoErrorResponseTransformStrategy() {
    ServerConfig serverConfig = Mockito.mock(ServerConfig.class);
    AllowedRegexErrorResponseTransformStrategy emptyAllowedRegexErrorResponseTransformStrategy = new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
    Mockito.when(serverConfig.getErrorResponseTransformStrategy()).thenReturn(emptyAllowedRegexErrorResponseTransformStrategy);
    ErrorHandler errorHandler = new ErrorHandler(serverConfig);
    Assert.assertTrue(errorHandler.hasAffectingErrorResponseTransformStrategy());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ServerConfig createMockServerConfig(AllowedRegexErrorResponseTransformStrategy errorResponseTransformStrategy) {
    ServerConfig serverConfig = Mockito.mock(ServerConfig.class);
    Mockito.when(serverConfig.getErrorResponseTransformStrategy()).thenReturn(errorResponseTransformStrategy);
    return serverConfig;
}
```
</details>

---
#### Test Case ID #druid_Test_27_3
#### Test Case Name: `testErrorHandlerHandlesNonSanitizableExceptionCorrectly`(File: `C:\Java_projects\Apache\druid\sql\src\test\java\org\apache\druid\sql\avatica\ErrorHandlerTest.java`)
#### Mock Object Variable Name: `serverConfig`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testErrorHandlerHandlesNonSanitizableExceptionCorrectly() {
-    ServerConfig serverConfig = Mockito.mock(ServerConfig.class);
     AllowedRegexErrorResponseTransformStrategy emptyAllowedRegexErrorResponseTransformStrategy = new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
-    Mockito.when(serverConfig.getErrorResponseTransformStrategy()).thenReturn(emptyAllowedRegexErrorResponseTransformStrategy);
+    ServerConfig serverConfig = createMockServerConfig(emptyAllowedRegexErrorResponseTransformStrategy);
     ErrorHandler errorHandler = new ErrorHandler(serverConfig);
     Exception input = new Exception("message");
     RuntimeException output = errorHandler.sanitize(input);
     Assert.assertEquals(null, output.getMessage());
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testErrorHandlerHandlesNonSanitizableExceptionCorrectly() {
    ServerConfig serverConfig = Mockito.mock(ServerConfig.class);
    AllowedRegexErrorResponseTransformStrategy emptyAllowedRegexErrorResponseTransformStrategy = new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
    Mockito.when(serverConfig.getErrorResponseTransformStrategy()).thenReturn(emptyAllowedRegexErrorResponseTransformStrategy);
    ErrorHandler errorHandler = new ErrorHandler(serverConfig);
    Exception input = new Exception("message");
    RuntimeException output = errorHandler.sanitize(input);
    Assert.assertEquals(null, output.getMessage());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ServerConfig createMockServerConfig(AllowedRegexErrorResponseTransformStrategy errorResponseTransformStrategy) {
    ServerConfig serverConfig = Mockito.mock(ServerConfig.class);
    Mockito.when(serverConfig.getErrorResponseTransformStrategy()).thenReturn(errorResponseTransformStrategy);
    return serverConfig;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_28
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.sql.calcite.planner.DruidOperatorTable`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
private static DruidOperatorTable createMockDruidOperatorTable(List<SqlOperator> operatorList) {
    DruidOperatorTable mockOperatorTable = Mockito.mock(DruidOperatorTable.class);
    Mockito.when(mockOperatorTable.getOperatorList()).thenReturn(operatorList);
    return mockOperatorTable;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_28_1
#### Test Case Name: `testScanRoutinesTableWithCustomOperators`(File: `C:\Java_projects\Apache\druid\sql\src\test\java\org\apache\druid\sql\calcite\schema\InformationSchemaTest.java`)
#### Mock Object Variable Name: `mockOperatorTable`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final List<SqlOperator> sqlOperators = sqlOperatorConversions.stream().map(SqlOperatorConversion::calciteOperator).collect(Collectors.toList());
-    DruidOperatorTable mockOperatorTable = Mockito.mock(DruidOperatorTable.class);
-    Mockito.when(mockOperatorTable.getOperatorList()).thenReturn(sqlOperators);
+    DruidOperatorTable mockOperatorTable = createMockDruidOperatorTable(sqlOperators);
    InformationSchema.RoutinesTable routinesTable = new InformationSchema.RoutinesTable(mockOperatorTable);
    DataContext dataContext = createDataContext();
    List<Object[]> rows = routinesTable.scan(dataContext).toList();
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testScanRoutinesTableWithCustomOperators() {
    Set<SqlOperatorConversion> sqlOperatorConversions = customOperatorsToOperatorConversions();
    final List<SqlOperator> sqlOperators = sqlOperatorConversions.stream().map(SqlOperatorConversion::calciteOperator).collect(Collectors.toList());
    DruidOperatorTable mockOperatorTable = Mockito.mock(DruidOperatorTable.class);
    Mockito.when(mockOperatorTable.getOperatorList()).thenReturn(sqlOperators);
    InformationSchema.RoutinesTable routinesTable = new InformationSchema.RoutinesTable(mockOperatorTable);
    DataContext dataContext = createDataContext();
    List<Object[]> rows = routinesTable.scan(dataContext).toList();
    Assert.assertNotNull(rows);
    Assert.assertEquals("There should be exactly 2 rows; any non-function syntax operator should get filtered out", 2, rows.size());
    Object[] expectedRow1 = { "druid", "INFORMATION_SCHEMA", "FOO", "FUNCTION", "NO", "'FOO([<ANY>])'" };
    Assert.assertTrue(rows.stream().anyMatch(row -> Arrays.equals(row, expectedRow1)));
    Object[] expectedRow2 = { "druid", "INFORMATION_SCHEMA", "BAR", "FUNCTION", "NO", "'BAR(<INTEGER>, <INTEGER>)'" };
    Assert.assertTrue(rows.stream().anyMatch(row -> Arrays.equals(row, expectedRow2)));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static DruidOperatorTable createMockDruidOperatorTable(List<SqlOperator> operatorList) {
    DruidOperatorTable mockOperatorTable = Mockito.mock(DruidOperatorTable.class);
    Mockito.when(mockOperatorTable.getOperatorList()).thenReturn(operatorList);
    return mockOperatorTable;
}
```
</details>

---
#### Test Case ID #druid_Test_28_2
#### Test Case Name: `testScanRoutinesTableWithAnEmptyOperatorTable`(File: `C:\Java_projects\Apache\druid\sql\src\test\java\org\apache\druid\sql\calcite\schema\InformationSchemaTest.java`)
#### Mock Object Variable Name: `mockOperatorTable`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testScanRoutinesTableWithAnEmptyOperatorTable() {
-    DruidOperatorTable mockOperatorTable = Mockito.mock(DruidOperatorTable.class);
     // This should never happen. Adding a test case, so we don't hit a NPE and instead return gracefully.
     List<SqlOperator> emptyOperatorList = new ArrayList<>();
-    Mockito.when(mockOperatorTable.getOperatorList()).thenReturn(emptyOperatorList);
+    DruidOperatorTable mockOperatorTable = createMockDruidOperatorTable(emptyOperatorList);
     InformationSchema.RoutinesTable routinesTable = new InformationSchema.RoutinesTable(mockOperatorTable);
     DataContext dataContext = createDataContext();
     List<Object[]> rows = routinesTable.scan(dataContext).toList();
     Assert.assertNotNull(rows);
     Assert.assertEquals(0, rows.size());
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testScanRoutinesTableWithAnEmptyOperatorTable() {
    DruidOperatorTable mockOperatorTable = Mockito.mock(DruidOperatorTable.class);
    // This should never happen. Adding a test case, so we don't hit a NPE and instead return gracefully.
    List<SqlOperator> emptyOperatorList = new ArrayList<>();
    Mockito.when(mockOperatorTable.getOperatorList()).thenReturn(emptyOperatorList);
    InformationSchema.RoutinesTable routinesTable = new InformationSchema.RoutinesTable(mockOperatorTable);
    DataContext dataContext = createDataContext();
    List<Object[]> rows = routinesTable.scan(dataContext).toList();
    Assert.assertNotNull(rows);
    Assert.assertEquals(0, rows.size());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static DruidOperatorTable createMockDruidOperatorTable(List<SqlOperator> operatorList) {
    DruidOperatorTable mockOperatorTable = Mockito.mock(DruidOperatorTable.class);
    Mockito.when(mockOperatorTable.getOperatorList()).thenReturn(operatorList);
    return mockOperatorTable;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_29
- **Scope**: method level
- **Mocked Class**: `com.fasterxml.jackson.databind.ObjectMapper`
- **Test Case Count**: 6
- **MO Count**: 6

### Reusable Method
```java
// === Declare in class scope ===
private ObjectMapper mockMapper;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockMapper = Mockito.mock(ObjectMapper.class);
}

// === Replace local variable in test with ===
mockMapper

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_29_1
#### Test Case Name: `testHandleExceptionWithFilterDisabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `mockMapper`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleExceptionWithFilterDisabled() throws Exception {
     String errorMessage = "test exception message";
-    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
+    // removed local mock; replaced with global field `mockMapper`
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
     ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
     Exception testException = new IllegalStateException(errorMessage);
-    servlet.handleException(response, mockMapper, testException);
+    servlet.handleException(response, mockMapper, testException);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
-    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
+    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertEquals(errorMessage, captor.getValue().getMessage());
     Assert.assertEquals(IllegalStateException.class.getName(), ((QueryException) captor.getValue()).getErrorClass());
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleExceptionWithFilterDisabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
    Exception testException = new IllegalStateException(errorMessage);
    servlet.handleException(response, mockMapper, testException);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertEquals(IllegalStateException.class.getName(), ((QueryException) captor.getValue()).getErrorClass());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ObjectMapper mockMapper;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockMapper = Mockito.mock(ObjectMapper.class);
}

// === Replace local variable in test with ===
mockMapper

```
</details>

---
#### Test Case ID #druid_Test_29_2
#### Test Case Name: `testHandleExceptionWithFilterEnabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `mockMapper`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleExceptionWithFilterEnabled() throws Exception {
     String errorMessage = "test exception message";
-    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
+    // removed local mock; replaced with global field `mockMapper`
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
     ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

         @Override
         public boolean isShowDetailedJettyErrors() {
             return true;
         }

         @Override
         public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
             return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
         }
     });
     Exception testException = new IllegalStateException(errorMessage);
     servlet.handleException(response, mockMapper, testException);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
-    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
+    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertNull(captor.getValue().getMessage());
     Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
     Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleExceptionWithFilterEnabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
        }
    });
    Exception testException = new IllegalStateException(errorMessage);
    servlet.handleException(response, mockMapper, testException);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertNull(captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ObjectMapper mockMapper;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockMapper = Mockito.mock(ObjectMapper.class);
}

// === Replace local variable in test with ===
mockMapper

```
</details>

---
#### Test Case ID #druid_Test_29_3
#### Test Case Name: `testHandleExceptionWithFilterEnabledButMessageMatchAllowedRegex`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `mockMapper`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleExceptionWithFilterEnabledButMessageMatchAllowedRegex() throws Exception {
     String errorMessage = "test exception message";
-    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
+    // removed local mock; replaced with global field `mockMapper`
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
     ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

         @Override
         public boolean isShowDetailedJettyErrors() {
             return true;
         }

         @Override
         public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
             return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of("test .*"));
         }
     });
     Exception testException = new IllegalStateException(errorMessage);
     servlet.handleException(response, mockMapper, testException);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
-    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
+    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertEquals(errorMessage, captor.getValue().getMessage());
     Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
     Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleExceptionWithFilterEnabledButMessageMatchAllowedRegex() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of("test .*"));
        }
    });
    Exception testException = new IllegalStateException(errorMessage);
    servlet.handleException(response, mockMapper, testException);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ObjectMapper mockMapper;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockMapper = Mockito.mock(ObjectMapper.class);
}

// === Replace local variable in test with ===
mockMapper

```
</details>

---
#### Test Case ID #druid_Test_29_4
#### Test Case Name: `testHandleQueryParseExceptionWithFilterDisabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `mockMapper`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleQueryParseExceptionWithFilterDisabled() throws Exception {
     String errorMessage = "test exception message";
-    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
+    // removed local mock; replaced with global field `mockMapper`
     HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
     ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
     IOException testException = new IOException(errorMessage);
     servlet.handleQueryParseException(request, response, mockMapper, testException, false);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
-    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
+    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertEquals(errorMessage, captor.getValue().getMessage());
     Assert.assertEquals(IOException.class.getName(), ((QueryException) captor.getValue()).getErrorClass());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleQueryParseExceptionWithFilterDisabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertEquals(IOException.class.getName(), ((QueryException) captor.getValue()).getErrorClass());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ObjectMapper mockMapper;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockMapper = Mockito.mock(ObjectMapper.class);
}

// === Replace local variable in test with ===
mockMapper

```
</details>

---
#### Test Case ID #druid_Test_29_5
#### Test Case Name: `testHandleQueryParseExceptionWithFilterEnabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `mockMapper`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleQueryParseExceptionWithFilterEnabled() throws Exception {
     String errorMessage = "test exception message";
-    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
+    // removed local mock; replaced with global field `mockMapper`
     HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
     ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

         @Override
         public boolean isShowDetailedJettyErrors() {
             return true;
         }

         @Override
         public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
             return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
         }
     });
     IOException testException = new IOException(errorMessage);
     servlet.handleQueryParseException(request, response, mockMapper, testException, false);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
-    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
+    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertNull(captor.getValue().getMessage());
     Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
     Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleQueryParseExceptionWithFilterEnabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
        }
    });
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertNull(captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ObjectMapper mockMapper;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockMapper = Mockito.mock(ObjectMapper.class);
}

// === Replace local variable in test with ===
mockMapper

```
</details>

---
#### Test Case ID #druid_Test_29_6
#### Test Case Name: `testHandleQueryParseExceptionWithFilterEnabledButMessageMatchAllowedRegex`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `mockMapper`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testHandleQueryParseExceptionWithFilterEnabledButMessageMatchAllowedRegex() throws Exception {
     String errorMessage = "test exception message";
-    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
+    // removed local mock; replaced with global field `mockMapper`
     HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
     HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
     ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
     Mockito.when(response.getOutputStream()).thenReturn(outputStream);
     final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

         @Override
         public boolean isShowDetailedJettyErrors() {
             return true;
         }

         @Override
         public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
             return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of("test .*"));
         }
     });
     IOException testException = new IOException(errorMessage);
     servlet.handleQueryParseException(request, response, mockMapper, testException, false);
     ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
-    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
+    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
     Assert.assertTrue(captor.getValue() instanceof QueryException);
     Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
     Assert.assertEquals(errorMessage, captor.getValue().getMessage());
     Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
     Assert.assertNull(((QueryException) captor.getValue()).getHost());
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleQueryParseExceptionWithFilterEnabledButMessageMatchAllowedRegex() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of("test .*"));
        }
    });
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ObjectMapper mockMapper;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    mockMapper = Mockito.mock(ObjectMapper.class);
}

// === Replace local variable in test with ===
mockMapper

```
</details>

---
## Mock Clone Instance #druid_MCI_30
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.client.indexing.ClientCompactionTaskQueryTuningConfig`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
private static ClientCompactionTaskQueryTuningConfig createMockClientCompactionTaskQueryTuningConfig(Integer maxNumConcurrentSubTasks, PartitionsSpec partitionsSpec) {
    ClientCompactionTaskQueryTuningConfig tuningConfig = Mockito.mock(ClientCompactionTaskQueryTuningConfig.class);
    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(maxNumConcurrentSubTasks);
    Mockito.when(tuningConfig.getPartitionsSpec()).thenReturn(partitionsSpec);
    return tuningConfig;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_30_1
#### Test Case Name: `testIsParallelModeNonRangePartitionVaryingMaxNumConcurrentSubTasks`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `tuningConfig`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testIsParallelModeNonRangePartitionVaryingMaxNumConcurrentSubTasks() {
-    ClientCompactionTaskQueryTuningConfig tuningConfig = Mockito.mock(ClientCompactionTaskQueryTuningConfig.class);
-    Mockito.when(tuningConfig.getPartitionsSpec()).thenReturn(Mockito.mock(PartitionsSpec.class));
+    PartitionsSpec partitionsSpec = Mockito.mock(PartitionsSpec.class);
+    ClientCompactionTaskQueryTuningConfig tuningConfig = createMockClientCompactionTaskQueryTuningConfig(null, partitionsSpec);
    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(null);
    Assert.assertFalse(CompactSegments.isParallelMode(tuningConfig));
    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(1);
    Assert.assertFalse(CompactSegments.isParallelMode(tuningConfig));
    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(2);
    Assert.assertTrue(CompactSegments.isParallelMode(tuningConfig));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testIsParallelModeNonRangePartitionVaryingMaxNumConcurrentSubTasks() {
    ClientCompactionTaskQueryTuningConfig tuningConfig = Mockito.mock(ClientCompactionTaskQueryTuningConfig.class);
    Mockito.when(tuningConfig.getPartitionsSpec()).thenReturn(Mockito.mock(PartitionsSpec.class));
    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(null);
    Assert.assertFalse(CompactSegments.isParallelMode(tuningConfig));
    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(1);
    Assert.assertFalse(CompactSegments.isParallelMode(tuningConfig));
    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(2);
    Assert.assertTrue(CompactSegments.isParallelMode(tuningConfig));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ClientCompactionTaskQueryTuningConfig createMockClientCompactionTaskQueryTuningConfig(Integer maxNumConcurrentSubTasks, PartitionsSpec partitionsSpec) {
    ClientCompactionTaskQueryTuningConfig tuningConfig = Mockito.mock(ClientCompactionTaskQueryTuningConfig.class);
    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(maxNumConcurrentSubTasks);
    Mockito.when(tuningConfig.getPartitionsSpec()).thenReturn(partitionsSpec);
    return tuningConfig;
}
```
</details>

---
#### Test Case ID #druid_Test_30_2
#### Test Case Name: `testFindMaxNumTaskSlotsUsedByOneCompactionTaskWhenIsParallelMode`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `tuningConfig`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testFindMaxNumTaskSlotsUsedByOneCompactionTaskWhenIsParallelMode() {
-    ClientCompactionTaskQueryTuningConfig tuningConfig = Mockito.mock(ClientCompactionTaskQueryTuningConfig.class);
-    Mockito.when(tuningConfig.getPartitionsSpec()).thenReturn(Mockito.mock(PartitionsSpec.class));
-    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(2);
+    PartitionsSpec partitionsSpec = Mockito.mock(PartitionsSpec.class);
+    ClientCompactionTaskQueryTuningConfig tuningConfig = createMockClientCompactionTaskQueryTuningConfig(2, partitionsSpec);
     Assert.assertEquals(3, CompactSegments.findMaxNumTaskSlotsUsedByOneCompactionTask(tuningConfig));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testFindMaxNumTaskSlotsUsedByOneCompactionTaskWhenIsParallelMode() {
    ClientCompactionTaskQueryTuningConfig tuningConfig = Mockito.mock(ClientCompactionTaskQueryTuningConfig.class);
    Mockito.when(tuningConfig.getPartitionsSpec()).thenReturn(Mockito.mock(PartitionsSpec.class));
    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(2);
    Assert.assertEquals(3, CompactSegments.findMaxNumTaskSlotsUsedByOneCompactionTask(tuningConfig));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ClientCompactionTaskQueryTuningConfig createMockClientCompactionTaskQueryTuningConfig(Integer maxNumConcurrentSubTasks, PartitionsSpec partitionsSpec) {
    ClientCompactionTaskQueryTuningConfig tuningConfig = Mockito.mock(ClientCompactionTaskQueryTuningConfig.class);
    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(maxNumConcurrentSubTasks);
    Mockito.when(tuningConfig.getPartitionsSpec()).thenReturn(partitionsSpec);
    return tuningConfig;
}
```
</details>

---
#### Test Case ID #druid_Test_30_3
#### Test Case Name: `testFindMaxNumTaskSlotsUsedByOneCompactionTaskWhenIsSequentialMode`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\coordinator\duty\CompactSegmentsTest.java`)
#### Mock Object Variable Name: `tuningConfig`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testFindMaxNumTaskSlotsUsedByOneCompactionTaskWhenIsSequentialMode() {
-    ClientCompactionTaskQueryTuningConfig tuningConfig = Mockito.mock(ClientCompactionTaskQueryTuningConfig.class);
-    Mockito.when(tuningConfig.getPartitionsSpec()).thenReturn(Mockito.mock(PartitionsSpec.class));
-    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(1);
+    PartitionsSpec partitionsSpec = Mockito.mock(PartitionsSpec.class);
+    ClientCompactionTaskQueryTuningConfig tuningConfig = createMockClientCompactionTaskQueryTuningConfig(1, partitionsSpec);
     Assert.assertEquals(1, CompactSegments.findMaxNumTaskSlotsUsedByOneCompactionTask(tuningConfig));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testFindMaxNumTaskSlotsUsedByOneCompactionTaskWhenIsSequentialMode() {
    ClientCompactionTaskQueryTuningConfig tuningConfig = Mockito.mock(ClientCompactionTaskQueryTuningConfig.class);
    Mockito.when(tuningConfig.getPartitionsSpec()).thenReturn(Mockito.mock(PartitionsSpec.class));
    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(1);
    Assert.assertEquals(1, CompactSegments.findMaxNumTaskSlotsUsedByOneCompactionTask(tuningConfig));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ClientCompactionTaskQueryTuningConfig createMockClientCompactionTaskQueryTuningConfig(Integer maxNumConcurrentSubTasks, PartitionsSpec partitionsSpec) {
    ClientCompactionTaskQueryTuningConfig tuningConfig = Mockito.mock(ClientCompactionTaskQueryTuningConfig.class);
    Mockito.when(tuningConfig.getMaxNumConcurrentSubTasks()).thenReturn(maxNumConcurrentSubTasks);
    Mockito.when(tuningConfig.getPartitionsSpec()).thenReturn(partitionsSpec);
    return tuningConfig;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_31
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.indexing.overlord.duty.OverlordDuty`
- **Test Case Count**: 2
- **MO Count**: 3

### Reusable Method
```java
private static OverlordDuty createMockOverlordDuty(boolean isEnabledReturn) {
    OverlordDuty mockDuty = Mockito.mock(OverlordDuty.class);
    Mockito.when(mockDuty.isEnabled()).thenReturn(isEnabledReturn);
    return mockDuty;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_31_1
#### Test Case Name: `testStartAndStop`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\overlord\duty\OverlordDutyExecutorTest.java`)
#### Mock Object Variable Name: `testDuty1`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testStartAndStop() {
-    OverlordDuty testDuty1 = Mockito.mock(OverlordDuty.class);
-    Mockito.when(testDuty1.isEnabled()).thenReturn(true);
-    Mockito.when(testDuty1.getSchedule()).thenReturn(new DutySchedule(0, 0));
+    OverlordDuty testDuty1 = createMockOverlordDuty(true);
+    Mockito.when(testDuty1.getSchedule()).thenReturn(new DutySchedule(0, 0));
    OverlordDuty testDuty2 = Mockito.mock(OverlordDuty.class);
    Mockito.when(testDuty2.isEnabled()).thenReturn(true);
    Mockito.when(testDuty2.getSchedule()).thenReturn(new DutySchedule(0, 0));
    ScheduledExecutorFactory executorFactory = Mockito.mock(ScheduledExecutorFactory.class);
    ScheduledExecutorService executorService = Mockito.mock(ScheduledExecutorService.class);
    Mockito.when(executorFactory.create(ArgumentMatchers.eq(1), ArgumentMatchers.anyString())).thenReturn(executorService);
    OverlordDutyExecutor dutyExecutor = new OverlordDutyExecutor(executorFactory, ImmutableSet.of(testDuty1, testDuty2));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStartAndStop() {
    OverlordDuty testDuty1 = Mockito.mock(OverlordDuty.class);
    Mockito.when(testDuty1.isEnabled()).thenReturn(true);
    Mockito.when(testDuty1.getSchedule()).thenReturn(new DutySchedule(0, 0));
    OverlordDuty testDuty2 = Mockito.mock(OverlordDuty.class);
    Mockito.when(testDuty2.isEnabled()).thenReturn(true);
    Mockito.when(testDuty2.getSchedule()).thenReturn(new DutySchedule(0, 0));
    ScheduledExecutorFactory executorFactory = Mockito.mock(ScheduledExecutorFactory.class);
    ScheduledExecutorService executorService = Mockito.mock(ScheduledExecutorService.class);
    Mockito.when(executorFactory.create(ArgumentMatchers.eq(1), ArgumentMatchers.anyString())).thenReturn(executorService);
    OverlordDutyExecutor dutyExecutor = new OverlordDutyExecutor(executorFactory, ImmutableSet.of(testDuty1, testDuty2));
    // Invoke start multiple times
    dutyExecutor.start();
    dutyExecutor.start();
    dutyExecutor.start();
    // Verify that executor is initialized and each duty is scheduled
    Mockito.verify(executorFactory, Mockito.times(1)).create(ArgumentMatchers.eq(1), ArgumentMatchers.anyString());
    Mockito.verify(executorService, Mockito.times(2)).schedule(ArgumentMatchers.any(Runnable.class), ArgumentMatchers.anyLong(), ArgumentMatchers.any());
    // Invoke stop multiple times
    dutyExecutor.stop();
    dutyExecutor.stop();
    dutyExecutor.stop();
    // Verify that the executor shutdown is invoked just once
    Mockito.verify(executorService, Mockito.times(1)).shutdownNow();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static OverlordDuty createMockOverlordDuty(boolean isEnabledReturn) {
    OverlordDuty mockDuty = Mockito.mock(OverlordDuty.class);
    Mockito.when(mockDuty.isEnabled()).thenReturn(isEnabledReturn);
    return mockDuty;
}
```
</details>

---
#### Test Case ID #druid_Test_31_2
#### Test Case Name: `testStartAndStop`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\overlord\duty\OverlordDutyExecutorTest.java`)
#### Mock Object Variable Name: `testDuty2`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    OverlordDuty testDuty1 = Mockito.mock(OverlordDuty.class);
    Mockito.when(testDuty1.isEnabled()).thenReturn(true);
    Mockito.when(testDuty1.getSchedule()).thenReturn(new DutySchedule(0, 0));
-    OverlordDuty testDuty2 = Mockito.mock(OverlordDuty.class);
-    Mockito.when(testDuty2.isEnabled()).thenReturn(true);
-    Mockito.when(testDuty2.getSchedule()).thenReturn(new DutySchedule(0, 0));
+    OverlordDuty testDuty2 = createMockOverlordDuty(true);
+    Mockito.when(testDuty2.getSchedule()).thenReturn(new DutySchedule(0, 0));
    ScheduledExecutorFactory executorFactory = Mockito.mock(ScheduledExecutorFactory.class);
    ScheduledExecutorService executorService = Mockito.mock(ScheduledExecutorService.class);
    Mockito.when(executorFactory.create(ArgumentMatchers.eq(1), ArgumentMatchers.anyString())).thenReturn(executorService);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStartAndStop() {
    OverlordDuty testDuty1 = Mockito.mock(OverlordDuty.class);
    Mockito.when(testDuty1.isEnabled()).thenReturn(true);
    Mockito.when(testDuty1.getSchedule()).thenReturn(new DutySchedule(0, 0));
    OverlordDuty testDuty2 = Mockito.mock(OverlordDuty.class);
    Mockito.when(testDuty2.isEnabled()).thenReturn(true);
    Mockito.when(testDuty2.getSchedule()).thenReturn(new DutySchedule(0, 0));
    ScheduledExecutorFactory executorFactory = Mockito.mock(ScheduledExecutorFactory.class);
    ScheduledExecutorService executorService = Mockito.mock(ScheduledExecutorService.class);
    Mockito.when(executorFactory.create(ArgumentMatchers.eq(1), ArgumentMatchers.anyString())).thenReturn(executorService);
    OverlordDutyExecutor dutyExecutor = new OverlordDutyExecutor(executorFactory, ImmutableSet.of(testDuty1, testDuty2));
    // Invoke start multiple times
    dutyExecutor.start();
    dutyExecutor.start();
    dutyExecutor.start();
    // Verify that executor is initialized and each duty is scheduled
    Mockito.verify(executorFactory, Mockito.times(1)).create(ArgumentMatchers.eq(1), ArgumentMatchers.anyString());
    Mockito.verify(executorService, Mockito.times(2)).schedule(ArgumentMatchers.any(Runnable.class), ArgumentMatchers.anyLong(), ArgumentMatchers.any());
    // Invoke stop multiple times
    dutyExecutor.stop();
    dutyExecutor.stop();
    dutyExecutor.stop();
    // Verify that the executor shutdown is invoked just once
    Mockito.verify(executorService, Mockito.times(1)).shutdownNow();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static OverlordDuty createMockOverlordDuty(boolean isEnabledReturn) {
    OverlordDuty mockDuty = Mockito.mock(OverlordDuty.class);
    Mockito.when(mockDuty.isEnabled()).thenReturn(isEnabledReturn);
    return mockDuty;
}
```
</details>

---
#### Test Case ID #druid_Test_31_3
#### Test Case Name: `testStartWithNoEnabledDuty`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\overlord\duty\OverlordDutyExecutorTest.java`)
#### Mock Object Variable Name: `testDuty`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testStartWithNoEnabledDuty() {
-    OverlordDuty testDuty = Mockito.mock(OverlordDuty.class);
-    Mockito.when(testDuty.isEnabled()).thenReturn(false);
+    OverlordDuty testDuty = createMockOverlordDuty(false);
     ScheduledExecutorFactory executorFactory = Mockito.mock(ScheduledExecutorFactory.class);
     OverlordDutyExecutor dutyExecutor = new OverlordDutyExecutor(executorFactory, Collections.singleton(testDuty));
     dutyExecutor.start();
     // Verify that executor is not initialized as there is no enabled duty
     Mockito.verify(executorFactory, Mockito.never()).create(ArgumentMatchers.eq(1), ArgumentMatchers.anyString());
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStartWithNoEnabledDuty() {
    OverlordDuty testDuty = Mockito.mock(OverlordDuty.class);
    Mockito.when(testDuty.isEnabled()).thenReturn(false);
    ScheduledExecutorFactory executorFactory = Mockito.mock(ScheduledExecutorFactory.class);
    OverlordDutyExecutor dutyExecutor = new OverlordDutyExecutor(executorFactory, Collections.singleton(testDuty));
    dutyExecutor.start();
    // Verify that executor is not initialized as there is no enabled duty
    Mockito.verify(executorFactory, Mockito.never()).create(ArgumentMatchers.eq(1), ArgumentMatchers.anyString());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static OverlordDuty createMockOverlordDuty(boolean isEnabledReturn) {
    OverlordDuty mockDuty = Mockito.mock(OverlordDuty.class);
    Mockito.when(mockDuty.isEnabled()).thenReturn(isEnabledReturn);
    return mockDuty;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_32
- **Scope**: method level
- **Mocked Class**: `javax.servlet.http.HttpServletResponse`
- **Test Case Count**: 6
- **MO Count**: 6

### Reusable Method
```java
private static HttpServletResponse createMockHttpServletResponse(ServletOutputStream outputStream) {
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    return response;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_32_1
#### Test Case Name: `testHandleExceptionWithFilterDisabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `response`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
-    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
-    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
+    HttpServletResponse response = createMockHttpServletResponse(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
    Exception testException = new IllegalStateException(errorMessage);
    servlet.handleException(response, mockMapper, testException);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleExceptionWithFilterDisabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
    Exception testException = new IllegalStateException(errorMessage);
    servlet.handleException(response, mockMapper, testException);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertEquals(IllegalStateException.class.getName(), ((QueryException) captor.getValue()).getErrorClass());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HttpServletResponse createMockHttpServletResponse(ServletOutputStream outputStream) {
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    return response;
}
```
</details>

---
#### Test Case ID #druid_Test_32_2
#### Test Case Name: `testHandleExceptionWithFilterEnabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `response`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
-    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
-    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
+    HttpServletResponse response = createMockHttpServletResponse(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleExceptionWithFilterEnabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
        }
    });
    Exception testException = new IllegalStateException(errorMessage);
    servlet.handleException(response, mockMapper, testException);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertNull(captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HttpServletResponse createMockHttpServletResponse(ServletOutputStream outputStream) {
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    return response;
}
```
</details>

---
#### Test Case ID #druid_Test_32_3
#### Test Case Name: `testHandleExceptionWithFilterEnabledButMessageMatchAllowedRegex`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `response`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
-    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
-    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
+    HttpServletResponse response = createMockHttpServletResponse(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleExceptionWithFilterEnabledButMessageMatchAllowedRegex() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of("test .*"));
        }
    });
    Exception testException = new IllegalStateException(errorMessage);
    servlet.handleException(response, mockMapper, testException);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HttpServletResponse createMockHttpServletResponse(ServletOutputStream outputStream) {
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    return response;
}
```
</details>

---
#### Test Case ID #druid_Test_32_4
#### Test Case Name: `testHandleQueryParseExceptionWithFilterDisabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `response`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
-    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
-    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
-    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
+    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
+    HttpServletResponse response = createMockHttpServletResponse(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleQueryParseExceptionWithFilterDisabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig());
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertEquals(IOException.class.getName(), ((QueryException) captor.getValue()).getErrorClass());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HttpServletResponse createMockHttpServletResponse(ServletOutputStream outputStream) {
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    return response;
}
```
</details>

---
#### Test Case ID #druid_Test_32_5
#### Test Case Name: `testHandleQueryParseExceptionWithFilterEnabled`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `response`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
-    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
-    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
-    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
+    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
+    HttpServletResponse response = createMockHttpServletResponse(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleQueryParseExceptionWithFilterEnabled() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of());
        }
    });
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertNull(captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HttpServletResponse createMockHttpServletResponse(ServletOutputStream outputStream) {
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    return response;
}
```
</details>

---
#### Test Case ID #druid_Test_32_6
#### Test Case Name: `testHandleQueryParseExceptionWithFilterEnabledButMessageMatchAllowedRegex`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\server\AsyncQueryForwardingServletTest.java`)
#### Mock Object Variable Name: `response`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
-    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
-    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
+    HttpServletResponse response = createMockHttpServletResponse(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testHandleQueryParseExceptionWithFilterEnabledButMessageMatchAllowedRegex() throws Exception {
    String errorMessage = "test exception message";
    ObjectMapper mockMapper = Mockito.mock(ObjectMapper.class);
    HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    ServletOutputStream outputStream = Mockito.mock(ServletOutputStream.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    final AsyncQueryForwardingServlet servlet = new AsyncQueryForwardingServlet(new MapQueryToolChestWarehouse(ImmutableMap.of()), mockMapper, TestHelper.makeSmileMapper(), null, null, null, new NoopServiceEmitter(), new NoopRequestLogger(), new DefaultGenericQueryMetricsFactory(), new AuthenticatorMapper(ImmutableMap.of()), new Properties(), new ServerConfig() {

        @Override
        public boolean isShowDetailedJettyErrors() {
            return true;
        }

        @Override
        public ErrorResponseTransformStrategy getErrorResponseTransformStrategy() {
            return new AllowedRegexErrorResponseTransformStrategy(ImmutableList.of("test .*"));
        }
    });
    IOException testException = new IOException(errorMessage);
    servlet.handleQueryParseException(request, response, mockMapper, testException, false);
    ArgumentCaptor<Exception> captor = ArgumentCaptor.forClass(Exception.class);
    Mockito.verify(mockMapper).writeValue(ArgumentMatchers.eq(outputStream), captor.capture());
    Assert.assertTrue(captor.getValue() instanceof QueryException);
    Assert.assertEquals("Unknown exception", ((QueryException) captor.getValue()).getErrorCode());
    Assert.assertEquals(errorMessage, captor.getValue().getMessage());
    Assert.assertNull(((QueryException) captor.getValue()).getErrorClass());
    Assert.assertNull(((QueryException) captor.getValue()).getHost());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HttpServletResponse createMockHttpServletResponse(ServletOutputStream outputStream) {
    HttpServletResponse response = Mockito.mock(HttpServletResponse.class);
    Mockito.when(response.getOutputStream()).thenReturn(outputStream);
    return response;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_33
- **Scope**: class level
- **Mocked Class**: `org.apache.druid.segment.vector.VectorColumnSelectorFactory`
- **Test Case Count**: 6
- **MO Count**: 6

### Reusable Method
```java
public class MockVectorColumnSelectorFactory {
  public static VectorColumnSelectorFactory createMockVectorColumnSelectorFactory(
      String fieldName,
      ColumnCapabilities columnCapabilities,
      VectorValueSelector valueSelector
  ) {
    VectorColumnSelectorFactory selectorFactory = Mockito.mock(VectorColumnSelectorFactory.class, Answers.RETURNS_DEEP_STUBS);
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(fieldName);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(fieldName);
    if (columnCapabilities != null) {
      Mockito.doReturn(columnCapabilities).when(selectorFactory).getColumnCapabilities(fieldName);
    }
    return selectorFactory;
  }
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_33_1
#### Test Case Name: `factorizeVectorForNumericTypeShouldReturnDoubleVectorAggregator`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\aggregation\any\DoubleAnyAggregatorFactoryTest.java`)
#### Mock Object Variable Name: `selectorFactory`
<summary>Suggested Diff</summary>

```diff
--- original.java
+++ refactored.java
@@
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(FIELD_NAME);
    target = new DoubleAnyAggregatorFactory(NAME, FIELD_NAME);
}

@Test
public void factorizeVectorForNumericTypeShouldReturnDoubleVectorAggregator() {
-    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
+    selectorFactory = MockVectorColumnSelectorFactory.createMockVectorColumnSelectorFactory(FIELD_NAME, capabilities, valueSelector);
    Mockito.doReturn(true).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(DoubleAnyVectorAggregator.class, aggregator.getClass());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Before
public void setUp() {
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(FIELD_NAME);
    target = new DoubleAnyAggregatorFactory(NAME, FIELD_NAME);
}
@Test
public void factorizeVectorForNumericTypeShouldReturnDoubleVectorAggregator() {
    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(true).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(DoubleAnyVectorAggregator.class, aggregator.getClass());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockVectorColumnSelectorFactory {
  public static VectorColumnSelectorFactory createMockVectorColumnSelectorFactory(
      String fieldName,
      ColumnCapabilities columnCapabilities,
      VectorValueSelector valueSelector
  ) {
    VectorColumnSelectorFactory selectorFactory = Mockito.mock(VectorColumnSelectorFactory.class, Answers.RETURNS_DEEP_STUBS);
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(fieldName);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(fieldName);
    if (columnCapabilities != null) {
      Mockito.doReturn(columnCapabilities).when(selectorFactory).getColumnCapabilities(fieldName);
    }
    return selectorFactory;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_33_2
#### Test Case Name: `factorizeVectorForStringTypeShouldReturnDoubleVectorAggregatorWithNilSelector`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\aggregation\any\DoubleAnyAggregatorFactoryTest.java`)
#### Mock Object Variable Name: `selectorFactory`
<summary>Suggested Diff</summary>

```diff
--- Original.java
+++ Refactored.java
@@
    @Before
    public void setUp() {
-    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
-    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(FIELD_NAME);
        target = new DoubleAnyAggregatorFactory(NAME, FIELD_NAME);
    }

    @Test
    public void factorizeVectorForStringTypeShouldReturnDoubleVectorAggregatorWithNilSelector() {
-    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
+    selectorFactory = MockVectorColumnSelectorFactory.createMockVectorColumnSelectorFactory(FIELD_NAME, capabilities, valueSelector);
        VectorAggregator aggregator = target.factorizeVector(selectorFactory);
        Assert.assertNotNull(aggregator);
        Assert.assertEquals(NullHandling.defaultDoubleValue(), aggregator.get(BUFFER, POSITION));
    }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Before
public void setUp() {
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(FIELD_NAME);
    target = new DoubleAnyAggregatorFactory(NAME, FIELD_NAME);
}
@Test
public void factorizeVectorForStringTypeShouldReturnDoubleVectorAggregatorWithNilSelector() {
    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(false).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(NullHandling.defaultDoubleValue(), aggregator.get(BUFFER, POSITION));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockVectorColumnSelectorFactory {
  public static VectorColumnSelectorFactory createMockVectorColumnSelectorFactory(
      String fieldName,
      ColumnCapabilities columnCapabilities,
      VectorValueSelector valueSelector
  ) {
    VectorColumnSelectorFactory selectorFactory = Mockito.mock(VectorColumnSelectorFactory.class, Answers.RETURNS_DEEP_STUBS);
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(fieldName);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(fieldName);
    if (columnCapabilities != null) {
      Mockito.doReturn(columnCapabilities).when(selectorFactory).getColumnCapabilities(fieldName);
    }
    return selectorFactory;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_33_3
#### Test Case Name: `factorizeVectorForNumericTypeShouldReturnFloatVectorAggregator`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\aggregation\any\FloatAnyAggregatorFactoryTest.java`)
#### Mock Object Variable Name: `selectorFactory`
<summary>Suggested Diff</summary>

```diff
--- Original.java
+++ Refactored.java
@@
    @Before
    public void setUp() {
-    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
-    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(FIELD_NAME);
        target = new FloatAnyAggregatorFactory(NAME, FIELD_NAME);
    }

    @Test
    public void factorizeVectorForNumericTypeShouldReturnFloatVectorAggregator() {
-    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
+    selectorFactory = MockVectorColumnSelectorFactory.createMockVectorColumnSelectorFactory(FIELD_NAME, capabilities, valueSelector);
        Mockito.doReturn(true).when(capabilities).isNumeric();
        VectorAggregator aggregator = target.factorizeVector(selectorFactory);
        Assert.assertNotNull(aggregator);
        Assert.assertEquals(FloatAnyVectorAggregator.class, aggregator.getClass());
    }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void factorizeVectorForNumericTypeShouldReturnFloatVectorAggregator() {
    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(true).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(FloatAnyVectorAggregator.class, aggregator.getClass());
}
@Before
public void setUp() {
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(FIELD_NAME);
    target = new FloatAnyAggregatorFactory(NAME, FIELD_NAME);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockVectorColumnSelectorFactory {
  public static VectorColumnSelectorFactory createMockVectorColumnSelectorFactory(
      String fieldName,
      ColumnCapabilities columnCapabilities,
      VectorValueSelector valueSelector
  ) {
    VectorColumnSelectorFactory selectorFactory = Mockito.mock(VectorColumnSelectorFactory.class, Answers.RETURNS_DEEP_STUBS);
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(fieldName);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(fieldName);
    if (columnCapabilities != null) {
      Mockito.doReturn(columnCapabilities).when(selectorFactory).getColumnCapabilities(fieldName);
    }
    return selectorFactory;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_33_4
#### Test Case Name: `factorizeVectorForStringTypeShouldReturnFloatVectorAggregatorWithNilSelector`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\aggregation\any\FloatAnyAggregatorFactoryTest.java`)
#### Mock Object Variable Name: `selectorFactory`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void factorizeVectorForStringTypeShouldReturnFloatVectorAggregatorWithNilSelector() {
-    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
+    selectorFactory = MockVectorColumnSelectorFactory.createMockVectorColumnSelectorFactory(FIELD_NAME, capabilities, valueSelector);
     Mockito.doReturn(false).when(capabilities).isNumeric();
     VectorAggregator aggregator = target.factorizeVector(selectorFactory);
     Assert.assertNotNull(aggregator);
     Assert.assertEquals(NullHandling.defaultFloatValue(), aggregator.get(BUFFER, POSITION));
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void factorizeVectorForStringTypeShouldReturnFloatVectorAggregatorWithNilSelector() {
    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(false).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(NullHandling.defaultFloatValue(), aggregator.get(BUFFER, POSITION));
}
@Before
public void setUp() {
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(FIELD_NAME);
    target = new FloatAnyAggregatorFactory(NAME, FIELD_NAME);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockVectorColumnSelectorFactory {
  public static VectorColumnSelectorFactory createMockVectorColumnSelectorFactory(
      String fieldName,
      ColumnCapabilities columnCapabilities,
      VectorValueSelector valueSelector
  ) {
    VectorColumnSelectorFactory selectorFactory = Mockito.mock(VectorColumnSelectorFactory.class, Answers.RETURNS_DEEP_STUBS);
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(fieldName);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(fieldName);
    if (columnCapabilities != null) {
      Mockito.doReturn(columnCapabilities).when(selectorFactory).getColumnCapabilities(fieldName);
    }
    return selectorFactory;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_33_5
#### Test Case Name: `factorizeVectorWithNumericColumnShouldReturnLongVectorAggregator`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\aggregation\any\LongAnyAggregatorFactoryTest.java`)
#### Mock Object Variable Name: `selectorFactory`
<summary>Suggested Diff</summary>

```diff
--- original.java
+++ refactored.java
@@
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(FIELD_NAME);
    target = new LongAnyAggregatorFactory(NAME, FIELD_NAME);
}

@Test
public void factorizeVectorWithNumericColumnShouldReturnLongVectorAggregator() {
-    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
+    selectorFactory = MockVectorColumnSelectorFactory.createMockVectorColumnSelectorFactory(FIELD_NAME, capabilities, valueSelector);
    Mockito.doReturn(true).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(LongAnyVectorAggregator.class, aggregator.getClass());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void factorizeVectorWithNumericColumnShouldReturnLongVectorAggregator() {
    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(true).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(LongAnyVectorAggregator.class, aggregator.getClass());
}
@Before
public void setUp() {
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(FIELD_NAME);
    target = new LongAnyAggregatorFactory(NAME, FIELD_NAME);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockVectorColumnSelectorFactory {
  public static VectorColumnSelectorFactory createMockVectorColumnSelectorFactory(
      String fieldName,
      ColumnCapabilities columnCapabilities,
      VectorValueSelector valueSelector
  ) {
    VectorColumnSelectorFactory selectorFactory = Mockito.mock(VectorColumnSelectorFactory.class, Answers.RETURNS_DEEP_STUBS);
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(fieldName);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(fieldName);
    if (columnCapabilities != null) {
      Mockito.doReturn(columnCapabilities).when(selectorFactory).getColumnCapabilities(fieldName);
    }
    return selectorFactory;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_33_6
#### Test Case Name: `factorizeVectorForStringTypeShouldReturnLongVectorAggregatorWithNilSelector`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\aggregation\any\LongAnyAggregatorFactoryTest.java`)
#### Mock Object Variable Name: `selectorFactory`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void factorizeVectorForStringTypeShouldReturnLongVectorAggregatorWithNilSelector() {
-    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
+    selectorFactory = MockVectorColumnSelectorFactory.createMockVectorColumnSelectorFactory(FIELD_NAME, capabilities, valueSelector);
     Mockito.doReturn(false).when(capabilities).isNumeric();
     VectorAggregator aggregator = target.factorizeVector(selectorFactory);
     Assert.assertNotNull(aggregator);
     Assert.assertEquals(NullHandling.defaultLongValue(), aggregator.get(BUFFER, POSITION));
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void factorizeVectorForStringTypeShouldReturnLongVectorAggregatorWithNilSelector() {
    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(false).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(NullHandling.defaultLongValue(), aggregator.get(BUFFER, POSITION));
}
@Before
public void setUp() {
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(FIELD_NAME);
    target = new LongAnyAggregatorFactory(NAME, FIELD_NAME);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockVectorColumnSelectorFactory {
  public static VectorColumnSelectorFactory createMockVectorColumnSelectorFactory(
      String fieldName,
      ColumnCapabilities columnCapabilities,
      VectorValueSelector valueSelector
  ) {
    VectorColumnSelectorFactory selectorFactory = Mockito.mock(VectorColumnSelectorFactory.class, Answers.RETURNS_DEEP_STUBS);
    Mockito.doReturn(null).when(selectorFactory).getColumnCapabilities(fieldName);
    Mockito.doReturn(valueSelector).when(selectorFactory).makeValueSelector(fieldName);
    if (columnCapabilities != null) {
      Mockito.doReturn(columnCapabilities).when(selectorFactory).getColumnCapabilities(fieldName);
    }
    return selectorFactory;
  }
}
```
</details>

---
## Mock Clone Instance #druid_MCI_34
- **Scope**: method level
- **Mocked Class**: `oshi.software.os.OSFileStore`
- **Test Case Count**: 1
- **MO Count**: 2

### Reusable Method
```java
private static OSFileStore createMockOSFileStore(long totalSpace, long usableSpace, long totalInodes, long freeInodes, String volume, String mount) {
    OSFileStore mock = Mockito.mock(OSFileStore.class);
    Mockito.when(mock.getTotalSpace()).thenReturn(totalSpace);
    Mockito.when(mock.getUsableSpace()).thenReturn(usableSpace);
    Mockito.when(mock.getTotalInodes()).thenReturn(totalInodes);
    Mockito.when(mock.getFreeInodes()).thenReturn(freeInodes);
    Mockito.when(mock.getVolume()).thenReturn(volume);
    Mockito.when(mock.getMount()).thenReturn(mount);
    return mock;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_34_1
#### Test Case Name: `testFsStats`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\OshiSysMonitorTest.java`)
#### Mock Object Variable Name: `fs1`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    FileSystem fileSystem = Mockito.mock(FileSystem.class);
-    OSFileStore fs1 = Mockito.mock(OSFileStore.class);
+    OSFileStore fs1 = createMockOSFileStore(300L, 200L, 1000L, 700L, "/dev/disk1", "/System/Volumes/boot1");
    OSFileStore fs2 = Mockito.mock(OSFileStore.class);
-    Mockito.when(fs1.getTotalSpace()).thenReturn(300L);
-    Mockito.when(fs1.getUsableSpace()).thenReturn(200L);
-    Mockito.when(fs1.getTotalInodes()).thenReturn(1000L);
-    Mockito.when(fs1.getFreeInodes()).thenReturn(700L);
-    Mockito.when(fs1.getVolume()).thenReturn("/dev/disk1");
-    Mockito.when(fs1.getMount()).thenReturn("/System/Volumes/boot1");
    Mockito.when(fs2.getTotalSpace()).thenReturn(400L);
    Mockito.when(fs2.getUsableSpace()).thenReturn(320L);
    Mockito.when(fs2.getTotalInodes()).thenReturn(800L);
    Mockito.when(fs2.getFreeInodes()).thenReturn(600L);
    Mockito.when(fs2.getVolume()).thenReturn("/dev/disk2");
    Mockito.when(fs2.getMount()).thenReturn("/System/Volumes/boot2");
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testFsStats() {
    StubServiceEmitter emitter = new StubServiceEmitter("dev/monitor-test", "localhost:0000");
    FileSystem fileSystem = Mockito.mock(FileSystem.class);
    OSFileStore fs1 = Mockito.mock(OSFileStore.class);
    OSFileStore fs2 = Mockito.mock(OSFileStore.class);
    Mockito.when(fs1.getTotalSpace()).thenReturn(300L);
    Mockito.when(fs1.getUsableSpace()).thenReturn(200L);
    Mockito.when(fs1.getTotalInodes()).thenReturn(1000L);
    Mockito.when(fs1.getFreeInodes()).thenReturn(700L);
    Mockito.when(fs1.getVolume()).thenReturn("/dev/disk1");
    Mockito.when(fs1.getMount()).thenReturn("/System/Volumes/boot1");
    Mockito.when(fs2.getTotalSpace()).thenReturn(400L);
    Mockito.when(fs2.getUsableSpace()).thenReturn(320L);
    Mockito.when(fs2.getTotalInodes()).thenReturn(800L);
    Mockito.when(fs2.getFreeInodes()).thenReturn(600L);
    Mockito.when(fs2.getVolume()).thenReturn("/dev/disk2");
    Mockito.when(fs2.getMount()).thenReturn("/System/Volumes/boot2");
    List<OSFileStore> osFileStores = ImmutableList.of(fs1, fs2);
    Mockito.when(fileSystem.getFileStores(true)).thenReturn(osFileStores);
    Mockito.when(os.getFileSystem()).thenReturn(fileSystem);
    OshiSysMonitor m = new OshiSysMonitor(si);
    m.start();
    m.monitorFsStats(emitter);
    Assert.assertEquals(8, emitter.getEvents().size());
    emitter.verifyEmitted("sys/fs/max", 2);
    emitter.verifyEmitted("sys/fs/used", 2);
    emitter.verifyEmitted("sys/fs/files/count", 2);
    emitter.verifyEmitted("sys/fs/files/free", 2);
    Map<String, Object> userDims1 = ImmutableMap.of("fsDevName", "/dev/disk1", "fsDirName", "/System/Volumes/boot1");
    List<Number> metricValues1 = emitter.getMetricValues("sys/fs/max", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(300L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/fs/used", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(100L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/fs/files/count", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(1000L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/fs/files/free", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(700L, metricValues1.get(0));
    Map<String, Object> userDims2 = ImmutableMap.of("fsDevName", "/dev/disk2", "fsDirName", "/System/Volumes/boot2");
    List<Number> metricValues2 = emitter.getMetricValues("sys/fs/max", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(400L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/fs/used", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(80L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/fs/files/count", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(800L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/fs/files/free", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(600L, metricValues2.get(0));
    m.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static OSFileStore createMockOSFileStore(long totalSpace, long usableSpace, long totalInodes, long freeInodes, String volume, String mount) {
    OSFileStore mock = Mockito.mock(OSFileStore.class);
    Mockito.when(mock.getTotalSpace()).thenReturn(totalSpace);
    Mockito.when(mock.getUsableSpace()).thenReturn(usableSpace);
    Mockito.when(mock.getTotalInodes()).thenReturn(totalInodes);
    Mockito.when(mock.getFreeInodes()).thenReturn(freeInodes);
    Mockito.when(mock.getVolume()).thenReturn(volume);
    Mockito.when(mock.getMount()).thenReturn(mount);
    return mock;
}
```
</details>

---
#### Test Case ID #druid_Test_34_2
#### Test Case Name: `testFsStats`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\OshiSysMonitorTest.java`)
#### Mock Object Variable Name: `fs2`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    FileSystem fileSystem = Mockito.mock(FileSystem.class);
    OSFileStore fs1 = Mockito.mock(OSFileStore.class);
-    OSFileStore fs2 = Mockito.mock(OSFileStore.class);
-    Mockito.when(fs1.getTotalSpace()).thenReturn(300L);
-    Mockito.when(fs1.getUsableSpace()).thenReturn(200L);
-    Mockito.when(fs1.getTotalInodes()).thenReturn(1000L);
-    Mockito.when(fs1.getFreeInodes()).thenReturn(700L);
-    Mockito.when(fs1.getVolume()).thenReturn("/dev/disk1");
-    Mockito.when(fs1.getMount()).thenReturn("/System/Volumes/boot1");
-    Mockito.when(fs2.getTotalSpace()).thenReturn(400L);
-    Mockito.when(fs2.getUsableSpace()).thenReturn(320L);
-    Mockito.when(fs2.getTotalInodes()).thenReturn(800L);
-    Mockito.when(fs2.getFreeInodes()).thenReturn(600L);
-    Mockito.when(fs2.getVolume()).thenReturn("/dev/disk2");
-    Mockito.when(fs2.getMount()).thenReturn("/System/Volumes/boot2");
+    OSFileStore fs2 = createMockOSFileStore(400L, 320L, 800L, 600L, "/dev/disk2", "/System/Volumes/boot2");
+    Mockito.when(fs1.getTotalSpace()).thenReturn(300L);
+    Mockito.when(fs1.getUsableSpace()).thenReturn(200L);
+    Mockito.when(fs1.getTotalInodes()).thenReturn(1000L);
+    Mockito.when(fs1.getFreeInodes()).thenReturn(700L);
+    Mockito.when(fs1.getVolume()).thenReturn("/dev/disk1");
+    Mockito.when(fs1.getMount()).thenReturn("/System/Volumes/boot1");
    List<OSFileStore> osFileStores = ImmutableList.of(fs1, fs2);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testFsStats() {
    StubServiceEmitter emitter = new StubServiceEmitter("dev/monitor-test", "localhost:0000");
    FileSystem fileSystem = Mockito.mock(FileSystem.class);
    OSFileStore fs1 = Mockito.mock(OSFileStore.class);
    OSFileStore fs2 = Mockito.mock(OSFileStore.class);
    Mockito.when(fs1.getTotalSpace()).thenReturn(300L);
    Mockito.when(fs1.getUsableSpace()).thenReturn(200L);
    Mockito.when(fs1.getTotalInodes()).thenReturn(1000L);
    Mockito.when(fs1.getFreeInodes()).thenReturn(700L);
    Mockito.when(fs1.getVolume()).thenReturn("/dev/disk1");
    Mockito.when(fs1.getMount()).thenReturn("/System/Volumes/boot1");
    Mockito.when(fs2.getTotalSpace()).thenReturn(400L);
    Mockito.when(fs2.getUsableSpace()).thenReturn(320L);
    Mockito.when(fs2.getTotalInodes()).thenReturn(800L);
    Mockito.when(fs2.getFreeInodes()).thenReturn(600L);
    Mockito.when(fs2.getVolume()).thenReturn("/dev/disk2");
    Mockito.when(fs2.getMount()).thenReturn("/System/Volumes/boot2");
    List<OSFileStore> osFileStores = ImmutableList.of(fs1, fs2);
    Mockito.when(fileSystem.getFileStores(true)).thenReturn(osFileStores);
    Mockito.when(os.getFileSystem()).thenReturn(fileSystem);
    OshiSysMonitor m = new OshiSysMonitor(si);
    m.start();
    m.monitorFsStats(emitter);
    Assert.assertEquals(8, emitter.getEvents().size());
    emitter.verifyEmitted("sys/fs/max", 2);
    emitter.verifyEmitted("sys/fs/used", 2);
    emitter.verifyEmitted("sys/fs/files/count", 2);
    emitter.verifyEmitted("sys/fs/files/free", 2);
    Map<String, Object> userDims1 = ImmutableMap.of("fsDevName", "/dev/disk1", "fsDirName", "/System/Volumes/boot1");
    List<Number> metricValues1 = emitter.getMetricValues("sys/fs/max", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(300L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/fs/used", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(100L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/fs/files/count", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(1000L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/fs/files/free", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(700L, metricValues1.get(0));
    Map<String, Object> userDims2 = ImmutableMap.of("fsDevName", "/dev/disk2", "fsDirName", "/System/Volumes/boot2");
    List<Number> metricValues2 = emitter.getMetricValues("sys/fs/max", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(400L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/fs/used", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(80L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/fs/files/count", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(800L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/fs/files/free", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(600L, metricValues2.get(0));
    m.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static OSFileStore createMockOSFileStore(long totalSpace, long usableSpace, long totalInodes, long freeInodes, String volume, String mount) {
    OSFileStore mock = Mockito.mock(OSFileStore.class);
    Mockito.when(mock.getTotalSpace()).thenReturn(totalSpace);
    Mockito.when(mock.getUsableSpace()).thenReturn(usableSpace);
    Mockito.when(mock.getTotalInodes()).thenReturn(totalInodes);
    Mockito.when(mock.getFreeInodes()).thenReturn(freeInodes);
    Mockito.when(mock.getVolume()).thenReturn(volume);
    Mockito.when(mock.getMount()).thenReturn(mount);
    return mock;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_35
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.query.aggregation.AggregatorAdapters`
- **Test Case Count**: 4
- **MO Count**: 4

### Reusable Method
```java
private static AggregatorAdapters createMockAggregatorAdapters(int spaceNeededReturn) {
    AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(spaceNeededReturn);
    return aggregatorAdapters;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_35_1
#### Test Case Name: `testGrowOnce`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\HashVectorGrouperTest.java`)
#### Mock Object Variable Name: `aggregatorAdapters`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final int keySize = 4;
    final int aggSize = 8;
    final WritableMemory keySpace = WritableMemory.allocate(keySize * maxVectorSize);
-    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
-    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(aggSize);
+    final AggregatorAdapters aggregatorAdapters = createMockAggregatorAdapters(aggSize);
    int startingNumBuckets = 4;
    int maxBuckets = 16;
    final int bufferSize = (keySize + aggSize) * maxBuckets;
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testGrowOnce() {
    final int maxVectorSize = 512;
    final int keySize = 4;
    final int aggSize = 8;
    final WritableMemory keySpace = WritableMemory.allocate(keySize * maxVectorSize);
    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(aggSize);
    int startingNumBuckets = 4;
    int maxBuckets = 16;
    final int bufferSize = (keySize + aggSize) * maxBuckets;
    final ByteBuffer buffer = ByteBuffer.wrap(new byte[bufferSize]);
    final HashVectorGrouper grouper = new HashVectorGrouper(Suppliers.ofInstance(buffer), keySize, aggregatorAdapters, maxBuckets, 0.f, startingNumBuckets);
    grouper.initVectorized(maxVectorSize);
    int tableStart = grouper.getTableStart();
    // two keys should not cause buffer to grow
    fillKeyspace(keySpace, maxVectorSize, 2);
    AggregateResult result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertEquals(tableStart, grouper.getTableStart());
    // 3rd key should cause buffer to grow
    // buffer should grow to maximum size
    fillKeyspace(keySpace, maxVectorSize, 3);
    result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertEquals(0, grouper.getTableStart());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static AggregatorAdapters createMockAggregatorAdapters(int spaceNeededReturn) {
    AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(spaceNeededReturn);
    return aggregatorAdapters;
}
```
</details>

---
#### Test Case ID #druid_Test_35_2
#### Test Case Name: `testGrowTwice`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\HashVectorGrouperTest.java`)
#### Mock Object Variable Name: `aggregatorAdapters`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final int keySize = 4;
    final int aggSize = 8;
    final WritableMemory keySpace = WritableMemory.allocate(keySize * maxVectorSize);
-    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
-    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(aggSize);
+    final AggregatorAdapters aggregatorAdapters = createMockAggregatorAdapters(aggSize);
    int startingNumBuckets = 4;
    int maxBuckets = 32;
    final int bufferSize = (keySize + aggSize) * maxBuckets;
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testGrowTwice() {
    final int maxVectorSize = 512;
    final int keySize = 4;
    final int aggSize = 8;
    final WritableMemory keySpace = WritableMemory.allocate(keySize * maxVectorSize);
    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(aggSize);
    int startingNumBuckets = 4;
    int maxBuckets = 32;
    final int bufferSize = (keySize + aggSize) * maxBuckets;
    final ByteBuffer buffer = ByteBuffer.wrap(new byte[bufferSize]);
    final HashVectorGrouper grouper = new HashVectorGrouper(Suppliers.ofInstance(buffer), keySize, aggregatorAdapters, maxBuckets, 0.f, startingNumBuckets);
    grouper.initVectorized(maxVectorSize);
    int tableStart = grouper.getTableStart();
    // two keys should not cause buffer to grow
    fillKeyspace(keySpace, maxVectorSize, 2);
    AggregateResult result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertEquals(tableStart, grouper.getTableStart());
    // 3rd key should cause buffer to grow
    // buffer should grow to next size, but is not full
    fillKeyspace(keySpace, maxVectorSize, 3);
    result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertTrue(grouper.getTableStart() > tableStart);
    // this time should be all the way
    fillKeyspace(keySpace, maxVectorSize, 6);
    result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertEquals(0, grouper.getTableStart());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static AggregatorAdapters createMockAggregatorAdapters(int spaceNeededReturn) {
    AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(spaceNeededReturn);
    return aggregatorAdapters;
}
```
</details>

---
#### Test Case ID #druid_Test_35_3
#### Test Case Name: `testGrowThreeTimes`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\HashVectorGrouperTest.java`)
#### Mock Object Variable Name: `aggregatorAdapters`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final int keySize = 4;
    final int aggSize = 8;
    final WritableMemory keySpace = WritableMemory.allocate(keySize * maxVectorSize);
-    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
-    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(aggSize);
+    final AggregatorAdapters aggregatorAdapters = createMockAggregatorAdapters(aggSize);
    int startingNumBuckets = 4;
    int maxBuckets = 64;
    final int bufferSize = (keySize + aggSize) * maxBuckets;
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testGrowThreeTimes() {
    final int maxVectorSize = 512;
    final int keySize = 4;
    final int aggSize = 8;
    final WritableMemory keySpace = WritableMemory.allocate(keySize * maxVectorSize);
    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(aggSize);
    int startingNumBuckets = 4;
    int maxBuckets = 64;
    final int bufferSize = (keySize + aggSize) * maxBuckets;
    final ByteBuffer buffer = ByteBuffer.wrap(new byte[bufferSize]);
    final HashVectorGrouper grouper = new HashVectorGrouper(Suppliers.ofInstance(buffer), keySize, aggregatorAdapters, maxBuckets, 0.f, startingNumBuckets);
    grouper.initVectorized(maxVectorSize);
    int tableStart = grouper.getTableStart();
    // two keys should cause buffer to grow
    fillKeyspace(keySpace, maxVectorSize, 2);
    AggregateResult result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertEquals(tableStart, grouper.getTableStart());
    // 3rd key should cause buffer to grow
    // buffer should grow to next size, but is not full
    fillKeyspace(keySpace, maxVectorSize, 3);
    result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertTrue(grouper.getTableStart() > tableStart);
    tableStart = grouper.getTableStart();
    // grow it again
    fillKeyspace(keySpace, maxVectorSize, 6);
    result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertTrue(grouper.getTableStart() > tableStart);
    // this time should be all the way
    fillKeyspace(keySpace, maxVectorSize, 14);
    result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertEquals(0, grouper.getTableStart());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static AggregatorAdapters createMockAggregatorAdapters(int spaceNeededReturn) {
    AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(spaceNeededReturn);
    return aggregatorAdapters;
}
```
</details>

---
#### Test Case ID #druid_Test_35_4
#### Test Case Name: `testGrowFourTimes`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\HashVectorGrouperTest.java`)
#### Mock Object Variable Name: `aggregatorAdapters`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final int keySize = 4;
    final int aggSize = 8;
    final WritableMemory keySpace = WritableMemory.allocate(keySize * maxVectorSize);
-    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
-    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(aggSize);
+    final AggregatorAdapters aggregatorAdapters = createMockAggregatorAdapters(aggSize);
    int startingNumBuckets = 4;
    int maxBuckets = 128;
    final int bufferSize = (keySize + aggSize) * maxBuckets;
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testGrowFourTimes() {
    final int maxVectorSize = 512;
    final int keySize = 4;
    final int aggSize = 8;
    final WritableMemory keySpace = WritableMemory.allocate(keySize * maxVectorSize);
    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(aggSize);
    int startingNumBuckets = 4;
    int maxBuckets = 128;
    final int bufferSize = (keySize + aggSize) * maxBuckets;
    final ByteBuffer buffer = ByteBuffer.wrap(new byte[bufferSize]);
    final HashVectorGrouper grouper = new HashVectorGrouper(Suppliers.ofInstance(buffer), keySize, aggregatorAdapters, maxBuckets, 0.f, startingNumBuckets);
    grouper.initVectorized(maxVectorSize);
    int tableStart = grouper.getTableStart();
    // two keys should cause buffer to grow
    fillKeyspace(keySpace, maxVectorSize, 2);
    AggregateResult result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertEquals(tableStart, grouper.getTableStart());
    // 3rd key should cause buffer to grow
    // buffer should grow to next size, but is not full
    fillKeyspace(keySpace, maxVectorSize, 3);
    result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertTrue(grouper.getTableStart() > tableStart);
    tableStart = grouper.getTableStart();
    // grow it again
    fillKeyspace(keySpace, maxVectorSize, 6);
    result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertTrue(grouper.getTableStart() > tableStart);
    tableStart = grouper.getTableStart();
    // more
    fillKeyspace(keySpace, maxVectorSize, 14);
    result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertTrue(grouper.getTableStart() > tableStart);
    // this time should be all the way
    fillKeyspace(keySpace, maxVectorSize, 25);
    result = grouper.aggregateVector(keySpace, 0, maxVectorSize);
    Assert.assertTrue(result.isOk());
    Assert.assertEquals(0, grouper.getTableStart());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static AggregatorAdapters createMockAggregatorAdapters(int spaceNeededReturn) {
    AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
    Mockito.when(aggregatorAdapters.spaceNeeded()).thenReturn(spaceNeededReturn);
    return aggregatorAdapters;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_36
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.query.aggregation.AggregatorAdapters`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
// === Declare in class scope ===
private AggregatorAdapters aggregatorAdapters;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
}

// === Replace local variable in test with ===
aggregatorAdapters;

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_36_1
#### Test Case Name: `testCloseAggregatorAdaptorsShouldBeClosed`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\HashVectorGrouperTest.java`)
#### Mock Object Variable Name: `aggregatorAdapters`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testCloseAggregatorAdaptorsShouldBeClosed() {
     final ByteBuffer buffer = ByteBuffer.wrap(new byte[4096]);
-    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
+    // removed local mock; replaced with global field `aggregatorAdapters`
     final HashVectorGrouper grouper = new HashVectorGrouper(Suppliers.ofInstance(buffer), 1024, aggregatorAdapters, Integer.MAX_VALUE, 0.f, 0);
     grouper.initVectorized(512);
     grouper.close();
-    Mockito.verify(aggregatorAdapters, Mockito.times(1)).close();
+    Mockito.verify(aggregatorAdapters, Mockito.times(1)).close();
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testCloseAggregatorAdaptorsShouldBeClosed() {
    final ByteBuffer buffer = ByteBuffer.wrap(new byte[4096]);
    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
    final HashVectorGrouper grouper = new HashVectorGrouper(Suppliers.ofInstance(buffer), 1024, aggregatorAdapters, Integer.MAX_VALUE, 0.f, 0);
    grouper.initVectorized(512);
    grouper.close();
    Mockito.verify(aggregatorAdapters, Mockito.times(1)).close();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AggregatorAdapters aggregatorAdapters;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
}

// === Replace local variable in test with ===
aggregatorAdapters;

```
</details>

---
#### Test Case ID #druid_Test_36_2
#### Test Case Name: `testTableStartIsNotMemoryStartIfNotMaxSized`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\HashVectorGrouperTest.java`)
#### Mock Object Variable Name: `aggregatorAdapters`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testTableStartIsNotMemoryStartIfNotMaxSized() {
     final int maxVectorSize = 512;
     final int keySize = 4;
     final int bufferSize = 100 * 1024;
     final WritableMemory keySpace = WritableMemory.allocate(keySize * maxVectorSize);
     final ByteBuffer buffer = ByteBuffer.wrap(new byte[bufferSize]);
-    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
+    // removed local mock; replaced with global field `aggregatorAdapters`
     final HashVectorGrouper grouper = new HashVectorGrouper(Suppliers.ofInstance(buffer), keySize, aggregatorAdapters, 8, 0.f, 4);
     grouper.initVectorized(maxVectorSize);
     Assert.assertNotEquals(0, grouper.getTableStart());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testTableStartIsNotMemoryStartIfNotMaxSized() {
    final int maxVectorSize = 512;
    final int keySize = 4;
    final int bufferSize = 100 * 1024;
    final WritableMemory keySpace = WritableMemory.allocate(keySize * maxVectorSize);
    final ByteBuffer buffer = ByteBuffer.wrap(new byte[bufferSize]);
    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
    final HashVectorGrouper grouper = new HashVectorGrouper(Suppliers.ofInstance(buffer), keySize, aggregatorAdapters, 8, 0.f, 4);
    grouper.initVectorized(maxVectorSize);
    Assert.assertNotEquals(0, grouper.getTableStart());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AggregatorAdapters aggregatorAdapters;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
}

// === Replace local variable in test with ===
aggregatorAdapters;

```
</details>

---
#### Test Case ID #druid_Test_36_3
#### Test Case Name: `testTableStartIsNotMemoryStartIfIsMaxSized`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\HashVectorGrouperTest.java`)
#### Mock Object Variable Name: `aggregatorAdapters`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testTableStartIsNotMemoryStartIfIsMaxSized() {
     final int maxVectorSize = 512;
     final int keySize = 10000;
     final int bufferSize = 100 * 1024;
     final ByteBuffer buffer = ByteBuffer.wrap(new byte[bufferSize]);
-    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
+    // removed local mock; replaced with global field `aggregatorAdapters`
     final HashVectorGrouper grouper = new HashVectorGrouper(Suppliers.ofInstance(buffer), keySize, aggregatorAdapters, 4, 0.f, 4);
     grouper.initVectorized(maxVectorSize);
     Assert.assertEquals(0, grouper.getTableStart());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testTableStartIsNotMemoryStartIfIsMaxSized() {
    final int maxVectorSize = 512;
    final int keySize = 10000;
    final int bufferSize = 100 * 1024;
    final ByteBuffer buffer = ByteBuffer.wrap(new byte[bufferSize]);
    final AggregatorAdapters aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
    final HashVectorGrouper grouper = new HashVectorGrouper(Suppliers.ofInstance(buffer), keySize, aggregatorAdapters, 4, 0.f, 4);
    grouper.initVectorized(maxVectorSize);
    Assert.assertEquals(0, grouper.getTableStart());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AggregatorAdapters aggregatorAdapters;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    aggregatorAdapters = Mockito.mock(AggregatorAdapters.class);
}

// === Replace local variable in test with ===
aggregatorAdapters;

```
</details>

---
## Mock Clone Instance #druid_MCI_37
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.java.util.metrics.DruidMonitorSchedulerConfig`
- **Test Case Count**: 4
- **MO Count**: 4

### Reusable Method
```java
private static DruidMonitorSchedulerConfig createMockDruidMonitorSchedulerConfig(org.joda.time.Duration emissionDuration) {
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(emissionDuration);
    return config;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_37_1
#### Test Case Name: `testStart_RepeatScheduling`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\ClockDriftSafeMonitorSchedulerTest.java`)
#### Mock Object Variable Name: `config`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    Monitor monitor = Mockito.mock(Monitor.class);
-    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
-    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
+    DruidMonitorSchedulerConfig config = createMockDruidMonitorSchedulerConfig(new org.joda.time.Duration(1000L));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStart_RepeatScheduling() throws InterruptedException {
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    CountDownLatch latch = new CountDownLatch(1);
    Mockito.doAnswer(new Answer<Future<?>>() {

        private int scheduleCount = 0;

        @SuppressWarnings("unchecked")
        @Override
        public Future<?> answer(InvocationOnMock invocation) {
            final Object originalArgument = (invocation.getArguments())[3];
            CronTask task = ((CronTask) originalArgument);
            Mockito.doAnswer(new Answer<Future<?>>() {

                @Override
                public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                    final Object originalArgument = (invocation.getArguments())[0];
                    ((Callable<Boolean>) originalArgument).call();
                    return CompletableFuture.completedFuture(Boolean.TRUE);
                }
            }).when(executor).submit(ArgumentMatchers.any(Callable.class));
            cronTaskRunner.submit(() -> {
                while (scheduleCount < 2) {
                    scheduleCount++;
                    task.run(123L);
                }
                latch.countDown();
                return null;
            });
            return createDummyFuture();
        }
    }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Monitor monitor = Mockito.mock(Monitor.class);
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
    latch.await(5, TimeUnit.SECONDS);
    Mockito.verify(monitor, Mockito.times(1)).start();
    Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Mockito.verify(executor, Mockito.times(2)).submit(ArgumentMatchers.any(Callable.class));
    Mockito.verify(monitor, Mockito.times(2)).monitor(ArgumentMatchers.any());
    scheduler.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static DruidMonitorSchedulerConfig createMockDruidMonitorSchedulerConfig(org.joda.time.Duration emissionDuration) {
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(emissionDuration);
    return config;
}
```
</details>

---
#### Test Case ID #druid_Test_37_2
#### Test Case Name: `testStart_RepeatAndStopScheduling`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\ClockDriftSafeMonitorSchedulerTest.java`)
#### Mock Object Variable Name: `config`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    Monitor monitor = Mockito.mock(Monitor.class);
-    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
-    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
+    DruidMonitorSchedulerConfig config = createMockDruidMonitorSchedulerConfig(new org.joda.time.Duration(1000L));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
    latch.await(5, TimeUnit.SECONDS);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStart_RepeatAndStopScheduling() throws InterruptedException {
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    CountDownLatch latch = new CountDownLatch(1);
    Mockito.doAnswer(new Answer<Future<?>>() {

        private int scheduleCount = 0;

        @SuppressWarnings("unchecked")
        @Override
        public Future<?> answer(InvocationOnMock invocation) {
            final Object originalArgument = (invocation.getArguments())[3];
            CronTask task = ((CronTask) originalArgument);
            Mockito.doAnswer(new Answer<Future<?>>() {

                @Override
                public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                    final Object originalArgument = (invocation.getArguments())[0];
                    ((Callable<Boolean>) originalArgument).call();
                    return CompletableFuture.completedFuture(Boolean.FALSE);
                }
            }).when(executor).submit(ArgumentMatchers.any(Callable.class));
            cronTaskRunner.submit(() -> {
                while (scheduleCount < 2) {
                    scheduleCount++;
                    task.run(123L);
                }
                latch.countDown();
                return null;
            });
            return createDummyFuture();
        }
    }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Monitor monitor = Mockito.mock(Monitor.class);
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
    latch.await(5, TimeUnit.SECONDS);
    Mockito.verify(monitor, Mockito.times(1)).start();
    Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
    Mockito.verify(monitor, Mockito.times(2)).monitor(ArgumentMatchers.any());
    Mockito.verify(monitor, Mockito.times(1)).stop();
    scheduler.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static DruidMonitorSchedulerConfig createMockDruidMonitorSchedulerConfig(org.joda.time.Duration emissionDuration) {
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(emissionDuration);
    return config;
}
```
</details>

---
#### Test Case ID #druid_Test_37_3
#### Test Case Name: `testStart_UnexpectedExceptionWhileMonitoring`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\ClockDriftSafeMonitorSchedulerTest.java`)
#### Mock Object Variable Name: `config`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    Monitor monitor = Mockito.mock(Monitor.class);
    Mockito.when(monitor.monitor(ArgumentMatchers.any(ServiceEmitter.class))).thenThrow(new RuntimeException("Test throwing exception while monitoring"));
-    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
-    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
+    DruidMonitorSchedulerConfig config = createMockDruidMonitorSchedulerConfig(new org.joda.time.Duration(1000L));
    CountDownLatch latch = new CountDownLatch(1);
    AtomicBoolean monitorResultHolder = new AtomicBoolean(false);
    Mockito.doAnswer(new Answer<Future<?>>() {
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStart_UnexpectedExceptionWhileMonitoring() throws InterruptedException {
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    Monitor monitor = Mockito.mock(Monitor.class);
    Mockito.when(monitor.monitor(ArgumentMatchers.any(ServiceEmitter.class))).thenThrow(new RuntimeException("Test throwing exception while monitoring"));
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
    CountDownLatch latch = new CountDownLatch(1);
    AtomicBoolean monitorResultHolder = new AtomicBoolean(false);
    Mockito.doAnswer(new Answer<Future<?>>() {

        @SuppressWarnings("unchecked")
        @Override
        public Future<?> answer(InvocationOnMock invocation) {
            final Object originalArgument = (invocation.getArguments())[3];
            CronTask task = ((CronTask) originalArgument);
            Mockito.doAnswer(new Answer<Future<?>>() {

                @Override
                public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                    final Object originalArgument = (invocation.getArguments())[0];
                    final boolean continueMonitor = ((Callable<Boolean>) originalArgument).call();
                    monitorResultHolder.set(continueMonitor);
                    return CompletableFuture.completedFuture(continueMonitor);
                }
            }).when(executor).submit(ArgumentMatchers.any(Callable.class));
            cronTaskRunner.submit(() -> {
                task.run(123L);
                latch.countDown();
                return null;
            });
            return createDummyFuture();
        }
    }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
    latch.await(5, TimeUnit.SECONDS);
    Mockito.verify(monitor, Mockito.times(1)).start();
    Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
    Mockito.verify(monitor, Mockito.times(1)).monitor(ArgumentMatchers.any());
    Assert.assertTrue(monitorResultHolder.get());
    scheduler.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static DruidMonitorSchedulerConfig createMockDruidMonitorSchedulerConfig(org.joda.time.Duration emissionDuration) {
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(emissionDuration);
    return config;
}
```
</details>

---
#### Test Case ID #druid_Test_37_4
#### Test Case Name: `testStart_UnexpectedExceptionWhileScheduling`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\ClockDriftSafeMonitorSchedulerTest.java`)
#### Mock Object Variable Name: `config`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    Monitor monitor = Mockito.mock(Monitor.class);
-    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
-    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
+    DruidMonitorSchedulerConfig config = createMockDruidMonitorSchedulerConfig(new org.joda.time.Duration(1000L));
    CountDownLatch latch = new CountDownLatch(1);
    Mockito.doAnswer(new Answer<Future<?>>() {

```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStart_UnexpectedExceptionWhileScheduling() throws InterruptedException {
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    Monitor monitor = Mockito.mock(Monitor.class);
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
    CountDownLatch latch = new CountDownLatch(1);
    Mockito.doAnswer(new Answer<Future<?>>() {

        @SuppressWarnings("unchecked")
        @Override
        public Future<?> answer(InvocationOnMock invocation) {
            final Object originalArgument = (invocation.getArguments())[3];
            CronTask task = ((CronTask) originalArgument);
            Mockito.when(executor.submit(ArgumentMatchers.any(Callable.class))).thenThrow(new RuntimeException("Test throwing exception while scheduling"));
            cronTaskRunner.submit(() -> {
                task.run(123L);
                latch.countDown();
                return null;
            });
            return createDummyFuture();
        }
    }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
    latch.await(5, TimeUnit.SECONDS);
    Mockito.verify(monitor, Mockito.times(1)).start();
    Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
    scheduler.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static DruidMonitorSchedulerConfig createMockDruidMonitorSchedulerConfig(org.joda.time.Duration emissionDuration) {
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(emissionDuration);
    return config;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_38
- **Scope**: method level
- **Mocked Class**: `com.google.inject.Injector`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
private static Injector createMockInjector(ObjectMapper objectMapper, Key<ObjectMapper> objectMapperKey) {
    Injector injector = Mockito.mock(Injector.class);
    Mockito.when(injector.getInstance(objectMapperKey)).thenReturn(objectMapper);
    return injector;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_38_1
#### Test Case Name: `testDumpBitmap`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\cli\DumpSegmentTest.java`)
#### Mock Object Variable Name: `injector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testDumpBitmap() throws IOException {
-    Injector injector = Mockito.mock(Injector.class);
     QueryableIndex queryableIndex = Mockito.mock(QueryableIndex.class);
     ObjectMapper mapper = new DefaultObjectMapper();
     BitmapFactory bitmapFactory = new RoaringBitmapFactory();
     ColumnHolder xHolder = Mockito.mock(ColumnHolder.class);
     ColumnHolder yHolder = Mockito.mock(ColumnHolder.class);
     ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
     DictionaryEncodedStringValueIndex valueIndex = Mockito.mock(DictionaryEncodedStringValueIndex.class);
     ImmutableBitmap bitmap = bitmapFactory.complement(bitmapFactory.makeEmptyImmutableBitmap(), 10);
+    Injector injector = createMockInjector(mapper, Key.get(ObjectMapper.class, Json.class));
     Mockito.when(queryableIndex.getBitmapFactoryForDimensions()).thenReturn(bitmapFactory);
     Mockito.when(queryableIndex.getColumnHolder("x")).thenReturn(xHolder);
     Mockito.when(queryableIndex.getColumnHolder("y")).thenReturn(yHolder);
     Mockito.when(xHolder.getIndexSupplier()).thenReturn(indexSupplier);
     Mockito.when(indexSupplier.as(DictionaryEncodedStringValueIndex.class)).thenReturn(valueIndex);
     Mockito.when(valueIndex.getCardinality()).thenReturn(1);
     Mockito.when(valueIndex.getBitmap(0)).thenReturn(bitmap);
     Mockito.when(valueIndex.getValue(0)).thenReturn("val");
     DumpSegment.runBitmaps(injector, tempFolder.newFile().getPath(), queryableIndex, ImmutableList.of("x", "y"), false);
     Assert.assertTrue(true);
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testDumpBitmap() throws IOException {
    Injector injector = Mockito.mock(Injector.class);
    QueryableIndex queryableIndex = Mockito.mock(QueryableIndex.class);
    ObjectMapper mapper = new DefaultObjectMapper();
    BitmapFactory bitmapFactory = new RoaringBitmapFactory();
    ColumnHolder xHolder = Mockito.mock(ColumnHolder.class);
    ColumnHolder yHolder = Mockito.mock(ColumnHolder.class);
    ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    DictionaryEncodedStringValueIndex valueIndex = Mockito.mock(DictionaryEncodedStringValueIndex.class);
    ImmutableBitmap bitmap = bitmapFactory.complement(bitmapFactory.makeEmptyImmutableBitmap(), 10);
    Mockito.when(injector.getInstance(Key.get(ObjectMapper.class, Json.class))).thenReturn(mapper);
    Mockito.when(queryableIndex.getBitmapFactoryForDimensions()).thenReturn(bitmapFactory);
    Mockito.when(queryableIndex.getColumnHolder("x")).thenReturn(xHolder);
    Mockito.when(queryableIndex.getColumnHolder("y")).thenReturn(yHolder);
    Mockito.when(xHolder.getIndexSupplier()).thenReturn(indexSupplier);
    Mockito.when(indexSupplier.as(DictionaryEncodedStringValueIndex.class)).thenReturn(valueIndex);
    Mockito.when(valueIndex.getCardinality()).thenReturn(1);
    Mockito.when(valueIndex.getBitmap(0)).thenReturn(bitmap);
    Mockito.when(valueIndex.getValue(0)).thenReturn("val");
    DumpSegment.runBitmaps(injector, tempFolder.newFile().getPath(), queryableIndex, ImmutableList.of("x", "y"), false);
    Assert.assertTrue(true);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static Injector createMockInjector(ObjectMapper objectMapper, Key<ObjectMapper> objectMapperKey) {
    Injector injector = Mockito.mock(Injector.class);
    Mockito.when(injector.getInstance(objectMapperKey)).thenReturn(objectMapper);
    return injector;
}
```
</details>

---
#### Test Case ID #druid_Test_38_2
#### Test Case Name: `testDumpNestedColumn`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\cli\DumpSegmentTest.java`)
#### Mock Object Variable Name: `injector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testDumpNestedColumn() throws Exception {
-    Injector injector = Mockito.mock(Injector.class);
     ObjectMapper mapper = TestHelper.makeJsonMapper();
     mapper.registerModules(NestedDataModule.getJacksonModulesList());
     mapper.setInjectableValues(new InjectableValues.Std().addValue(ExprMacroTable.class.getName(), TestExprMacroTable.INSTANCE).addValue(ObjectMapper.class.getName(), mapper).addValue(DefaultColumnFormatConfig.class, new DefaultColumnFormatConfig(null)));
+    Key<ObjectMapper> objectMapperKey = Key.get(ObjectMapper.class, Json.class);
+    Injector injector = createMockInjector(mapper, objectMapperKey);
     Mockito.when(injector.getInstance(DefaultColumnFormatConfig.class)).thenReturn(new DefaultColumnFormatConfig(null));
     List<Segment> segments = createSegments(tempFolder, closer);
     QueryableIndex queryableIndex = segments.get(0).asQueryableIndex();
     File outputFile = tempFolder.newFile();
     DumpSegment.runDumpNestedColumn(injector, outputFile.getPath(), queryableIndex, "nest");
     final byte[] fileBytes = Files.readAllBytes(outputFile.toPath());
     final String output = StringUtils.fromUtf8(fileBytes);
     if (NullHandling.sqlCompatible()) {
         Assert.assertEquals("{\"nest\":{\"fields\":[{\"path\":\"$.x\",\"types\":[\"LONG\"]},{\"path\":\"$.y\",\"types\":[\"DOUBLE\"]},{\"path\":\"$.z\",\"types\":[\"STRING\"]}],\"dictionaries\":{\"strings\":[{\"globalId\":0,\"value\":null},{\"globalId\":1,\"value\":\"a\"},{\"globalId\":2,\"value\":\"b\"}],\"longs\":[{\"globalId\":3,\"value\":100},{\"globalId\":4,\"value\":200},{\"globalId\":5,\"value\":400}],\"doubles\":[{\"globalId\":6,\"value\":1.1},{\"globalId\":7,\"value\":2.2},{\"globalId\":8,\"value\":3.3}],\"nullRows\":[]}}}", output);
     } else {
         Assert.assertEquals("{\"nest\":{\"fields\":[{\"path\":\"$.x\",\"types\":[\"LONG\"]},{\"path\":\"$.y\",\"types\":[\"DOUBLE\"]},{\"path\":\"$.z\",\"types\":[\"STRING\"]}],\"dictionaries\":{\"strings\":[{\"globalId\":0,\"value\":null},{\"globalId\":1,\"value\":\"a\"},{\"globalId\":2,\"value\":\"b\"}],\"longs\":[{\"globalId\":3,\"value\":0},{\"globalId\":4,\"value\":100},{\"globalId\":5,\"value\":200},{\"globalId\":6,\"value\":400}],\"doubles\":[{\"globalId\":7,\"value\":0.0},{\"globalId\":8,\"value\":1.1},{\"globalId\":9,\"value\":2.2},{\"globalId\":10,\"value\":3.3}],\"nullRows\":[]}}}", output);
     }
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testDumpNestedColumn() throws Exception {
    Injector injector = Mockito.mock(Injector.class);
    ObjectMapper mapper = TestHelper.makeJsonMapper();
    mapper.registerModules(NestedDataModule.getJacksonModulesList());
    mapper.setInjectableValues(new InjectableValues.Std().addValue(ExprMacroTable.class.getName(), TestExprMacroTable.INSTANCE).addValue(ObjectMapper.class.getName(), mapper).addValue(DefaultColumnFormatConfig.class, new DefaultColumnFormatConfig(null)));
    Mockito.when(injector.getInstance(Key.get(ObjectMapper.class, Json.class))).thenReturn(mapper);
    Mockito.when(injector.getInstance(DefaultColumnFormatConfig.class)).thenReturn(new DefaultColumnFormatConfig(null));
    List<Segment> segments = createSegments(tempFolder, closer);
    QueryableIndex queryableIndex = segments.get(0).asQueryableIndex();
    File outputFile = tempFolder.newFile();
    DumpSegment.runDumpNestedColumn(injector, outputFile.getPath(), queryableIndex, "nest");
    final byte[] fileBytes = Files.readAllBytes(outputFile.toPath());
    final String output = StringUtils.fromUtf8(fileBytes);
    if (NullHandling.sqlCompatible()) {
        Assert.assertEquals("{\"nest\":{\"fields\":[{\"path\":\"$.x\",\"types\":[\"LONG\"]},{\"path\":\"$.y\",\"types\":[\"DOUBLE\"]},{\"path\":\"$.z\",\"types\":[\"STRING\"]}],\"dictionaries\":{\"strings\":[{\"globalId\":0,\"value\":null},{\"globalId\":1,\"value\":\"a\"},{\"globalId\":2,\"value\":\"b\"}],\"longs\":[{\"globalId\":3,\"value\":100},{\"globalId\":4,\"value\":200},{\"globalId\":5,\"value\":400}],\"doubles\":[{\"globalId\":6,\"value\":1.1},{\"globalId\":7,\"value\":2.2},{\"globalId\":8,\"value\":3.3}],\"nullRows\":[]}}}", output);
    } else {
        Assert.assertEquals("{\"nest\":{\"fields\":[{\"path\":\"$.x\",\"types\":[\"LONG\"]},{\"path\":\"$.y\",\"types\":[\"DOUBLE\"]},{\"path\":\"$.z\",\"types\":[\"STRING\"]}],\"dictionaries\":{\"strings\":[{\"globalId\":0,\"value\":null},{\"globalId\":1,\"value\":\"a\"},{\"globalId\":2,\"value\":\"b\"}],\"longs\":[{\"globalId\":3,\"value\":0},{\"globalId\":4,\"value\":100},{\"globalId\":5,\"value\":200},{\"globalId\":6,\"value\":400}],\"doubles\":[{\"globalId\":7,\"value\":0.0},{\"globalId\":8,\"value\":1.1},{\"globalId\":9,\"value\":2.2},{\"globalId\":10,\"value\":3.3}],\"nullRows\":[]}}}", output);
    }
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static Injector createMockInjector(ObjectMapper objectMapper, Key<ObjectMapper> objectMapperKey) {
    Injector injector = Mockito.mock(Injector.class);
    Mockito.when(injector.getInstance(objectMapperKey)).thenReturn(objectMapper);
    return injector;
}
```
</details>

---
#### Test Case ID #druid_Test_38_3
#### Test Case Name: `testDumpNestedColumnPath`(File: `C:\Java_projects\Apache\druid\services\src\test\java\org\apache\druid\cli\DumpSegmentTest.java`)
#### Mock Object Variable Name: `injector`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testDumpNestedColumnPath() throws Exception {
-    Injector injector = Mockito.mock(Injector.class);
     ObjectMapper mapper = TestHelper.makeJsonMapper();
     mapper.registerModules(NestedDataModule.getJacksonModulesList());
     mapper.setInjectableValues(new InjectableValues.Std().addValue(ExprMacroTable.class.getName(), TestExprMacroTable.INSTANCE).addValue(ObjectMapper.class.getName(), mapper).addValue(DefaultColumnFormatConfig.class, new DefaultColumnFormatConfig(null)));
+    Injector injector = createMockInjector(mapper, Key.get(ObjectMapper.class, Json.class));
     Mockito.when(injector.getInstance(DefaultColumnFormatConfig.class)).thenReturn(new DefaultColumnFormatConfig(null));
     List<Segment> segments = createSegments(tempFolder, closer);
     QueryableIndex queryableIndex = segments.get(0).asQueryableIndex();
     File outputFile = tempFolder.newFile();
     DumpSegment.runDumpNestedColumnPath(injector, outputFile.getPath(), queryableIndex, "nest", "$.x");
     final byte[] fileBytes = Files.readAllBytes(outputFile.toPath());
     final String output = StringUtils.fromUtf8(fileBytes);
     if (NullHandling.sqlCompatible()) {
         Assert.assertEquals("{\"bitmapSerdeFactory\":{\"type\":\"roaring\"},\"nest\":{\"$.x\":{\"types\":[\"LONG\"],\"dictionary\":[{\"localId\":0,\"globalId\":0,\"value\":null,\"rows\":[4]},{\"localId\":1,\"globalId\":3,\"value\":\"100\",\"rows\":[3]},{\"localId\":2,\"globalId\":4,\"value\":\"200\",\"rows\":[0,2]},{\"localId\":3,\"globalId\":5,\"value\":\"400\",\"rows\":[1]}],\"column\":[{\"row\":0,\"raw\":{\"x\":200,\"y\":2.2},\"fieldId\":2,\"fieldValue\":\"200\"},{\"row\":1,\"raw\":{\"x\":400,\"y\":1.1,\"z\":\"a\"},\"fieldId\":3,\"fieldValue\":\"400\"},{\"row\":2,\"raw\":{\"x\":200,\"z\":\"b\"},\"fieldId\":2,\"fieldValue\":\"200\"},{\"row\":3,\"raw\":{\"x\":100,\"y\":1.1,\"z\":\"a\"},\"fieldId\":1,\"fieldValue\":\"100\"},{\"row\":4,\"raw\":{\"y\":3.3,\"z\":\"b\"},\"fieldId\":0,\"fieldValue\":null}]}}}", output);
     } else {
         Assert.assertEquals("{\"bitmapSerdeFactory\":{\"type\":\"roaring\"},\"nest\":{\"$.x\":{\"types\":[\"LONG\"],\"dictionary\":[{\"localId\":0,\"globalId\":0,\"value\":null,\"rows\":[4]},{\"localId\":1,\"globalId\":4,\"value\":\"100\",\"rows\":[3]},{\"localId\":2,\"globalId\":5,\"value\":\"200\",\"rows\":[0,2]},{\"localId\":3,\"globalId\":6,\"value\":\"400\",\"rows\":[1]}],\"column\":[{\"row\":0,\"raw\":{\"x\":200,\"y\":2.2},\"fieldId\":2,\"fieldValue\":\"200\"},{\"row\":1,\"raw\":{\"x\":400,\"y\":1.1,\"z\":\"a\"},\"fieldId\":3,\"fieldValue\":\"400\"},{\"row\":2,\"raw\":{\"x\":200,\"z\":\"b\"},\"fieldId\":2,\"fieldValue\":\"200\"},{\"row\":3,\"raw\":{\"x\":100,\"y\":1.1,\"z\":\"a\"},\"fieldId\":1,\"fieldValue\":\"100\"},{\"row\":4,\"raw\":{\"y\":3.3,\"z\":\"b\"},\"fieldId\":0,\"fieldValue\":null}]}}}", output);
     }
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testDumpNestedColumnPath() throws Exception {
    Injector injector = Mockito.mock(Injector.class);
    ObjectMapper mapper = TestHelper.makeJsonMapper();
    mapper.registerModules(NestedDataModule.getJacksonModulesList());
    mapper.setInjectableValues(new InjectableValues.Std().addValue(ExprMacroTable.class.getName(), TestExprMacroTable.INSTANCE).addValue(ObjectMapper.class.getName(), mapper).addValue(DefaultColumnFormatConfig.class, new DefaultColumnFormatConfig(null)));
    Mockito.when(injector.getInstance(Key.get(ObjectMapper.class, Json.class))).thenReturn(mapper);
    Mockito.when(injector.getInstance(DefaultColumnFormatConfig.class)).thenReturn(new DefaultColumnFormatConfig(null));
    List<Segment> segments = createSegments(tempFolder, closer);
    QueryableIndex queryableIndex = segments.get(0).asQueryableIndex();
    File outputFile = tempFolder.newFile();
    DumpSegment.runDumpNestedColumnPath(injector, outputFile.getPath(), queryableIndex, "nest", "$.x");
    final byte[] fileBytes = Files.readAllBytes(outputFile.toPath());
    final String output = StringUtils.fromUtf8(fileBytes);
    if (NullHandling.sqlCompatible()) {
        Assert.assertEquals("{\"bitmapSerdeFactory\":{\"type\":\"roaring\"},\"nest\":{\"$.x\":{\"types\":[\"LONG\"],\"dictionary\":[{\"localId\":0,\"globalId\":0,\"value\":null,\"rows\":[4]},{\"localId\":1,\"globalId\":3,\"value\":\"100\",\"rows\":[3]},{\"localId\":2,\"globalId\":4,\"value\":\"200\",\"rows\":[0,2]},{\"localId\":3,\"globalId\":5,\"value\":\"400\",\"rows\":[1]}],\"column\":[{\"row\":0,\"raw\":{\"x\":200,\"y\":2.2},\"fieldId\":2,\"fieldValue\":\"200\"},{\"row\":1,\"raw\":{\"x\":400,\"y\":1.1,\"z\":\"a\"},\"fieldId\":3,\"fieldValue\":\"400\"},{\"row\":2,\"raw\":{\"x\":200,\"z\":\"b\"},\"fieldId\":2,\"fieldValue\":\"200\"},{\"row\":3,\"raw\":{\"x\":100,\"y\":1.1,\"z\":\"a\"},\"fieldId\":1,\"fieldValue\":\"100\"},{\"row\":4,\"raw\":{\"y\":3.3,\"z\":\"b\"},\"fieldId\":0,\"fieldValue\":null}]}}}", output);
    } else {
        Assert.assertEquals("{\"bitmapSerdeFactory\":{\"type\":\"roaring\"},\"nest\":{\"$.x\":{\"types\":[\"LONG\"],\"dictionary\":[{\"localId\":0,\"globalId\":0,\"value\":null,\"rows\":[4]},{\"localId\":1,\"globalId\":4,\"value\":\"100\",\"rows\":[3]},{\"localId\":2,\"globalId\":5,\"value\":\"200\",\"rows\":[0,2]},{\"localId\":3,\"globalId\":6,\"value\":\"400\",\"rows\":[1]}],\"column\":[{\"row\":0,\"raw\":{\"x\":200,\"y\":2.2},\"fieldId\":2,\"fieldValue\":\"200\"},{\"row\":1,\"raw\":{\"x\":400,\"y\":1.1,\"z\":\"a\"},\"fieldId\":3,\"fieldValue\":\"400\"},{\"row\":2,\"raw\":{\"x\":200,\"z\":\"b\"},\"fieldId\":2,\"fieldValue\":\"200\"},{\"row\":3,\"raw\":{\"x\":100,\"y\":1.1,\"z\":\"a\"},\"fieldId\":1,\"fieldValue\":\"100\"},{\"row\":4,\"raw\":{\"y\":3.3,\"z\":\"b\"},\"fieldId\":0,\"fieldValue\":null}]}}}", output);
    }
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static Injector createMockInjector(ObjectMapper objectMapper, Key<ObjectMapper> objectMapperKey) {
    Injector injector = Mockito.mock(Injector.class);
    Mockito.when(injector.getInstance(objectMapperKey)).thenReturn(objectMapper);
    return injector;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_39
- **Scope**: class level
- **Mocked Class**: `org.apache.druid.segment.column.ColumnIndexSupplier`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
public class MockColumnIndexSupplier {
  public static ColumnIndexSupplier createMockColumnIndexSupplier(StringValueSetIndexes valueIndex) {
    ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    Mockito.when(indexSupplier.as(StringValueSetIndexes.class)).thenReturn(valueIndex);
    return indexSupplier;
  }
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_39_1
#### Test Case Name: `testUsesStringSetIndex`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\filter\InDimFilterTest.java`)
#### Mock Object Variable Name: `indexSupplier`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final Filter inFilter = new InDimFilter("dim0", ImmutableSet.of("v1", "v2")).toFilter();
    final ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
-    final ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    final StringValueSetIndexes valueIndex = Mockito.mock(StringValueSetIndexes.class);
    final BitmapColumnIndex bitmapColumnIndex = Mockito.mock(BitmapColumnIndex.class);
    final InDimFilter.ValuesSet expectedValuesSet = new InDimFilter.ValuesSet();
    expectedValuesSet.addAll(Arrays.asList("v1", "v2"));
+    final ColumnIndexSupplier indexSupplier = MockColumnIndexSupplier.createMockColumnIndexSupplier(valueIndex);
    Mockito.when(indexSelector.getIndexSupplier("dim0")).thenReturn(indexSupplier);
-    // Will check for UTF-8 first.
-    Mockito.when(indexSupplier.as(Utf8ValueSetIndexes.class)).thenReturn(null);
    Mockito.when(valueIndex.forSortedValues(expectedValuesSet)).thenReturn(bitmapColumnIndex);
    final BitmapColumnIndex retVal = inFilter.getBitmapColumnIndex(indexSelector);
    Assert.assertSame("inFilter returns the intended bitmapColumnIndex", bitmapColumnIndex, retVal);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testUsesStringSetIndex() {
    // An implementation test.
    // This test confirms that "in" filters use non-utf8 string index lookups when utf8 indexes are not available.
    final Filter inFilter = new InDimFilter("dim0", ImmutableSet.of("v1", "v2")).toFilter();
    final ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
    final ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    final StringValueSetIndexes valueIndex = Mockito.mock(StringValueSetIndexes.class);
    final BitmapColumnIndex bitmapColumnIndex = Mockito.mock(BitmapColumnIndex.class);
    final InDimFilter.ValuesSet expectedValuesSet = new InDimFilter.ValuesSet();
    expectedValuesSet.addAll(Arrays.asList("v1", "v2"));
    Mockito.when(indexSelector.getIndexSupplier("dim0")).thenReturn(indexSupplier);
    // Will check for UTF-8 first.
    Mockito.when(indexSupplier.as(Utf8ValueSetIndexes.class)).thenReturn(null);
    Mockito.when(indexSupplier.as(StringValueSetIndexes.class)).thenReturn(valueIndex);
    Mockito.when(valueIndex.forSortedValues(expectedValuesSet)).thenReturn(bitmapColumnIndex);
    final BitmapColumnIndex retVal = inFilter.getBitmapColumnIndex(indexSelector);
    Assert.assertSame("inFilter returns the intended bitmapColumnIndex", bitmapColumnIndex, retVal);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnIndexSupplier {
  public static ColumnIndexSupplier createMockColumnIndexSupplier(StringValueSetIndexes valueIndex) {
    ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    Mockito.when(indexSupplier.as(StringValueSetIndexes.class)).thenReturn(valueIndex);
    return indexSupplier;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_39_2
#### Test Case Name: `testExactMatchUsesValueIndex`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\filter\LikeDimFilterTest.java`)
#### Mock Object Variable Name: `indexSupplier`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final Filter likeFilter = new LikeDimFilter("dim0", "f", null, null, null).toFilter();
    final ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
-    final ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    final StringValueSetIndexes valueIndex = Mockito.mock(StringValueSetIndexes.class);
    final BitmapColumnIndex bitmapColumnIndex = Mockito.mock(BitmapColumnIndex.class);
+    final ColumnIndexSupplier indexSupplier = MockColumnIndexSupplier.createMockColumnIndexSupplier(valueIndex);
    Mockito.when(indexSelector.getIndexSupplier("dim0")).thenReturn(indexSupplier);
-    Mockito.when(indexSupplier.as(StringValueSetIndexes.class)).thenReturn(valueIndex);
    Mockito.when(valueIndex.forValue("f")).thenReturn(bitmapColumnIndex);
    final BitmapColumnIndex retVal = likeFilter.getBitmapColumnIndex(indexSelector);
    Assert.assertSame("likeFilter returns the intended bitmapColumnIndex", bitmapColumnIndex, retVal);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testExactMatchUsesValueIndex() {
    // An implementation test.
    // This test confirms that "like" filters with exact matchers use index lookups.
    final Filter likeFilter = new LikeDimFilter("dim0", "f", null, null, null).toFilter();
    final ColumnIndexSelector indexSelector = Mockito.mock(ColumnIndexSelector.class);
    final ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    final StringValueSetIndexes valueIndex = Mockito.mock(StringValueSetIndexes.class);
    final BitmapColumnIndex bitmapColumnIndex = Mockito.mock(BitmapColumnIndex.class);
    Mockito.when(indexSelector.getIndexSupplier("dim0")).thenReturn(indexSupplier);
    Mockito.when(indexSupplier.as(StringValueSetIndexes.class)).thenReturn(valueIndex);
    Mockito.when(valueIndex.forValue("f")).thenReturn(bitmapColumnIndex);
    final BitmapColumnIndex retVal = likeFilter.getBitmapColumnIndex(indexSelector);
    Assert.assertSame("likeFilter returns the intended bitmapColumnIndex", bitmapColumnIndex, retVal);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnIndexSupplier {
  public static ColumnIndexSupplier createMockColumnIndexSupplier(StringValueSetIndexes valueIndex) {
    ColumnIndexSupplier indexSupplier = Mockito.mock(ColumnIndexSupplier.class);
    Mockito.when(indexSupplier.as(StringValueSetIndexes.class)).thenReturn(valueIndex);
    return indexSupplier;
  }
}
```
</details>

---
## Mock Clone Instance #druid_MCI_40
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.java.util.metrics.Monitor`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
// === Declare in class scope ===
private Monitor monitor;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    monitor = Mockito.mock(Monitor.class);
}

// === Replace local variable in test with ===
monitor;

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_40_1
#### Test Case Name: `testStart_RepeatScheduling`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\ClockDriftSafeMonitorSchedulerTest.java`)
#### Mock Object Variable Name: `monitor`
<summary>Suggested Diff</summary>

```diff
@@
     Mockito.doAnswer(new Answer<Future<?>>() {
 
         private int scheduleCount = 0;
 
         @SuppressWarnings("unchecked")
         @Override
         public Future<?> answer(InvocationOnMock invocation) {
             final Object originalArgument = (invocation.getArguments())[3];
             CronTask task = ((CronTask) originalArgument);
             Mockito.doAnswer(new Answer<Future<?>>() {
 
                 @Override
                 public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                     final Object originalArgument = (invocation.getArguments())[0];
                     ((Callable<Boolean>) originalArgument).call();
                     return CompletableFuture.completedFuture(Boolean.TRUE);
                 }
             }).when(executor).submit(ArgumentMatchers.any(Callable.class));
             cronTaskRunner.submit(() -> {
                 while (scheduleCount < 2) {
                     scheduleCount++;
                     task.run(123L);
                 }
                 latch.countDown();
                 return null;
             });
             return createDummyFuture();
         }
     }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
-    Monitor monitor = Mockito.mock(Monitor.class);
+    // removed local mock; replaced with global field `monitor`
     DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
     Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
     final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
     scheduler.start();
     latch.await(5, TimeUnit.SECONDS);
@@
-    Mockito.verify(monitor, Mockito.times(1)).start();
+    Mockito.verify(monitor, Mockito.times(1)).start();
     Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
     Mockito.verify(executor, Mockito.times(2)).submit(ArgumentMatchers.any(Callable.class));
-    Mockito.verify(monitor, Mockito.times(2)).monitor(ArgumentMatchers.any());
+    Mockito.verify(monitor, Mockito.times(2)).monitor(ArgumentMatchers.any());
     scheduler.stop();
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStart_RepeatScheduling() throws InterruptedException {
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    CountDownLatch latch = new CountDownLatch(1);
    Mockito.doAnswer(new Answer<Future<?>>() {

        private int scheduleCount = 0;

        @SuppressWarnings("unchecked")
        @Override
        public Future<?> answer(InvocationOnMock invocation) {
            final Object originalArgument = (invocation.getArguments())[3];
            CronTask task = ((CronTask) originalArgument);
            Mockito.doAnswer(new Answer<Future<?>>() {

                @Override
                public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                    final Object originalArgument = (invocation.getArguments())[0];
                    ((Callable<Boolean>) originalArgument).call();
                    return CompletableFuture.completedFuture(Boolean.TRUE);
                }
            }).when(executor).submit(ArgumentMatchers.any(Callable.class));
            cronTaskRunner.submit(() -> {
                while (scheduleCount < 2) {
                    scheduleCount++;
                    task.run(123L);
                }
                latch.countDown();
                return null;
            });
            return createDummyFuture();
        }
    }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Monitor monitor = Mockito.mock(Monitor.class);
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
    latch.await(5, TimeUnit.SECONDS);
    Mockito.verify(monitor, Mockito.times(1)).start();
    Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Mockito.verify(executor, Mockito.times(2)).submit(ArgumentMatchers.any(Callable.class));
    Mockito.verify(monitor, Mockito.times(2)).monitor(ArgumentMatchers.any());
    scheduler.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private Monitor monitor;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    monitor = Mockito.mock(Monitor.class);
}

// === Replace local variable in test with ===
monitor;

```
</details>

---
#### Test Case ID #druid_Test_40_2
#### Test Case Name: `testStart_RepeatAndStopScheduling`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\ClockDriftSafeMonitorSchedulerTest.java`)
#### Mock Object Variable Name: `monitor`
<summary>Suggested Diff</summary>

```diff
@@
     Mockito.doAnswer(new Answer<Future<?>>() {
         public Future<?> answer(InvocationOnMock invocation) {
             final Object originalArgument = (invocation.getArguments())[3];
             CronTask task = ((CronTask) originalArgument);
             Mockito.doAnswer(new Answer<Future<?>>() {
                 public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                     final Object originalArgument = (invocation.getArguments())[0];
                     ((Callable<Boolean>) originalArgument).call();
                     return CompletableFuture.completedFuture(Boolean.FALSE);
                 }
             }).when(executor).submit(ArgumentMatchers.any(Callable.class));
             cronTaskRunner.submit(() -> {
                 while (scheduleCount < 2) {
                     scheduleCount++;
                     task.run(123L);
                 }
                 latch.countDown();
                 return null;
             });
             return createDummyFuture();
         }
     }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
-    Monitor monitor = Mockito.mock(Monitor.class);
+    // removed local mock; replaced with global field `monitor`
     DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
     Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
     final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
     scheduler.start();
     latch.await(5, TimeUnit.SECONDS);
@@
-    Mockito.verify(monitor, Mockito.times(1)).start();
+    Mockito.verify(monitor, Mockito.times(1)).start();
     Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
     Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
@@
-    Mockito.verify(monitor, Mockito.times(2)).monitor(ArgumentMatchers.any());
+    Mockito.verify(monitor, Mockito.times(2)).monitor(ArgumentMatchers.any());
-    Mockito.verify(monitor, Mockito.times(1)).stop();
+    Mockito.verify(monitor, Mockito.times(1)).stop();
     scheduler.stop();
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStart_RepeatAndStopScheduling() throws InterruptedException {
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    CountDownLatch latch = new CountDownLatch(1);
    Mockito.doAnswer(new Answer<Future<?>>() {

        private int scheduleCount = 0;

        @SuppressWarnings("unchecked")
        @Override
        public Future<?> answer(InvocationOnMock invocation) {
            final Object originalArgument = (invocation.getArguments())[3];
            CronTask task = ((CronTask) originalArgument);
            Mockito.doAnswer(new Answer<Future<?>>() {

                @Override
                public Future<Boolean> answer(InvocationOnMock invocation) throws Exception {
                    final Object originalArgument = (invocation.getArguments())[0];
                    ((Callable<Boolean>) originalArgument).call();
                    return CompletableFuture.completedFuture(Boolean.FALSE);
                }
            }).when(executor).submit(ArgumentMatchers.any(Callable.class));
            cronTaskRunner.submit(() -> {
                while (scheduleCount < 2) {
                    scheduleCount++;
                    task.run(123L);
                }
                latch.countDown();
                return null;
            });
            return createDummyFuture();
        }
    }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Monitor monitor = Mockito.mock(Monitor.class);
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
    latch.await(5, TimeUnit.SECONDS);
    Mockito.verify(monitor, Mockito.times(1)).start();
    Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
    Mockito.verify(monitor, Mockito.times(2)).monitor(ArgumentMatchers.any());
    Mockito.verify(monitor, Mockito.times(1)).stop();
    scheduler.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private Monitor monitor;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    monitor = Mockito.mock(Monitor.class);
}

// === Replace local variable in test with ===
monitor;

```
</details>

---
#### Test Case ID #druid_Test_40_3
#### Test Case Name: `testStart_UnexpectedExceptionWhileScheduling`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\ClockDriftSafeMonitorSchedulerTest.java`)
#### Mock Object Variable Name: `monitor`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testStart_UnexpectedExceptionWhileScheduling() throws InterruptedException {
     ExecutorService executor = Mockito.mock(ExecutorService.class);
-    Monitor monitor = Mockito.mock(Monitor.class);
+    // removed local mock; replaced with global field `monitor`
     DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
     Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
     CountDownLatch latch = new CountDownLatch(1);
     Mockito.doAnswer(new Answer<Future<?>>() {

         @SuppressWarnings("unchecked")
         @Override
         public Future<?> answer(InvocationOnMock invocation) {
             final Object originalArgument = (invocation.getArguments())[3];
             CronTask task = ((CronTask) originalArgument);
             Mockito.when(executor.submit(ArgumentMatchers.any(Callable.class))).thenThrow(new RuntimeException("Test throwing exception while scheduling"));
             cronTaskRunner.submit(() -> {
                 task.run(123L);
                 latch.countDown();
                 return null;
             });
             return createDummyFuture();
         }
     }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
     final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
     scheduler.start();
     latch.await(5, TimeUnit.SECONDS);
-    Mockito.verify(monitor, Mockito.times(1)).start();
+    Mockito.verify(monitor, Mockito.times(1)).start();
     Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
     Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
     scheduler.stop();
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testStart_UnexpectedExceptionWhileScheduling() throws InterruptedException {
    ExecutorService executor = Mockito.mock(ExecutorService.class);
    Monitor monitor = Mockito.mock(Monitor.class);
    DruidMonitorSchedulerConfig config = Mockito.mock(DruidMonitorSchedulerConfig.class);
    Mockito.when(config.getEmissionDuration()).thenReturn(new org.joda.time.Duration(1000L));
    CountDownLatch latch = new CountDownLatch(1);
    Mockito.doAnswer(new Answer<Future<?>>() {

        @SuppressWarnings("unchecked")
        @Override
        public Future<?> answer(InvocationOnMock invocation) {
            final Object originalArgument = (invocation.getArguments())[3];
            CronTask task = ((CronTask) originalArgument);
            Mockito.when(executor.submit(ArgumentMatchers.any(Callable.class))).thenThrow(new RuntimeException("Test throwing exception while scheduling"));
            cronTaskRunner.submit(() -> {
                task.run(123L);
                latch.countDown();
                return null;
            });
            return createDummyFuture();
        }
    }).when(cronScheduler).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    final MonitorScheduler scheduler = new ClockDriftSafeMonitorScheduler(config, Mockito.mock(ServiceEmitter.class), ImmutableList.of(monitor), cronScheduler, executor);
    scheduler.start();
    latch.await(5, TimeUnit.SECONDS);
    Mockito.verify(monitor, Mockito.times(1)).start();
    Mockito.verify(cronScheduler, Mockito.times(1)).scheduleAtFixedRate(ArgumentMatchers.anyLong(), ArgumentMatchers.anyLong(), ArgumentMatchers.any(), ArgumentMatchers.any(CronTask.class));
    Mockito.verify(executor, Mockito.times(1)).submit(ArgumentMatchers.any(Callable.class));
    scheduler.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private Monitor monitor;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    monitor = Mockito.mock(Monitor.class);
}

// === Replace local variable in test with ===
monitor;

```
</details>

---
## Mock Clone Instance #druid_MCI_41
- **Scope**: class level
- **Mocked Class**: `org.apache.druid.tasklogs.TaskLogs`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
public class MockTaskLogs {
  /**
   * Creates a mock TaskLogs with stubbed streamTaskPayload behavior.
   *
   * @param id The id argument to match in streamTaskPayload.
   * @param payloadStream The ByteArrayInputStream to wrap in Optional and return.
   * @return Configured mock TaskLogs instance.
   */
  public static TaskLogs createMockTaskLogs(String id, ByteArrayInputStream payloadStream) {
    TaskLogs mockTaskLogs = Mockito.mock(TaskLogs.class);
    Mockito.when(mockTaskLogs.streamTaskPayload(id))
           .thenReturn(com.google.common.base.Optional.of(payloadStream));
    return mockTaskLogs;
  }
}

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_41_1
#### Test Case Name: `toTask_useTaskPayloadManager`(File: `C:\Java_projects\Apache\druid\extensions-contrib\kubernetes-overlord-extensions\src\test\java\org\apache\druid\k8s\overlord\taskadapter\K8sTaskAdapterTest.java`)
#### Mock Object Variable Name: `mockTestLogs`
<summary>Suggested Diff</summary>

```diff
@@
    Task taskInTaskPayloadManager = K8sTestUtils.getTask();
-    TaskLogs mockTestLogs = Mockito.mock(TaskLogs.class);
-    Mockito.when(mockTestLogs.streamTaskPayload("ID")).thenReturn(com.google.common.base.Optional.of(new ByteArrayInputStream(jsonMapper.writeValueAsString(taskInTaskPayloadManager).getBytes(Charset.defaultCharset()))));
+    TaskLogs mockTestLogs = MockTaskLogs.createMockTaskLogs(
+        "ID",
+        new ByteArrayInputStream(jsonMapper.writeValueAsString(taskInTaskPayloadManager).getBytes(Charset.defaultCharset()))
+    );
    K8sTaskAdapter adapter = new SingleContainerTaskAdapter(testClient, config, taskConfig, startupLoggingConfig, node, jsonMapper, mockTestLogs);
    Job job = new JobBuilder().editMetadata().withName("job").endMetadata().editSpec().editTemplate().editMetadata().addToAnnotations(DruidK8sConstants.TASK_ID, "ID").endMetadata().editSpec().addToContainers(new ContainerBuilder().withName("main").build()).endSpec().endTemplate().endSpec().build();
    Task taskFromJob = adapter.toTask(job);
    assertEquals(taskInTaskPayloadManager, taskFromJob);
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void toTask_useTaskPayloadManager() throws IOException {
    TestKubernetesClient testClient = new TestKubernetesClient(client);
    KubernetesTaskRunnerConfig config = KubernetesTaskRunnerConfig.builder().withNamespace("test").build();
    Task taskInTaskPayloadManager = K8sTestUtils.getTask();
    TaskLogs mockTestLogs = Mockito.mock(TaskLogs.class);
    Mockito.when(mockTestLogs.streamTaskPayload("ID")).thenReturn(com.google.common.base.Optional.of(new ByteArrayInputStream(jsonMapper.writeValueAsString(taskInTaskPayloadManager).getBytes(Charset.defaultCharset()))));
    K8sTaskAdapter adapter = new SingleContainerTaskAdapter(testClient, config, taskConfig, startupLoggingConfig, node, jsonMapper, mockTestLogs);
    Job job = new JobBuilder().editMetadata().withName("job").endMetadata().editSpec().editTemplate().editMetadata().addToAnnotations(DruidK8sConstants.TASK_ID, "ID").endMetadata().editSpec().addToContainers(new ContainerBuilder().withName("main").build()).endSpec().endTemplate().endSpec().build();
    Task taskFromJob = adapter.toTask(job);
    assertEquals(taskInTaskPayloadManager, taskFromJob);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockTaskLogs {
  /**
   * Creates a mock TaskLogs with stubbed streamTaskPayload behavior.
   *
   * @param id The id argument to match in streamTaskPayload.
   * @param payloadStream The ByteArrayInputStream to wrap in Optional and return.
   * @return Configured mock TaskLogs instance.
   */
  public static TaskLogs createMockTaskLogs(String id, ByteArrayInputStream payloadStream) {
    TaskLogs mockTaskLogs = Mockito.mock(TaskLogs.class);
    Mockito.when(mockTaskLogs.streamTaskPayload(id))
           .thenReturn(com.google.common.base.Optional.of(payloadStream));
    return mockTaskLogs;
  }
}

```
</details>

---
#### Test Case ID #druid_Test_41_2
#### Test Case Name: `test_toTask_useTaskPayloadManager`(File: `C:\Java_projects\Apache\druid\extensions-contrib\kubernetes-overlord-extensions\src\test\java\org\apache\druid\k8s\overlord\taskadapter\PodTemplateTaskAdapterTest.java`)
#### Mock Object Variable Name: `mockTestLogs`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    Task expected = new NoopTask("id", null, "datasource", 0, 0, ImmutableMap.of());
-    TaskLogs mockTestLogs = Mockito.mock(TaskLogs.class);
-    Mockito.when(mockTestLogs.streamTaskPayload("id")).thenReturn(Optional.of(new ByteArrayInputStream(mapper.writeValueAsString(expected).getBytes(Charset.defaultCharset()))));
+    TaskLogs mockTestLogs = MockTaskLogs.createMockTaskLogs(
+        "id",
+        new ByteArrayInputStream(mapper.writeValueAsString(expected).getBytes(Charset.defaultCharset()))
+    );
    PodTemplateTaskAdapter adapter = new PodTemplateTaskAdapter(taskRunnerConfig, taskConfig, node, mapper, props, mockTestLogs);
    Job job = K8sTestUtils.fileToResource("expectedNoopJob.yaml", Job.class);
    Task actual = adapter.toTask(job);
    Assertions.assertEquals(expected, actual);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void test_toTask_useTaskPayloadManager() throws IOException {
    Path templatePath = Files.createFile(tempDir.resolve("base.yaml"));
    mapper.writeValue(templatePath.toFile(), podTemplateSpec);
    Properties props = new Properties();
    props.put("druid.indexer.runner.k8s.podTemplate.base", templatePath.toString());
    Task expected = new NoopTask("id", null, "datasource", 0, 0, ImmutableMap.of());
    TaskLogs mockTestLogs = Mockito.mock(TaskLogs.class);
    Mockito.when(mockTestLogs.streamTaskPayload("id")).thenReturn(Optional.of(new ByteArrayInputStream(mapper.writeValueAsString(expected).getBytes(Charset.defaultCharset()))));
    PodTemplateTaskAdapter adapter = new PodTemplateTaskAdapter(taskRunnerConfig, taskConfig, node, mapper, props, mockTestLogs);
    Job job = K8sTestUtils.fileToResource("expectedNoopJob.yaml", Job.class);
    Task actual = adapter.toTask(job);
    Assertions.assertEquals(expected, actual);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockTaskLogs {
  /**
   * Creates a mock TaskLogs with stubbed streamTaskPayload behavior.
   *
   * @param id The id argument to match in streamTaskPayload.
   * @param payloadStream The ByteArrayInputStream to wrap in Optional and return.
   * @return Configured mock TaskLogs instance.
   */
  public static TaskLogs createMockTaskLogs(String id, ByteArrayInputStream payloadStream) {
    TaskLogs mockTaskLogs = Mockito.mock(TaskLogs.class);
    Mockito.when(mockTaskLogs.streamTaskPayload(id))
           .thenReturn(com.google.common.base.Optional.of(payloadStream));
    return mockTaskLogs;
  }
}

```
</details>

---
## Mock Clone Instance #druid_MCI_42
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.tasklogs.TaskLogPusher`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
// === Declare in class scope ===
private TaskLogPusher pusher;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    pusher = mock(TaskLogPusher.class);
}

// === Replace local variable in test with ===
pusher

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_42_1
#### Test Case Name: `testSetupAndCleanupIsCalledWtihParameter`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\AbstractTaskTest.java`)
#### Mock Object Variable Name: `pusher`
<summary>Suggested Diff</summary>

```diff
@@
     when(toolbox.getTaskExecutorNode()).thenReturn(node);
-    TaskLogPusher pusher = mock(TaskLogPusher.class);
+    // removed local mock; replaced with global field `pusher`
     when(toolbox.getTaskLogPusher()).thenReturn(pusher);
     TaskConfig config = mock(TaskConfig.class);
@@
     Mockito.verify(taskActionClient, times(3)).submit(any());
-    verify(pusher, times(1)).pushTaskReports(eq("myID"), any());
+    verify(pusher, times(1)).pushTaskReports(eq("myID"), any());
-    verify(pusher, times(1)).pushTaskStatus(eq("myID"), any());
+    verify(pusher, times(1)).pushTaskStatus(eq("myID"), any());
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSetupAndCleanupIsCalledWtihParameter() throws Exception {
    // These tests apparently use Mockito.  Mockito is bad as we've seen it rewrite byte code and effectively cause
    // impact to other totally unrelated tests.  Mockito needs to be completely erradicated from the codebase.  This
    // comment is here to either cause me to do it in this commit or just for posterity so that it is clear that it
    // should happen in the future.
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn("1");
    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
    when(toolbox.getTaskExecutorNode()).thenReturn(node);
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(true);
    File folder = temporaryFolder.newFolder();
    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Nullable
        @Override
        public String setup(TaskToolbox toolbox) throws Exception {
            // create a reports file to test the taskLogPusher pushes task reports
            String result = super.setup(toolbox);
            File attemptDir = Paths.get(folder.getAbsolutePath(), "attempt", toolbox.getAttemptId()).toFile();
            File reportsDir = new File(attemptDir, "report.json");
            File statusDir = new File(attemptDir, "status.json");
            FileUtils.write(reportsDir, "foo", StandardCharsets.UTF_8);
            FileUtils.write(statusDir, "{}", StandardCharsets.UTF_8);
            return result;
        }
    };
    task.run(toolbox);
    // call it 3 times, once to update location in setup, then one for status and location in cleanup
    Mockito.verify(taskActionClient, times(3)).submit(any());
    verify(pusher, times(1)).pushTaskReports(eq("myID"), any());
    verify(pusher, times(1)).pushTaskStatus(eq("myID"), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private TaskLogPusher pusher;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    pusher = mock(TaskLogPusher.class);
}

// === Replace local variable in test with ===
pusher

```
</details>

---
#### Test Case ID #druid_Test_42_2
#### Test Case Name: `testWithNoEncapsulatedTask`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\AbstractTaskTest.java`)
#### Mock Object Variable Name: `pusher`
<summary>Suggested Diff</summary>

```diff
@@
     DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
     when(toolbox.getTaskExecutorNode()).thenReturn(node);
-    TaskLogPusher pusher = mock(TaskLogPusher.class);
+    // removed local mock; replaced with global field `pusher`
     when(toolbox.getTaskLogPusher()).thenReturn(pusher);
     TaskConfig config = mock(TaskConfig.class);
@@
     Mockito.verify(taskActionClient, never()).submit(any());
-    verify(pusher, never()).pushTaskReports(eq("myID"), any());
+    verify(pusher, never()).pushTaskReports(eq("myID"), any());
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testWithNoEncapsulatedTask() throws Exception {
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn("1");
    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
    when(toolbox.getTaskExecutorNode()).thenReturn(node);
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(false);
    File folder = temporaryFolder.newFolder();
    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Nullable
        @Override
        public String setup(TaskToolbox toolbox) throws Exception {
            // create a reports file to test the taskLogPusher pushes task reports
            String result = super.setup(toolbox);
            File attemptDir = Paths.get(folder.getAbsolutePath(), "attempt", toolbox.getAttemptId()).toFile();
            File reportsDir = new File(attemptDir, "report.json");
            FileUtils.write(reportsDir, "foo", StandardCharsets.UTF_8);
            return result;
        }
    };
    task.run(toolbox);
    // encapsulated task is set to false, should never get called
    Mockito.verify(taskActionClient, never()).submit(any());
    verify(pusher, never()).pushTaskReports(eq("myID"), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private TaskLogPusher pusher;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    pusher = mock(TaskLogPusher.class);
}

// === Replace local variable in test with ===
pusher

```
</details>

---
#### Test Case ID #druid_Test_42_3
#### Test Case Name: `testTaskFailureWithoutExceptionGetsReportedCorrectly`(File: `C:\Java_projects\Apache\druid\indexing-service\src\test\java\org\apache\druid\indexing\common\task\AbstractTaskTest.java`)
#### Mock Object Variable Name: `pusher`
<summary>Suggested Diff</summary>

```diff
@@
     when(toolbox.getTaskExecutorNode()).thenReturn(node);
-    TaskLogPusher pusher = mock(TaskLogPusher.class);
+    // removed local mock; replaced with global field `pusher`
     when(toolbox.getTaskLogPusher()).thenReturn(pusher);
     TaskConfig config = mock(TaskConfig.class);
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testTaskFailureWithoutExceptionGetsReportedCorrectly() throws Exception {
    TaskToolbox toolbox = mock(TaskToolbox.class);
    when(toolbox.getAttemptId()).thenReturn("1");
    DruidNode node = new DruidNode("foo", "foo", false, 1, 2, true, true);
    when(toolbox.getTaskExecutorNode()).thenReturn(node);
    TaskLogPusher pusher = mock(TaskLogPusher.class);
    when(toolbox.getTaskLogPusher()).thenReturn(pusher);
    TaskConfig config = mock(TaskConfig.class);
    when(config.isEncapsulatedTask()).thenReturn(true);
    File folder = temporaryFolder.newFolder();
    when(config.getTaskDir(eq("myID"))).thenReturn(folder);
    when(toolbox.getConfig()).thenReturn(config);
    when(toolbox.getJsonMapper()).thenReturn(objectMapper);
    TaskActionClient taskActionClient = mock(TaskActionClient.class);
    when(taskActionClient.submit(any())).thenReturn(TaskConfig.class);
    when(toolbox.getTaskActionClient()).thenReturn(taskActionClient);
    AbstractTask task = new NoopTask("myID", null, null, 1, 0, null) {

        @Override
        public TaskStatus runTask(TaskToolbox toolbox) {
            return TaskStatus.failure("myId", "failed");
        }
    };
    task.run(toolbox);
    UpdateStatusAction action = new UpdateStatusAction("failure");
    verify(taskActionClient).submit(eq(action));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private TaskLogPusher pusher;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    pusher = mock(TaskLogPusher.class);
}

// === Replace local variable in test with ===
pusher

```
</details>

---
## Mock Clone Instance #druid_MCI_43
- **Scope**: class level
- **Mocked Class**: `org.apache.druid.query.groupby.epinephelinae.column.GroupByColumnSelectorPlus`
- **Test Case Count**: 3
- **MO Count**: 9

### Reusable Method
```java
public class MockGroupByColumnSelectorPlus {
  public static GroupByColumnSelectorPlus createMockGroupByColumnSelectorPlus() {
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    return groupByColumnSelectorPlus;
  }
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_43_1
#### Test Case Name: `testSanity`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayDoubleGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `groupByColumnSelectorPlus`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    Assert.assertEquals(0, strategy.computeDictionaryId(columnValueSelector));
-    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
-    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
+    GroupByColumnSelectorPlus groupByColumnSelectorPlus = MockGroupByColumnSelectorPlus.createMockGroupByColumnSelectorPlus();
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(0);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(1.0, 2.0)), row.get(0));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSanity() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of(1.0, 2.0));
    Assert.assertEquals(0, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(0);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(1.0, 2.0)), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockGroupByColumnSelectorPlus {
  public static GroupByColumnSelectorPlus createMockGroupByColumnSelectorPlus() {
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    return groupByColumnSelectorPlus;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_43_2
#### Test Case Name: `testAddingInDictionary`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayDoubleGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `groupByColumnSelectorPlus`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
-    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
-    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
+    GroupByColumnSelectorPlus groupByColumnSelectorPlus = MockGroupByColumnSelectorPlus.createMockGroupByColumnSelectorPlus();
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4.0, 2.0)), row.get(0));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testAddingInDictionary() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of(4.0, 2.0));
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4.0, 2.0)), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockGroupByColumnSelectorPlus {
  public static GroupByColumnSelectorPlus createMockGroupByColumnSelectorPlus() {
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    return groupByColumnSelectorPlus;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_43_3
#### Test Case Name: `testAddingInDictionaryWithObjects`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayDoubleGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `groupByColumnSelectorPlus`
<summary>Suggested Diff</summary>

```diff
@@
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
-    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
-    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
+    GroupByColumnSelectorPlus groupByColumnSelectorPlus = MockGroupByColumnSelectorPlus.createMockGroupByColumnSelectorPlus();
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4.0, 2.0)), row.get(0));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testAddingInDictionaryWithObjects() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(new Object[] { 4.0D, 2.0D });
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4.0, 2.0)), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockGroupByColumnSelectorPlus {
  public static GroupByColumnSelectorPlus createMockGroupByColumnSelectorPlus() {
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    return groupByColumnSelectorPlus;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_43_4
#### Test Case Name: `testSanity`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayLongGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `groupByColumnSelectorPlus`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    Assert.assertEquals(0, strategy.computeDictionaryId(columnValueSelector));
-    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
-    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
+    GroupByColumnSelectorPlus groupByColumnSelectorPlus = MockGroupByColumnSelectorPlus.createMockGroupByColumnSelectorPlus();
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(0);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(1L, 2L)), row.get(0));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSanity() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of(1L, 2L));
    Assert.assertEquals(0, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(0);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(1L, 2L)), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockGroupByColumnSelectorPlus {
  public static GroupByColumnSelectorPlus createMockGroupByColumnSelectorPlus() {
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    return groupByColumnSelectorPlus;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_43_5
#### Test Case Name: `testAddingInDictionary`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayLongGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `groupByColumnSelectorPlus`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
-    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
-    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
+    GroupByColumnSelectorPlus groupByColumnSelectorPlus = MockGroupByColumnSelectorPlus.createMockGroupByColumnSelectorPlus();
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4L, 2L)), row.get(0));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testAddingInDictionary() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of(4L, 2L));
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4L, 2L)), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockGroupByColumnSelectorPlus {
  public static GroupByColumnSelectorPlus createMockGroupByColumnSelectorPlus() {
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    return groupByColumnSelectorPlus;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_43_6
#### Test Case Name: `testAddingInDictionaryWithObjects`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayLongGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `groupByColumnSelectorPlus`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
-    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
-    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
+    GroupByColumnSelectorPlus groupByColumnSelectorPlus = MockGroupByColumnSelectorPlus.createMockGroupByColumnSelectorPlus();
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4L, 2L)), row.get(0));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testAddingInDictionaryWithObjects() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(new Object[] { 4L, 2L });
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(new ComparableList<>(ImmutableList.of(4L, 2L)), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockGroupByColumnSelectorPlus {
  public static GroupByColumnSelectorPlus createMockGroupByColumnSelectorPlus() {
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    return groupByColumnSelectorPlus;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_43_7
#### Test Case Name: `testSanity`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayStringGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `groupByColumnSelectorPlus`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    Assert.assertEquals(0, strategy.computeDictionaryId(columnValueSelector));
-    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
-    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
+    GroupByColumnSelectorPlus groupByColumnSelectorPlus = MockGroupByColumnSelectorPlus.createMockGroupByColumnSelectorPlus();
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(0);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(ComparableStringArray.of("a", "b"), row.get(0));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testSanity() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of("a", "b"));
    Assert.assertEquals(0, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(0);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(ComparableStringArray.of("a", "b"), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockGroupByColumnSelectorPlus {
  public static GroupByColumnSelectorPlus createMockGroupByColumnSelectorPlus() {
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    return groupByColumnSelectorPlus;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_43_8
#### Test Case Name: `testAddingInDictionary`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayStringGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `groupByColumnSelectorPlus`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
-    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
-    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
+    GroupByColumnSelectorPlus groupByColumnSelectorPlus = MockGroupByColumnSelectorPlus.createMockGroupByColumnSelectorPlus();
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(ComparableStringArray.of("f", "a"), row.get(0));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testAddingInDictionary() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(ImmutableList.of("f", "a"));
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(ComparableStringArray.of("f", "a"), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockGroupByColumnSelectorPlus {
  public static GroupByColumnSelectorPlus createMockGroupByColumnSelectorPlus() {
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    return groupByColumnSelectorPlus;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_43_9
#### Test Case Name: `testAddingInDictionaryWithObjects`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\groupby\epinephelinae\column\ArrayStringGroupByColumnSelectorStrategyTest.java`)
#### Mock Object Variable Name: `groupByColumnSelectorPlus`
<summary>Suggested Diff</summary>

```diff
@@
     Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
-    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
-    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
+    GroupByColumnSelectorPlus groupByColumnSelectorPlus = MockGroupByColumnSelectorPlus.createMockGroupByColumnSelectorPlus();
     ResultRow row = ResultRow.create(1);
     buffer1.putInt(3);
     strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
     Assert.assertEquals(ComparableStringArray.of("f", "a"), row.get(0));
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testAddingInDictionaryWithObjects() {
    ColumnValueSelector columnValueSelector = Mockito.mock(ColumnValueSelector.class);
    Mockito.when(columnValueSelector.getObject()).thenReturn(new Object[] { "f", "a" });
    Assert.assertEquals(3, strategy.computeDictionaryId(columnValueSelector));
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    ResultRow row = ResultRow.create(1);
    buffer1.putInt(3);
    strategy.processValueFromGroupingKey(groupByColumnSelectorPlus, buffer1, row, 0);
    Assert.assertEquals(ComparableStringArray.of("f", "a"), row.get(0));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockGroupByColumnSelectorPlus {
  public static GroupByColumnSelectorPlus createMockGroupByColumnSelectorPlus() {
    GroupByColumnSelectorPlus groupByColumnSelectorPlus = Mockito.mock(GroupByColumnSelectorPlus.class);
    Mockito.when(groupByColumnSelectorPlus.getResultRowPosition()).thenReturn(0);
    return groupByColumnSelectorPlus;
  }
}
```
</details>

---
## Mock Clone Instance #druid_MCI_44
- **Scope**: method level
- **Mocked Class**: `oshi.hardware.HWDiskStore`
- **Test Case Count**: 1
- **MO Count**: 2

### Reusable Method
```java
private static HWDiskStore createMockHWDiskStore(long readBytes, long reads, long writeBytes, long writes, long currentQueueLength, long transferTime, String name) {
    HWDiskStore disk = Mockito.mock(HWDiskStore.class);
    Mockito.when(disk.getReadBytes()).thenReturn(readBytes);
    Mockito.when(disk.getReads()).thenReturn(reads);
    Mockito.when(disk.getWriteBytes()).thenReturn(writeBytes);
    Mockito.when(disk.getWrites()).thenReturn(writes);
    Mockito.when(disk.getCurrentQueueLength()).thenReturn(currentQueueLength);
    Mockito.when(disk.getTransferTime()).thenReturn(transferTime);
    Mockito.when(disk.getName()).thenReturn(name);
    return disk;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_44_1
#### Test Case Name: `testDiskStats`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\OshiSysMonitorTest.java`)
#### Mock Object Variable Name: `disk1`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    StubServiceEmitter emitter = new StubServiceEmitter("dev/monitor-test", "localhost:0000");
-    HWDiskStore disk1 = Mockito.mock(HWDiskStore.class);
+    HWDiskStore disk1 = createMockHWDiskStore(300L, 200L, 400L, 500L, 100L, 150L, "disk1");
    HWDiskStore disk2 = Mockito.mock(HWDiskStore.class);
-    Mockito.when(disk1.getReadBytes()).thenReturn(300L);
-    Mockito.when(disk1.getReads()).thenReturn(200L);
-    Mockito.when(disk1.getWriteBytes()).thenReturn(400L);
-    Mockito.when(disk1.getWrites()).thenReturn(500L);
-    Mockito.when(disk1.getCurrentQueueLength()).thenReturn(100L);
-    Mockito.when(disk1.getTransferTime()).thenReturn(150L);
-    Mockito.when(disk1.getName()).thenReturn("disk1");
    Mockito.when(disk2.getReadBytes()).thenReturn(2000L);
    Mockito.when(disk2.getReads()).thenReturn(3000L);
    Mockito.when(disk2.getWriteBytes()).thenReturn(1000L);
    Mockito.when(disk2.getWrites()).thenReturn(4000L);
    Mockito.when(disk2.getCurrentQueueLength()).thenReturn(750L);
    Mockito.when(disk2.getTransferTime()).thenReturn(800L);
    Mockito.when(disk2.getName()).thenReturn("disk2");
    List<HWDiskStore> hwDiskStores = ImmutableList.of(disk1, disk2);
    Mockito.when(hal.getDiskStores()).thenReturn(hwDiskStores);
    OshiSysMonitor m = new OshiSysMonitor(si);
    m.start();
    m.monitorDiskStats(emitter);
    Assert.assertEquals(0, emitter.getEvents().size());
+    Mockito.when(disk1.getReadBytes()).thenReturn(400L);
+    Mockito.when(disk1.getReads()).thenReturn(220L);
+    Mockito.when(disk1.getWriteBytes()).thenReturn(600L);
+    Mockito.when(disk1.getWrites()).thenReturn(580L);
+    Mockito.when(disk1.getCurrentQueueLength()).thenReturn(300L);
+    Mockito.when(disk1.getTransferTime()).thenReturn(250L);
    Mockito.when(disk2.getReadBytes()).thenReturn(4500L);
    Mockito.when(disk2.getReads()).thenReturn(3500L);
    Mockito.when(disk2.getWriteBytes()).thenReturn(2300L);
    Mockito.when(disk2.getWrites()).thenReturn(5000L);
    Mockito.when(disk2.getCurrentQueueLength()).thenReturn(900L);
    Mockito.when(disk2.getTransferTime()).thenReturn(1100L);
    m.monitorDiskStats(emitter);
    Assert.assertEquals(12, emitter.getEvents().size());
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testDiskStats() {
    StubServiceEmitter emitter = new StubServiceEmitter("dev/monitor-test", "localhost:0000");
    HWDiskStore disk1 = Mockito.mock(HWDiskStore.class);
    HWDiskStore disk2 = Mockito.mock(HWDiskStore.class);
    Mockito.when(disk1.getReadBytes()).thenReturn(300L);
    Mockito.when(disk1.getReads()).thenReturn(200L);
    Mockito.when(disk1.getWriteBytes()).thenReturn(400L);
    Mockito.when(disk1.getWrites()).thenReturn(500L);
    Mockito.when(disk1.getCurrentQueueLength()).thenReturn(100L);
    Mockito.when(disk1.getTransferTime()).thenReturn(150L);
    Mockito.when(disk1.getName()).thenReturn("disk1");
    Mockito.when(disk2.getReadBytes()).thenReturn(2000L);
    Mockito.when(disk2.getReads()).thenReturn(3000L);
    Mockito.when(disk2.getWriteBytes()).thenReturn(1000L);
    Mockito.when(disk2.getWrites()).thenReturn(4000L);
    Mockito.when(disk2.getCurrentQueueLength()).thenReturn(750L);
    Mockito.when(disk2.getTransferTime()).thenReturn(800L);
    Mockito.when(disk2.getName()).thenReturn("disk2");
    List<HWDiskStore> hwDiskStores = ImmutableList.of(disk1, disk2);
    Mockito.when(hal.getDiskStores()).thenReturn(hwDiskStores);
    OshiSysMonitor m = new OshiSysMonitor(si);
    m.start();
    m.monitorDiskStats(emitter);
    Assert.assertEquals(0, emitter.getEvents().size());
    Mockito.when(disk1.getReadBytes()).thenReturn(400L);
    Mockito.when(disk1.getReads()).thenReturn(220L);
    Mockito.when(disk1.getWriteBytes()).thenReturn(600L);
    Mockito.when(disk1.getWrites()).thenReturn(580L);
    Mockito.when(disk1.getCurrentQueueLength()).thenReturn(300L);
    Mockito.when(disk1.getTransferTime()).thenReturn(250L);
    Mockito.when(disk2.getReadBytes()).thenReturn(4500L);
    Mockito.when(disk2.getReads()).thenReturn(3500L);
    Mockito.when(disk2.getWriteBytes()).thenReturn(2300L);
    Mockito.when(disk2.getWrites()).thenReturn(5000L);
    Mockito.when(disk2.getCurrentQueueLength()).thenReturn(900L);
    Mockito.when(disk2.getTransferTime()).thenReturn(1100L);
    m.monitorDiskStats(emitter);
    Assert.assertEquals(12, emitter.getEvents().size());
    Map<String, Object> userDims1 = ImmutableMap.of("diskName", "disk1");
    List<Number> metricValues1 = emitter.getMetricValues("sys/disk/read/size", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(100L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/disk/read/count", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(20L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/disk/write/size", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(200L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/disk/write/count", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(80L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/disk/queue", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(200L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/disk/transferTime", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(100L, metricValues1.get(0));
    Map<String, Object> userDims2 = ImmutableMap.of("diskName", "disk2");
    List<Number> metricValues2 = emitter.getMetricValues("sys/disk/read/size", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(2500L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/disk/read/count", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(500L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/disk/write/size", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(1300L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/disk/write/count", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(1000L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/disk/queue", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(150L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/disk/transferTime", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(300L, metricValues2.get(0));
    m.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HWDiskStore createMockHWDiskStore(long readBytes, long reads, long writeBytes, long writes, long currentQueueLength, long transferTime, String name) {
    HWDiskStore disk = Mockito.mock(HWDiskStore.class);
    Mockito.when(disk.getReadBytes()).thenReturn(readBytes);
    Mockito.when(disk.getReads()).thenReturn(reads);
    Mockito.when(disk.getWriteBytes()).thenReturn(writeBytes);
    Mockito.when(disk.getWrites()).thenReturn(writes);
    Mockito.when(disk.getCurrentQueueLength()).thenReturn(currentQueueLength);
    Mockito.when(disk.getTransferTime()).thenReturn(transferTime);
    Mockito.when(disk.getName()).thenReturn(name);
    return disk;
}
```
</details>

---
#### Test Case ID #druid_Test_44_2
#### Test Case Name: `testDiskStats`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\java\util\metrics\OshiSysMonitorTest.java`)
#### Mock Object Variable Name: `disk2`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    StubServiceEmitter emitter = new StubServiceEmitter("dev/monitor-test", "localhost:0000");
    HWDiskStore disk1 = Mockito.mock(HWDiskStore.class);
-    HWDiskStore disk2 = Mockito.mock(HWDiskStore.class);
-    Mockito.when(disk1.getReadBytes()).thenReturn(300L);
-    Mockito.when(disk1.getReads()).thenReturn(200L);
-    Mockito.when(disk1.getWriteBytes()).thenReturn(400L);
-    Mockito.when(disk1.getWrites()).thenReturn(500L);
-    Mockito.when(disk1.getCurrentQueueLength()).thenReturn(100L);
-    Mockito.when(disk1.getTransferTime()).thenReturn(150L);
-    Mockito.when(disk1.getName()).thenReturn("disk1");
-    Mockito.when(disk2.getReadBytes()).thenReturn(2000L);
-    Mockito.when(disk2.getReads()).thenReturn(3000L);
-    Mockito.when(disk2.getWriteBytes()).thenReturn(1000L);
-    Mockito.when(disk2.getWrites()).thenReturn(4000L);
-    Mockito.when(disk2.getCurrentQueueLength()).thenReturn(750L);
-    Mockito.when(disk2.getTransferTime()).thenReturn(800L);
-    Mockito.when(disk2.getName()).thenReturn("disk2");
+    HWDiskStore disk2 = createMockHWDiskStore(2000L, 3000L, 1000L, 4000L, 750L, 800L, "disk2");
    List<HWDiskStore> hwDiskStores = ImmutableList.of(disk1, disk2);
    Mockito.when(hal.getDiskStores()).thenReturn(hwDiskStores);
    OshiSysMonitor m = new OshiSysMonitor(si);
    m.start();
    m.monitorDiskStats(emitter);
    Assert.assertEquals(0, emitter.getEvents().size());
    Mockito.when(disk1.getReadBytes()).thenReturn(400L);
    Mockito.when(disk1.getReads()).thenReturn(220L);
    Mockito.when(disk1.getWriteBytes()).thenReturn(600L);
    Mockito.when(disk1.getWrites()).thenReturn(580L);
    Mockito.when(disk1.getCurrentQueueLength()).thenReturn(300L);
    Mockito.when(disk1.getTransferTime()).thenReturn(250L);
-    Mockito.when(disk2.getReadBytes()).thenReturn(4500L);
-    Mockito.when(disk2.getReads()).thenReturn(3500L);
-    Mockito.when(disk2.getWriteBytes()).thenReturn(2300L);
-    Mockito.when(disk2.getWrites()).thenReturn(5000L);
-    Mockito.when(disk2.getCurrentQueueLength()).thenReturn(900L);
-    Mockito.when(disk2.getTransferTime()).thenReturn(1100L);
+    Mockito.when(disk2.getReadBytes()).thenReturn(4500L);
+    Mockito.when(disk2.getReads()).thenReturn(3500L);
+    Mockito.when(disk2.getWriteBytes()).thenReturn(2300L);
+    Mockito.when(disk2.getWrites()).thenReturn(5000L);
+    Mockito.when(disk2.getCurrentQueueLength()).thenReturn(900L);
+    Mockito.when(disk2.getTransferTime()).thenReturn(1100L);
    m.monitorDiskStats(emitter);
    Assert.assertEquals(12, emitter.getEvents().size());
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testDiskStats() {
    StubServiceEmitter emitter = new StubServiceEmitter("dev/monitor-test", "localhost:0000");
    HWDiskStore disk1 = Mockito.mock(HWDiskStore.class);
    HWDiskStore disk2 = Mockito.mock(HWDiskStore.class);
    Mockito.when(disk1.getReadBytes()).thenReturn(300L);
    Mockito.when(disk1.getReads()).thenReturn(200L);
    Mockito.when(disk1.getWriteBytes()).thenReturn(400L);
    Mockito.when(disk1.getWrites()).thenReturn(500L);
    Mockito.when(disk1.getCurrentQueueLength()).thenReturn(100L);
    Mockito.when(disk1.getTransferTime()).thenReturn(150L);
    Mockito.when(disk1.getName()).thenReturn("disk1");
    Mockito.when(disk2.getReadBytes()).thenReturn(2000L);
    Mockito.when(disk2.getReads()).thenReturn(3000L);
    Mockito.when(disk2.getWriteBytes()).thenReturn(1000L);
    Mockito.when(disk2.getWrites()).thenReturn(4000L);
    Mockito.when(disk2.getCurrentQueueLength()).thenReturn(750L);
    Mockito.when(disk2.getTransferTime()).thenReturn(800L);
    Mockito.when(disk2.getName()).thenReturn("disk2");
    List<HWDiskStore> hwDiskStores = ImmutableList.of(disk1, disk2);
    Mockito.when(hal.getDiskStores()).thenReturn(hwDiskStores);
    OshiSysMonitor m = new OshiSysMonitor(si);
    m.start();
    m.monitorDiskStats(emitter);
    Assert.assertEquals(0, emitter.getEvents().size());
    Mockito.when(disk1.getReadBytes()).thenReturn(400L);
    Mockito.when(disk1.getReads()).thenReturn(220L);
    Mockito.when(disk1.getWriteBytes()).thenReturn(600L);
    Mockito.when(disk1.getWrites()).thenReturn(580L);
    Mockito.when(disk1.getCurrentQueueLength()).thenReturn(300L);
    Mockito.when(disk1.getTransferTime()).thenReturn(250L);
    Mockito.when(disk2.getReadBytes()).thenReturn(4500L);
    Mockito.when(disk2.getReads()).thenReturn(3500L);
    Mockito.when(disk2.getWriteBytes()).thenReturn(2300L);
    Mockito.when(disk2.getWrites()).thenReturn(5000L);
    Mockito.when(disk2.getCurrentQueueLength()).thenReturn(900L);
    Mockito.when(disk2.getTransferTime()).thenReturn(1100L);
    m.monitorDiskStats(emitter);
    Assert.assertEquals(12, emitter.getEvents().size());
    Map<String, Object> userDims1 = ImmutableMap.of("diskName", "disk1");
    List<Number> metricValues1 = emitter.getMetricValues("sys/disk/read/size", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(100L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/disk/read/count", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(20L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/disk/write/size", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(200L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/disk/write/count", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(80L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/disk/queue", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(200L, metricValues1.get(0));
    metricValues1 = emitter.getMetricValues("sys/disk/transferTime", userDims1);
    Assert.assertEquals(1, metricValues1.size());
    Assert.assertEquals(100L, metricValues1.get(0));
    Map<String, Object> userDims2 = ImmutableMap.of("diskName", "disk2");
    List<Number> metricValues2 = emitter.getMetricValues("sys/disk/read/size", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(2500L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/disk/read/count", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(500L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/disk/write/size", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(1300L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/disk/write/count", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(1000L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/disk/queue", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(150L, metricValues2.get(0));
    metricValues2 = emitter.getMetricValues("sys/disk/transferTime", userDims2);
    Assert.assertEquals(1, metricValues2.size());
    Assert.assertEquals(300L, metricValues2.get(0));
    m.stop();
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HWDiskStore createMockHWDiskStore(long readBytes, long reads, long writeBytes, long writes, long currentQueueLength, long transferTime, String name) {
    HWDiskStore disk = Mockito.mock(HWDiskStore.class);
    Mockito.when(disk.getReadBytes()).thenReturn(readBytes);
    Mockito.when(disk.getReads()).thenReturn(reads);
    Mockito.when(disk.getWriteBytes()).thenReturn(writeBytes);
    Mockito.when(disk.getWrites()).thenReturn(writes);
    Mockito.when(disk.getCurrentQueueLength()).thenReturn(currentQueueLength);
    Mockito.when(disk.getTransferTime()).thenReturn(transferTime);
    Mockito.when(disk.getName()).thenReturn(name);
    return disk;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_45
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.java.util.emitter.service.ServiceEmitter`
- **Test Case Count**: 4
- **MO Count**: 4

### Reusable Method
```java
// === Declare in class scope ===
private ServiceEmitter emitter;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    emitter = Mockito.mock(ServiceEmitter.class);
}

// === Replace local variable in test with ===
emitter;

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_45_1
#### Test Case Name: `testGetSysMonitorViaInjector`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\metrics\MetricsModuleTest.java`)
#### Mock Object Variable Name: `emitter`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testGetSysMonitorViaInjector() {
     // Do not run the tests on ARM64. Sigar library has no binaries for ARM64
     Assume.assumeFalse("aarch64".equals(CPU_ARCH));
     final Injector injector = createInjector(new Properties(), ImmutableSet.of(NodeRole.PEON));
     final SysMonitor sysMonitor = injector.getInstance(SysMonitor.class);
-    final ServiceEmitter emitter = Mockito.mock(ServiceEmitter.class);
+    // removed local mock; replaced with global field `emitter`
     sysMonitor.doMonitor(emitter);
     Assert.assertTrue(sysMonitor instanceof NoopSysMonitor);
     Mockito.verify(emitter, Mockito.never()).emit(ArgumentMatchers.any(ServiceEventBuilder.class));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testGetSysMonitorViaInjector() {
    // Do not run the tests on ARM64. Sigar library has no binaries for ARM64
    Assume.assumeFalse("aarch64".equals(CPU_ARCH));
    final Injector injector = createInjector(new Properties(), ImmutableSet.of(NodeRole.PEON));
    final SysMonitor sysMonitor = injector.getInstance(SysMonitor.class);
    final ServiceEmitter emitter = Mockito.mock(ServiceEmitter.class);
    sysMonitor.doMonitor(emitter);
    Assert.assertTrue(sysMonitor instanceof NoopSysMonitor);
    Mockito.verify(emitter, Mockito.never()).emit(ArgumentMatchers.any(ServiceEventBuilder.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ServiceEmitter emitter;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    emitter = Mockito.mock(ServiceEmitter.class);
}

// === Replace local variable in test with ===
emitter;

```
</details>

---
#### Test Case ID #druid_Test_45_2
#### Test Case Name: `testGetSysMonitorWhenNull`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\metrics\MetricsModuleTest.java`)
#### Mock Object Variable Name: `emitter`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testGetSysMonitorWhenNull() {
     // Do not run the tests on ARM64. Sigar library has no binaries for ARM64
     Assume.assumeFalse("aarch64".equals(CPU_ARCH));
     Injector injector = createInjector(new Properties(), ImmutableSet.of());
     final SysMonitor sysMonitor = injector.getInstance(SysMonitor.class);
-    final ServiceEmitter emitter = Mockito.mock(ServiceEmitter.class);
+    // removed local mock; replaced with global field `emitter`
     sysMonitor.doMonitor(emitter);
     Assert.assertFalse(sysMonitor instanceof NoopSysMonitor);
     Mockito.verify(emitter, Mockito.atLeastOnce()).emit(ArgumentMatchers.any(ServiceEventBuilder.class));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testGetSysMonitorWhenNull() {
    // Do not run the tests on ARM64. Sigar library has no binaries for ARM64
    Assume.assumeFalse("aarch64".equals(CPU_ARCH));
    Injector injector = createInjector(new Properties(), ImmutableSet.of());
    final SysMonitor sysMonitor = injector.getInstance(SysMonitor.class);
    final ServiceEmitter emitter = Mockito.mock(ServiceEmitter.class);
    sysMonitor.doMonitor(emitter);
    Assert.assertFalse(sysMonitor instanceof NoopSysMonitor);
    Mockito.verify(emitter, Mockito.atLeastOnce()).emit(ArgumentMatchers.any(ServiceEventBuilder.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ServiceEmitter emitter;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    emitter = Mockito.mock(ServiceEmitter.class);
}

// === Replace local variable in test with ===
emitter;

```
</details>

---
#### Test Case ID #druid_Test_45_3
#### Test Case Name: `testGetOshiSysMonitorViaInjector`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\metrics\MetricsModuleTest.java`)
#### Mock Object Variable Name: `emitter`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testGetOshiSysMonitorViaInjector() {
     final Injector injector = createInjector(new Properties(), ImmutableSet.of(NodeRole.PEON));
     final OshiSysMonitor sysMonitor = injector.getInstance(OshiSysMonitor.class);
-    final ServiceEmitter emitter = Mockito.mock(ServiceEmitter.class);
+    // removed local mock; replaced with global field `emitter`
     sysMonitor.doMonitor(emitter);
     Assert.assertTrue(sysMonitor instanceof NoopOshiSysMonitor);
     Mockito.verify(emitter, Mockito.never()).emit(ArgumentMatchers.any(ServiceEventBuilder.class));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testGetOshiSysMonitorViaInjector() {
    final Injector injector = createInjector(new Properties(), ImmutableSet.of(NodeRole.PEON));
    final OshiSysMonitor sysMonitor = injector.getInstance(OshiSysMonitor.class);
    final ServiceEmitter emitter = Mockito.mock(ServiceEmitter.class);
    sysMonitor.doMonitor(emitter);
    Assert.assertTrue(sysMonitor instanceof NoopOshiSysMonitor);
    Mockito.verify(emitter, Mockito.never()).emit(ArgumentMatchers.any(ServiceEventBuilder.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ServiceEmitter emitter;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    emitter = Mockito.mock(ServiceEmitter.class);
}

// === Replace local variable in test with ===
emitter;

```
</details>

---
#### Test Case ID #druid_Test_45_4
#### Test Case Name: `testGetOshiSysMonitorWhenNull`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\server\metrics\MetricsModuleTest.java`)
#### Mock Object Variable Name: `emitter`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void testGetOshiSysMonitorWhenNull() {
     Injector injector = createInjector(new Properties(), ImmutableSet.of());
     final OshiSysMonitor sysMonitor = injector.getInstance(OshiSysMonitor.class);
-    final ServiceEmitter emitter = Mockito.mock(ServiceEmitter.class);
+    // removed local mock; replaced with global field `emitter`
     sysMonitor.doMonitor(emitter);
     Assert.assertFalse(sysMonitor instanceof NoopOshiSysMonitor);
     Mockito.verify(emitter, Mockito.atLeastOnce()).emit(ArgumentMatchers.any(ServiceEventBuilder.class));
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testGetOshiSysMonitorWhenNull() {
    Injector injector = createInjector(new Properties(), ImmutableSet.of());
    final OshiSysMonitor sysMonitor = injector.getInstance(OshiSysMonitor.class);
    final ServiceEmitter emitter = Mockito.mock(ServiceEmitter.class);
    sysMonitor.doMonitor(emitter);
    Assert.assertFalse(sysMonitor instanceof NoopOshiSysMonitor);
    Mockito.verify(emitter, Mockito.atLeastOnce()).emit(ArgumentMatchers.any(ServiceEventBuilder.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private ServiceEmitter emitter;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    emitter = Mockito.mock(ServiceEmitter.class);
}

// === Replace local variable in test with ===
emitter;

```
</details>

---
## Mock Clone Instance #druid_MCI_46
- **Scope**: method level
- **Mocked Class**: `org.apache.druid.data.input.HandlingInputRowIterator.InputRowHandler`
- **Test Case Count**: 4
- **MO Count**: 6

### Reusable Method
```java
private static HandlingInputRowIterator.InputRowHandler createMockInputRowHandler(boolean handleReturnValue) {
    HandlingInputRowIterator.InputRowHandler handler = mock(HandlingInputRowIterator.InputRowHandler.class);
    when(handler.handle(any(InputRow.class))).thenReturn(handleReturnValue);
    return handler;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_46_1
#### Test Case Name: `yieldsNullIfHandledByFirst`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\HandlingInputRowIteratorTest.java`)
#### Mock Object Variable Name: `successfulHandler`
<summary>Suggested Diff</summary>

```diff
@@
 @Before
-public void setup() {
-    // Construct mock object
-    successfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
-    // Method Stubs
-    when(successfulHandler.handle(any(InputRow.class))).thenReturn(true);
-    // Construct mock object
-    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
-    // Method Stubs
-    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
-}
+public void setup() {
+    successfulHandler = createMockInputRowHandler(true);
+    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
+    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
+}
 
 @Test
 public void yieldsNullIfHandledByFirst() {
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void yieldsNullIfHandledByFirst() {
    HandlingInputRowIterator target = createInputRowIterator(successfulHandler, unsuccessfulHandler);
    Assert.assertNull(target.next());
    Mockito.verify(successfulHandler, Mockito.times(1)).handle(INPUT_ROW1);
    Mockito.verify(unsuccessfulHandler, Mockito.never()).handle(INPUT_ROW1);
}
@Before
public void setup() {
    // Construct mock object
    successfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
    when(successfulHandler.handle(any(InputRow.class))).thenReturn(true);
    // Construct mock object
    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HandlingInputRowIterator.InputRowHandler createMockInputRowHandler(boolean handleReturnValue) {
    HandlingInputRowIterator.InputRowHandler handler = mock(HandlingInputRowIterator.InputRowHandler.class);
    when(handler.handle(any(InputRow.class))).thenReturn(handleReturnValue);
    return handler;
}
```
</details>

---
#### Test Case ID #druid_Test_46_2
#### Test Case Name: `yieldsNullIfHandledBySecond`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\HandlingInputRowIteratorTest.java`)
#### Mock Object Variable Name: `successfulHandler`
<summary>Suggested Diff</summary>

```diff
@@
 @Before
-public void setup() {
-    // Construct mock object
-    successfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
-    // Method Stubs
-    when(successfulHandler.handle(any(InputRow.class))).thenReturn(true);
-    // Construct mock object
-    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
-    // Method Stubs
-    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
-}
+public void setup() {
+    successfulHandler = createMockInputRowHandler(true);
+    unsuccessfulHandler = createMockInputRowHandler(false);
+}
 
 @Test
 public void yieldsNullIfHandledBySecond() {
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void yieldsNullIfHandledBySecond() {
    HandlingInputRowIterator target = createInputRowIterator(unsuccessfulHandler, successfulHandler);
    Assert.assertNull(target.next());
    Mockito.verify(unsuccessfulHandler, Mockito.times(1)).handle(INPUT_ROW1);
    Mockito.verify(successfulHandler, Mockito.times(1)).handle(INPUT_ROW1);
}
@Before
public void setup() {
    // Construct mock object
    successfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
    when(successfulHandler.handle(any(InputRow.class))).thenReturn(true);
    // Construct mock object
    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HandlingInputRowIterator.InputRowHandler createMockInputRowHandler(boolean handleReturnValue) {
    HandlingInputRowIterator.InputRowHandler handler = mock(HandlingInputRowIterator.InputRowHandler.class);
    when(handler.handle(any(InputRow.class))).thenReturn(handleReturnValue);
    return handler;
}
```
</details>

---
#### Test Case ID #druid_Test_46_3
#### Test Case Name: `hasNext`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\HandlingInputRowIteratorTest.java`)
#### Mock Object Variable Name: `unsuccessfulHandler`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    // Construct mock object
-    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
-    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
+    unsuccessfulHandler = createMockInputRowHandler(false);
}
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void hasNext() {
    HandlingInputRowIterator target = createInputRowIterator(unsuccessfulHandler, unsuccessfulHandler);
    Assert.assertTrue(target.hasNext());
    Mockito.verify(unsuccessfulHandler, Mockito.never()).handle(INPUT_ROW1);
}
@Before
public void setup() {
    // Construct mock object
    successfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
    when(successfulHandler.handle(any(InputRow.class))).thenReturn(true);
    // Construct mock object
    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HandlingInputRowIterator.InputRowHandler createMockInputRowHandler(boolean handleReturnValue) {
    HandlingInputRowIterator.InputRowHandler handler = mock(HandlingInputRowIterator.InputRowHandler.class);
    when(handler.handle(any(InputRow.class))).thenReturn(handleReturnValue);
    return handler;
}
```
</details>

---
#### Test Case ID #druid_Test_46_4
#### Test Case Name: `yieldsNextIfUnhandled`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\HandlingInputRowIteratorTest.java`)
#### Mock Object Variable Name: `unsuccessfulHandler`
<summary>Suggested Diff</summary>

```diff
--- Original.java
+++ Refactored.java
@@
 @Before
 public void setup() {
     // Construct mock object
     successfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
     // Method Stubs
     when(successfulHandler.handle(any(InputRow.class))).thenReturn(true);
-    // Construct mock object
-    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
-    // Method Stubs
-    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
+    // Construct mock object and method stubs for unsuccessfulHandler
+    unsuccessfulHandler = createMockInputRowHandler(false);
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Before
public void setup() {
    // Construct mock object
    successfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
    when(successfulHandler.handle(any(InputRow.class))).thenReturn(true);
    // Construct mock object
    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
}
@Test
public void yieldsNextIfUnhandled() {
    HandlingInputRowIterator target = createInputRowIterator(unsuccessfulHandler, unsuccessfulHandler);
    Assert.assertEquals(INPUT_ROW1, target.next());
    Mockito.verify(unsuccessfulHandler, Mockito.times(2)).handle(INPUT_ROW1);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HandlingInputRowIterator.InputRowHandler createMockInputRowHandler(boolean handleReturnValue) {
    HandlingInputRowIterator.InputRowHandler handler = mock(HandlingInputRowIterator.InputRowHandler.class);
    when(handler.handle(any(InputRow.class))).thenReturn(handleReturnValue);
    return handler;
}
```
</details>

---
#### Test Case ID #druid_Test_46_5
#### Test Case Name: `yieldsNullIfHandledByFirst`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\HandlingInputRowIteratorTest.java`)
#### Mock Object Variable Name: `unsuccessfulHandler`
<summary>Suggested Diff</summary>

```diff
--- OriginalTest.java
+++ RefactoredTest.java
@@
 @Before
 public void setup() {
     // Construct mock object
     successfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
     // Method Stubs
     when(successfulHandler.handle(any(InputRow.class))).thenReturn(true);
-    // Construct mock object
-    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
-    // Method Stubs
-    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
+    // Construct mock object
+    unsuccessfulHandler = createMockInputRowHandler(false);
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void yieldsNullIfHandledByFirst() {
    HandlingInputRowIterator target = createInputRowIterator(successfulHandler, unsuccessfulHandler);
    Assert.assertNull(target.next());
    Mockito.verify(successfulHandler, Mockito.times(1)).handle(INPUT_ROW1);
    Mockito.verify(unsuccessfulHandler, Mockito.never()).handle(INPUT_ROW1);
}
@Before
public void setup() {
    // Construct mock object
    successfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
    when(successfulHandler.handle(any(InputRow.class))).thenReturn(true);
    // Construct mock object
    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HandlingInputRowIterator.InputRowHandler createMockInputRowHandler(boolean handleReturnValue) {
    HandlingInputRowIterator.InputRowHandler handler = mock(HandlingInputRowIterator.InputRowHandler.class);
    when(handler.handle(any(InputRow.class))).thenReturn(handleReturnValue);
    return handler;
}
```
</details>

---
#### Test Case ID #druid_Test_46_6
#### Test Case Name: `yieldsNullIfHandledBySecond`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\data\input\HandlingInputRowIteratorTest.java`)
#### Mock Object Variable Name: `unsuccessfulHandler`
<summary>Suggested Diff</summary>

```diff
--- OriginalTest.java
+++ RefactoredTest.java
@@
    // Construct mock object
-    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
-    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
+    unsuccessfulHandler = createMockInputRowHandler(false);
 }

```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void yieldsNullIfHandledBySecond() {
    HandlingInputRowIterator target = createInputRowIterator(unsuccessfulHandler, successfulHandler);
    Assert.assertNull(target.next());
    Mockito.verify(unsuccessfulHandler, Mockito.times(1)).handle(INPUT_ROW1);
    Mockito.verify(successfulHandler, Mockito.times(1)).handle(INPUT_ROW1);
}
@Before
public void setup() {
    // Construct mock object
    successfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
    when(successfulHandler.handle(any(InputRow.class))).thenReturn(true);
    // Construct mock object
    unsuccessfulHandler = mock(HandlingInputRowIterator.InputRowHandler.class);
    // Method Stubs
    when(unsuccessfulHandler.handle(any(InputRow.class))).thenReturn(false);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static HandlingInputRowIterator.InputRowHandler createMockInputRowHandler(boolean handleReturnValue) {
    HandlingInputRowIterator.InputRowHandler handler = mock(HandlingInputRowIterator.InputRowHandler.class);
    when(handler.handle(any(InputRow.class))).thenReturn(handleReturnValue);
    return handler;
}
```
</details>

---
## Mock Clone Instance #druid_MCI_47
- **Scope**: class level
- **Mocked Class**: `org.apache.druid.timeline.DataSegment`
- **Test Case Count**: 10
- **MO Count**: 12

### Reusable Method
```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_47_1
#### Test Case Name: `testArchiveSegmentWithType`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentArchiverTest.java`)
#### Mock Object Variable Name: `segment`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final DataSegmentArchiver archiver = Mockito.mock(DataSegmentArchiver.class);
-    final DataSegment segment = Mockito.mock(DataSegment.class);
-    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
+    final DataSegment segment = MockDataSegment.createMockDataSegment("sane");
    final Injector injector = createInjector(archiver);
    final DataSegmentArchiver segmentArchiver = injector.getInstance(OmniDataSegmentArchiver.class);
    segmentArchiver.archive(segment);
    Mockito.verify(archiver, Mockito.times(1)).archive(segment);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testArchiveSegmentWithType() throws SegmentLoadingException {
    final DataSegmentArchiver archiver = Mockito.mock(DataSegmentArchiver.class);
    final DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    final Injector injector = createInjector(archiver);
    final DataSegmentArchiver segmentArchiver = injector.getInstance(OmniDataSegmentArchiver.class);
    segmentArchiver.archive(segment);
    Mockito.verify(archiver, Mockito.times(1)).archive(segment);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```
</details>

---
#### Test Case ID #druid_Test_47_2
#### Test Case Name: `testArchiveSegmentUnknowType`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentArchiverTest.java`)
#### Mock Object Variable Name: `segment`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testArchiveSegmentUnknowType() {
-    final DataSegment segment = Mockito.mock(DataSegment.class);
-    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "unknown-type"));
+    final DataSegment segment = MockDataSegment.createMockDataSegment("unknown-type");
     final Injector injector = createInjector(null);
     final OmniDataSegmentArchiver segmentArchiver = injector.getInstance(OmniDataSegmentArchiver.class);
     Assert.assertThrows("Unknown loader type[unknown-type]. Known types are [explode]", SegmentLoadingException.class, () -> segmentArchiver.archive(segment));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testArchiveSegmentUnknowType() {
    final DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "unknown-type"));
    final Injector injector = createInjector(null);
    final OmniDataSegmentArchiver segmentArchiver = injector.getInstance(OmniDataSegmentArchiver.class);
    Assert.assertThrows("Unknown loader type[unknown-type]. Known types are [explode]", SegmentLoadingException.class, () -> segmentArchiver.archive(segment));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```
</details>

---
#### Test Case ID #druid_Test_47_3
#### Test Case Name: `testBadSegmentArchiverAccessException`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentArchiverTest.java`)
#### Mock Object Variable Name: `segment`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void testBadSegmentArchiverAccessException() {
-    final DataSegment segment = Mockito.mock(DataSegment.class);
-    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "bad"));
+    final DataSegment segment = MockDataSegment.createMockDataSegment("bad");
     final Injector injector = createInjector(null);
     final OmniDataSegmentArchiver segmentArchiver = injector.getInstance(OmniDataSegmentArchiver.class);
     Assert.assertThrows("BadSegmentArchiver must not have been initialized", RuntimeException.class, () -> segmentArchiver.archive(segment));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testBadSegmentArchiverAccessException() {
    final DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "bad"));
    final Injector injector = createInjector(null);
    final OmniDataSegmentArchiver segmentArchiver = injector.getInstance(OmniDataSegmentArchiver.class);
    Assert.assertThrows("BadSegmentArchiver must not have been initialized", RuntimeException.class, () -> segmentArchiver.archive(segment));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```
</details>

---
#### Test Case ID #druid_Test_47_4
#### Test Case Name: `testKillSegmentWithType`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentKillerTest.java`)
#### Mock Object Variable Name: `segment`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final DataSegmentKiller killer = Mockito.mock(DataSegmentKiller.class);
-    final DataSegment segment = Mockito.mock(DataSegment.class);
-    Mockito.when(segment.isTombstone()).thenReturn(false);
-    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
+    final DataSegment segment = MockDataSegment.createMockDataSegment("sane");
+    Mockito.when(segment.isTombstone()).thenReturn(false);
    final Injector injector = createInjector(killer);
    final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
    segmentKiller.kill(segment);
    Mockito.verify(killer, Mockito.times(1)).kill(segment);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testKillSegmentWithType() throws SegmentLoadingException {
    final DataSegmentKiller killer = Mockito.mock(DataSegmentKiller.class);
    final DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.isTombstone()).thenReturn(false);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    final Injector injector = createInjector(killer);
    final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
    segmentKiller.kill(segment);
    Mockito.verify(killer, Mockito.times(1)).kill(segment);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```
</details>

---
#### Test Case ID #druid_Test_47_5
#### Test Case Name: `testKillSegmentUnknowType`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentKillerTest.java`)
#### Mock Object Variable Name: `segment`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testKillSegmentUnknowType() {
-    final DataSegment segment = Mockito.mock(DataSegment.class);
-    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "unknown-type"));
+    final DataSegment segment = MockDataSegment.createMockDataSegment("unknown-type");
    final Injector injector = createInjector(null);
    final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
    Assert.assertThrows("Unknown loader type[unknown-type]. Known types are [explode]", SegmentLoadingException.class, () -> segmentKiller.kill(segment));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testKillSegmentUnknowType() {
    final DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "unknown-type"));
    final Injector injector = createInjector(null);
    final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
    Assert.assertThrows("Unknown loader type[unknown-type]. Known types are [explode]", SegmentLoadingException.class, () -> segmentKiller.kill(segment));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```
</details>

---
#### Test Case ID #druid_Test_47_6
#### Test Case Name: `testBadSegmentKillerAccessException`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentKillerTest.java`)
#### Mock Object Variable Name: `segment`
<summary>Suggested Diff</summary>

```diff
@@
     final DataSegment segment = Mockito.mock(DataSegment.class);
-    final DataSegment segment = Mockito.mock(DataSegment.class);
-    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "bad"));
+    final DataSegment segment = MockDataSegment.createMockDataSegment("bad");
     final Injector injector = createInjector(null);
     final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
     Assert.assertThrows("BadSegmentKiller must not have been initialized", RuntimeException.class, () -> segmentKiller.kill(segment));
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testBadSegmentKillerAccessException() {
    final DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "bad"));
    final Injector injector = createInjector(null);
    final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
    Assert.assertThrows("BadSegmentKiller must not have been initialized", RuntimeException.class, () -> segmentKiller.kill(segment));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```
</details>

---
#### Test Case ID #druid_Test_47_7
#### Test Case Name: `testKillMultipleSegmentsWithType`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentKillerTest.java`)
#### Mock Object Variable Name: `segment1`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
     final DataSegmentKiller killerSane = Mockito.mock(DataSegmentKiller.class);
     final DataSegmentKiller killerSaneTwo = Mockito.mock(DataSegmentKiller.class);
-    final DataSegment segment1 = Mockito.mock(DataSegment.class);
+    final DataSegment segment1 = MockDataSegment.createMockDataSegment("sane");
     final DataSegment segment2 = Mockito.mock(DataSegment.class);
     final DataSegment segment3 = Mockito.mock(DataSegment.class);
+    Mockito.when(segment1.isTombstone()).thenReturn(false);
     Mockito.when(segment2.isTombstone()).thenReturn(false);
     Mockito.when(segment2.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
     Mockito.when(segment3.isTombstone()).thenReturn(false);
     Mockito.when(segment3.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane_2"));
     final Injector injector = createInjectorFromMap(ImmutableMap.of("sane", killerSane, "sane_2", killerSaneTwo));
     final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
     segmentKiller.kill(ImmutableList.of(segment1, segment2, segment3));
     Mockito.verify(killerSane, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment1, segment2)));
     Mockito.verify(killerSaneTwo, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment3)));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testKillMultipleSegmentsWithType() throws SegmentLoadingException {
    final DataSegmentKiller killerSane = Mockito.mock(DataSegmentKiller.class);
    final DataSegmentKiller killerSaneTwo = Mockito.mock(DataSegmentKiller.class);
    final DataSegment segment1 = Mockito.mock(DataSegment.class);
    final DataSegment segment2 = Mockito.mock(DataSegment.class);
    final DataSegment segment3 = Mockito.mock(DataSegment.class);
    Mockito.when(segment1.isTombstone()).thenReturn(false);
    Mockito.when(segment1.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    Mockito.when(segment2.isTombstone()).thenReturn(false);
    Mockito.when(segment2.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    Mockito.when(segment3.isTombstone()).thenReturn(false);
    Mockito.when(segment3.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane_2"));
    final Injector injector = createInjectorFromMap(ImmutableMap.of("sane", killerSane, "sane_2", killerSaneTwo));
    final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
    segmentKiller.kill(ImmutableList.of(segment1, segment2, segment3));
    Mockito.verify(killerSane, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment1, segment2)));
    Mockito.verify(killerSaneTwo, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment3)));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```
</details>

---
#### Test Case ID #druid_Test_47_8
#### Test Case Name: `testKillMultipleSegmentsWithType`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentKillerTest.java`)
#### Mock Object Variable Name: `segment2`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final DataSegmentKiller killerSane = Mockito.mock(DataSegmentKiller.class);
    final DataSegmentKiller killerSaneTwo = Mockito.mock(DataSegmentKiller.class);
    final DataSegment segment1 = Mockito.mock(DataSegment.class);
-    final DataSegment segment2 = Mockito.mock(DataSegment.class);
    final DataSegment segment3 = Mockito.mock(DataSegment.class);
    Mockito.when(segment1.isTombstone()).thenReturn(false);
    Mockito.when(segment1.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
+    final DataSegment segment2 = MockDataSegment.createMockDataSegment("sane");
    Mockito.when(segment2.isTombstone()).thenReturn(false);
-    Mockito.when(segment2.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    Mockito.when(segment3.isTombstone()).thenReturn(false);
    Mockito.when(segment3.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane_2"));
    final Injector injector = createInjectorFromMap(ImmutableMap.of("sane", killerSane, "sane_2", killerSaneTwo));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testKillMultipleSegmentsWithType() throws SegmentLoadingException {
    final DataSegmentKiller killerSane = Mockito.mock(DataSegmentKiller.class);
    final DataSegmentKiller killerSaneTwo = Mockito.mock(DataSegmentKiller.class);
    final DataSegment segment1 = Mockito.mock(DataSegment.class);
    final DataSegment segment2 = Mockito.mock(DataSegment.class);
    final DataSegment segment3 = Mockito.mock(DataSegment.class);
    Mockito.when(segment1.isTombstone()).thenReturn(false);
    Mockito.when(segment1.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    Mockito.when(segment2.isTombstone()).thenReturn(false);
    Mockito.when(segment2.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    Mockito.when(segment3.isTombstone()).thenReturn(false);
    Mockito.when(segment3.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane_2"));
    final Injector injector = createInjectorFromMap(ImmutableMap.of("sane", killerSane, "sane_2", killerSaneTwo));
    final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
    segmentKiller.kill(ImmutableList.of(segment1, segment2, segment3));
    Mockito.verify(killerSane, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment1, segment2)));
    Mockito.verify(killerSaneTwo, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment3)));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```
</details>

---
#### Test Case ID #druid_Test_47_9
#### Test Case Name: `testKillMultipleSegmentsWithType`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentKillerTest.java`)
#### Mock Object Variable Name: `segment3`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final DataSegment segment2 = Mockito.mock(DataSegment.class);
-    final DataSegment segment3 = Mockito.mock(DataSegment.class);
+    final DataSegment segment3 = MockDataSegment.createMockDataSegment("sane_2");
    Mockito.when(segment1.isTombstone()).thenReturn(false);
    Mockito.when(segment1.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    Mockito.when(segment2.isTombstone()).thenReturn(false);
    Mockito.when(segment2.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
-    Mockito.when(segment3.isTombstone()).thenReturn(false);
-    Mockito.when(segment3.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane_2"));
+    Mockito.when(segment3.isTombstone()).thenReturn(false);
    final Injector injector = createInjectorFromMap(ImmutableMap.of("sane", killerSane, "sane_2", killerSaneTwo));
    final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
    segmentKiller.kill(ImmutableList.of(segment1, segment2, segment3));
    Mockito.verify(killerSane, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment1, segment2)));
    Mockito.verify(killerSaneTwo, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment3)));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testKillMultipleSegmentsWithType() throws SegmentLoadingException {
    final DataSegmentKiller killerSane = Mockito.mock(DataSegmentKiller.class);
    final DataSegmentKiller killerSaneTwo = Mockito.mock(DataSegmentKiller.class);
    final DataSegment segment1 = Mockito.mock(DataSegment.class);
    final DataSegment segment2 = Mockito.mock(DataSegment.class);
    final DataSegment segment3 = Mockito.mock(DataSegment.class);
    Mockito.when(segment1.isTombstone()).thenReturn(false);
    Mockito.when(segment1.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    Mockito.when(segment2.isTombstone()).thenReturn(false);
    Mockito.when(segment2.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    Mockito.when(segment3.isTombstone()).thenReturn(false);
    Mockito.when(segment3.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane_2"));
    final Injector injector = createInjectorFromMap(ImmutableMap.of("sane", killerSane, "sane_2", killerSaneTwo));
    final OmniDataSegmentKiller segmentKiller = injector.getInstance(OmniDataSegmentKiller.class);
    segmentKiller.kill(ImmutableList.of(segment1, segment2, segment3));
    Mockito.verify(killerSane, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment1, segment2)));
    Mockito.verify(killerSaneTwo, Mockito.times(1)).kill((List<DataSegment>) argThat(containsInAnyOrder(segment3)));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```
</details>

---
#### Test Case ID #druid_Test_47_10
#### Test Case Name: `testMoveSegmentWithType`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentMoverTest.java`)
#### Mock Object Variable Name: `segment`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final DataSegmentMover mover = Mockito.mock(DataSegmentMover.class);
-    final DataSegment segment = Mockito.mock(DataSegment.class);
-    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
+    final DataSegment segment = MockDataSegment.createMockDataSegment("sane");
    final Injector injector = createInjector(mover);
    final OmniDataSegmentMover segmentMover = injector.getInstance(OmniDataSegmentMover.class);
    segmentMover.move(segment, ImmutableMap.of());
    Mockito.verify(mover, Mockito.times(1)).move(segment, ImmutableMap.of());
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testMoveSegmentWithType() throws SegmentLoadingException {
    final DataSegmentMover mover = Mockito.mock(DataSegmentMover.class);
    final DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "sane"));
    final Injector injector = createInjector(mover);
    final OmniDataSegmentMover segmentMover = injector.getInstance(OmniDataSegmentMover.class);
    segmentMover.move(segment, ImmutableMap.of());
    Mockito.verify(mover, Mockito.times(1)).move(segment, ImmutableMap.of());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```
</details>

---
#### Test Case ID #druid_Test_47_11
#### Test Case Name: `testMoveSegmentUnknownType`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentMoverTest.java`)
#### Mock Object Variable Name: `segment`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testMoveSegmentUnknownType() {
-    final DataSegment segment = Mockito.mock(DataSegment.class);
-    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "unknown-type"));
+    final DataSegment segment = MockDataSegment.createMockDataSegment("unknown-type");
    final Injector injector = createInjector(null);
    final OmniDataSegmentMover segmentMover = injector.getInstance(OmniDataSegmentMover.class);
    Assert.assertThrows("Unknown loader type[unknown-type]. Known types are [explode]", SegmentLoadingException.class, () -> segmentMover.move(segment, ImmutableMap.of()));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testMoveSegmentUnknownType() {
    final DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "unknown-type"));
    final Injector injector = createInjector(null);
    final OmniDataSegmentMover segmentMover = injector.getInstance(OmniDataSegmentMover.class);
    Assert.assertThrows("Unknown loader type[unknown-type]. Known types are [explode]", SegmentLoadingException.class, () -> segmentMover.move(segment, ImmutableMap.of()));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```
</details>

---
#### Test Case ID #druid_Test_47_12
#### Test Case Name: `testBadSegmentMoverAccessException`(File: `C:\Java_projects\Apache\druid\server\src\test\java\org\apache\druid\segment\loading\OmniDataSegmentMoverTest.java`)
#### Mock Object Variable Name: `segment`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
public void testBadSegmentMoverAccessException() {
-    final DataSegment segment = Mockito.mock(DataSegment.class);
-    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "bad"));
+    final DataSegment segment = MockDataSegment.createMockDataSegment("bad");
    final Injector injector = createInjector(null);
    final OmniDataSegmentMover segmentMover = injector.getInstance(OmniDataSegmentMover.class);
    Assert.assertThrows("BadSegmentMover must not have been initialized", RuntimeException.class, () -> segmentMover.move(segment, null));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
public void testBadSegmentMoverAccessException() {
    final DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", "bad"));
    final Injector injector = createInjector(null);
    final OmniDataSegmentMover segmentMover = injector.getInstance(OmniDataSegmentMover.class);
    Assert.assertThrows("BadSegmentMover must not have been initialized", RuntimeException.class, () -> segmentMover.move(segment, null));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockDataSegment {
  /**
   * Creates a mock DataSegment with configurable loadSpec map.
   *
   * @param typeValue the value for the "type" key in the loadSpec map
   * @return a Mockito mock of DataSegment with getLoadSpec() stubbed
   */
  public static DataSegment createMockDataSegment(String typeValue) {
    DataSegment segment = Mockito.mock(DataSegment.class);
    Mockito.when(segment.getLoadSpec()).thenReturn(ImmutableMap.of("type", typeValue));
    return segment;
  }
}

```
</details>

---
## Mock Clone Instance #druid_MCI_48
- **Scope**: class level
- **Mocked Class**: `org.apache.druid.segment.column.ColumnCapabilities`
- **Test Case Count**: 6
- **MO Count**: 6

### Reusable Method
```java
public class MockColumnCapabilities {
  public static ColumnCapabilities createMockColumnCapabilities(boolean isNumericReturn) {
    ColumnCapabilities capabilities = Mockito.mock(ColumnCapabilities.class);
    Mockito.doReturn(isNumericReturn).when(capabilities).isNumeric();
    return capabilities;
  }
}
```

### The refactoring details in each test cases
---
#### Test Case ID #druid_Test_48_1
#### Test Case Name: `factorizeVectorForNumericTypeShouldReturnDoubleVectorAggregator`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\aggregation\any\DoubleAnyAggregatorFactoryTest.java`)
#### Mock Object Variable Name: `capabilities`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void factorizeVectorForNumericTypeShouldReturnDoubleVectorAggregator() {
-    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
-    Mockito.doReturn(true).when(capabilities).isNumeric();
+    capabilities = MockColumnCapabilities.createMockColumnCapabilities(true);
+    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
     VectorAggregator aggregator = target.factorizeVector(selectorFactory);
     Assert.assertNotNull(aggregator);
     Assert.assertEquals(DoubleAnyVectorAggregator.class, aggregator.getClass());
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void factorizeVectorForNumericTypeShouldReturnDoubleVectorAggregator() {
    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(true).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(DoubleAnyVectorAggregator.class, aggregator.getClass());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnCapabilities {
  public static ColumnCapabilities createMockColumnCapabilities(boolean isNumericReturn) {
    ColumnCapabilities capabilities = Mockito.mock(ColumnCapabilities.class);
    Mockito.doReturn(isNumericReturn).when(capabilities).isNumeric();
    return capabilities;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_48_2
#### Test Case Name: `factorizeVectorForStringTypeShouldReturnDoubleVectorAggregatorWithNilSelector`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\aggregation\any\DoubleAnyAggregatorFactoryTest.java`)
#### Mock Object Variable Name: `capabilities`
<summary>Suggested Diff</summary>

```diff
@@
     Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
-    Mockito.doReturn(false).when(capabilities).isNumeric();
+    capabilities = MockColumnCapabilities.createMockColumnCapabilities(false);
     VectorAggregator aggregator = target.factorizeVector(selectorFactory);
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void factorizeVectorForStringTypeShouldReturnDoubleVectorAggregatorWithNilSelector() {
    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(false).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(NullHandling.defaultDoubleValue(), aggregator.get(BUFFER, POSITION));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnCapabilities {
  public static ColumnCapabilities createMockColumnCapabilities(boolean isNumericReturn) {
    ColumnCapabilities capabilities = Mockito.mock(ColumnCapabilities.class);
    Mockito.doReturn(isNumericReturn).when(capabilities).isNumeric();
    return capabilities;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_48_3
#### Test Case Name: `factorizeVectorForNumericTypeShouldReturnFloatVectorAggregator`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\aggregation\any\FloatAnyAggregatorFactoryTest.java`)
#### Mock Object Variable Name: `capabilities`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void factorizeVectorForNumericTypeShouldReturnFloatVectorAggregator() {
-    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
-    Mockito.doReturn(true).when(capabilities).isNumeric();
+    capabilities = MockColumnCapabilities.createMockColumnCapabilities(true);
+    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
     VectorAggregator aggregator = target.factorizeVector(selectorFactory);
     Assert.assertNotNull(aggregator);
     Assert.assertEquals(FloatAnyVectorAggregator.class, aggregator.getClass());
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void factorizeVectorForNumericTypeShouldReturnFloatVectorAggregator() {
    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(true).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(FloatAnyVectorAggregator.class, aggregator.getClass());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnCapabilities {
  public static ColumnCapabilities createMockColumnCapabilities(boolean isNumericReturn) {
    ColumnCapabilities capabilities = Mockito.mock(ColumnCapabilities.class);
    Mockito.doReturn(isNumericReturn).when(capabilities).isNumeric();
    return capabilities;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_48_4
#### Test Case Name: `factorizeVectorForStringTypeShouldReturnFloatVectorAggregatorWithNilSelector`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\aggregation\any\FloatAnyAggregatorFactoryTest.java`)
#### Mock Object Variable Name: `capabilities`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
 @Test
 public void factorizeVectorForStringTypeShouldReturnFloatVectorAggregatorWithNilSelector() {
-    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
-    Mockito.doReturn(false).when(capabilities).isNumeric();
+    capabilities = MockColumnCapabilities.createMockColumnCapabilities(false);
+    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
     VectorAggregator aggregator = target.factorizeVector(selectorFactory);
     Assert.assertNotNull(aggregator);
     Assert.assertEquals(NullHandling.defaultFloatValue(), aggregator.get(BUFFER, POSITION));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void factorizeVectorForStringTypeShouldReturnFloatVectorAggregatorWithNilSelector() {
    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(false).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(NullHandling.defaultFloatValue(), aggregator.get(BUFFER, POSITION));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnCapabilities {
  public static ColumnCapabilities createMockColumnCapabilities(boolean isNumericReturn) {
    ColumnCapabilities capabilities = Mockito.mock(ColumnCapabilities.class);
    Mockito.doReturn(isNumericReturn).when(capabilities).isNumeric();
    return capabilities;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_48_5
#### Test Case Name: `factorizeVectorWithNumericColumnShouldReturnLongVectorAggregator`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\aggregation\any\LongAnyAggregatorFactoryTest.java`)
#### Mock Object Variable Name: `capabilities`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void factorizeVectorWithNumericColumnShouldReturnLongVectorAggregator() {
-    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
-    Mockito.doReturn(true).when(capabilities).isNumeric();
+    capabilities = MockColumnCapabilities.createMockColumnCapabilities(true);
+    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
     VectorAggregator aggregator = target.factorizeVector(selectorFactory);
     Assert.assertNotNull(aggregator);
     Assert.assertEquals(LongAnyVectorAggregator.class, aggregator.getClass());
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void factorizeVectorWithNumericColumnShouldReturnLongVectorAggregator() {
    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(true).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(LongAnyVectorAggregator.class, aggregator.getClass());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnCapabilities {
  public static ColumnCapabilities createMockColumnCapabilities(boolean isNumericReturn) {
    ColumnCapabilities capabilities = Mockito.mock(ColumnCapabilities.class);
    Mockito.doReturn(isNumericReturn).when(capabilities).isNumeric();
    return capabilities;
  }
}
```
</details>

---
#### Test Case ID #druid_Test_48_6
#### Test Case Name: `factorizeVectorForStringTypeShouldReturnLongVectorAggregatorWithNilSelector`(File: `C:\Java_projects\Apache\druid\processing\src\test\java\org\apache\druid\query\aggregation\any\LongAnyAggregatorFactoryTest.java`)
#### Mock Object Variable Name: `capabilities`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 public void factorizeVectorForStringTypeShouldReturnLongVectorAggregatorWithNilSelector() {
-    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
-    Mockito.doReturn(false).when(capabilities).isNumeric();
+    capabilities = MockColumnCapabilities.createMockColumnCapabilities(false);
+    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
     VectorAggregator aggregator = target.factorizeVector(selectorFactory);
     Assert.assertNotNull(aggregator);
     Assert.assertEquals(NullHandling.defaultLongValue(), aggregator.get(BUFFER, POSITION));
 }
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java

@Test
public void factorizeVectorForStringTypeShouldReturnLongVectorAggregatorWithNilSelector() {
    Mockito.doReturn(capabilities).when(selectorFactory).getColumnCapabilities(FIELD_NAME);
    Mockito.doReturn(false).when(capabilities).isNumeric();
    VectorAggregator aggregator = target.factorizeVector(selectorFactory);
    Assert.assertNotNull(aggregator);
    Assert.assertEquals(NullHandling.defaultLongValue(), aggregator.get(BUFFER, POSITION));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
public class MockColumnCapabilities {
  public static ColumnCapabilities createMockColumnCapabilities(boolean isNumericReturn) {
    ColumnCapabilities capabilities = Mockito.mock(ColumnCapabilities.class);
    Mockito.doReturn(isNumericReturn).when(capabilities).isNumeric();
    return capabilities;
  }
}
```
</details>

---