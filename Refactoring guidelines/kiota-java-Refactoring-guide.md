# A guide to Refactoring mock clones in kiota-java

## Mock Clone Instance #kiota-java_MCI_1
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.serialization.SerializationWriter`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
private static SerializationWriter createMockSerializationWriter(ByteArrayInputStream serializedContent) {
    SerializationWriter mockSerializationWriter = mock(SerializationWriter.class);
    when(mockSerializationWriter.getSerializedContent()).thenReturn(serializedContent);
    return mockSerializationWriter;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_1_1
#### Test Case Name: `serializesObject`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\serialization\SerializationHelpersTest.java`)
#### Mock Object Variable Name: `mockSerializationWriter`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
void serializesObject() throws IOException {
-    final var mockSerializationWriter = mock(SerializationWriter.class);
-    when(mockSerializationWriter.getSerializedContent()).thenReturn(new ByteArrayInputStream("{'id':'123'}".getBytes(_charset)));
+    final var mockSerializationWriter = createMockSerializationWriter(new ByteArrayInputStream("{'id':'123'}".getBytes(_charset)));
    final var mockSerializationWriterFactory = mock(SerializationWriterFactory.class);
    when(mockSerializationWriterFactory.getSerializationWriter(_jsonContentType)).thenReturn(mockSerializationWriter);
    SerializationWriterFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockSerializationWriterFactory);
    final var result = KiotaSerialization.serializeAsString(_jsonContentType, new TestEntity() {

        {
            setId("123");
        }
    });
    assertEquals("{'id':'123'}", result);
    verify(mockSerializationWriter, times(1)).writeObjectValue(eq(""), any(Parsable.class));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void serializesObject() throws IOException {
    final var mockSerializationWriter = mock(SerializationWriter.class);
    when(mockSerializationWriter.getSerializedContent()).thenReturn(new ByteArrayInputStream("{'id':'123'}".getBytes(_charset)));
    final var mockSerializationWriterFactory = mock(SerializationWriterFactory.class);
    when(mockSerializationWriterFactory.getSerializationWriter(_jsonContentType)).thenReturn(mockSerializationWriter);
    SerializationWriterFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockSerializationWriterFactory);
    final var result = KiotaSerialization.serializeAsString(_jsonContentType, new TestEntity() {

        {
            setId("123");
        }
    });
    assertEquals("{'id':'123'}", result);
    verify(mockSerializationWriter, times(1)).writeObjectValue(eq(""), any(Parsable.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static SerializationWriter createMockSerializationWriter(ByteArrayInputStream serializedContent) {
    SerializationWriter mockSerializationWriter = mock(SerializationWriter.class);
    when(mockSerializationWriter.getSerializedContent()).thenReturn(serializedContent);
    return mockSerializationWriter;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_1_2
#### Test Case Name: `serializesObjectCollection`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\serialization\SerializationHelpersTest.java`)
#### Mock Object Variable Name: `mockSerializationWriter`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
void serializesObjectCollection() throws IOException {
-    final var mockSerializationWriter = mock(SerializationWriter.class);
-    when(mockSerializationWriter.getSerializedContent()).thenReturn(new ByteArrayInputStream("[{\'id\':\'123\'}]".getBytes(_charset)));
+    final var mockSerializationWriter = createMockSerializationWriter(new ByteArrayInputStream("[{\'id\':\'123\'}]".getBytes(_charset)));
    final var mockSerializationWriterFactory = mock(SerializationWriterFactory.class);
    when(mockSerializationWriterFactory.getSerializationWriter(_jsonContentType)).thenReturn(mockSerializationWriter);
    SerializationWriterFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockSerializationWriterFactory);
    final var result = KiotaSerialization.serializeAsString(_jsonContentType, new ArrayList<>() {

        {
            add(new TestEntity() {

                {
                    setId("123");
                }
            });
        }
    });
    assertEquals("[{\'id\':\'123\'}]", result);
    verify(mockSerializationWriter, times(1)).writeCollectionOfObjectValues(eq(""), any());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void serializesObjectCollection() throws IOException {
    final var mockSerializationWriter = mock(SerializationWriter.class);
    when(mockSerializationWriter.getSerializedContent()).thenReturn(new ByteArrayInputStream("[{'id':'123'}]".getBytes(_charset)));
    final var mockSerializationWriterFactory = mock(SerializationWriterFactory.class);
    when(mockSerializationWriterFactory.getSerializationWriter(_jsonContentType)).thenReturn(mockSerializationWriter);
    SerializationWriterFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockSerializationWriterFactory);
    final var result = KiotaSerialization.serializeAsString(_jsonContentType, new ArrayList<>() {

        {
            add(new TestEntity() {

                {
                    setId("123");
                }
            });
        }
    });
    assertEquals("[{'id':'123'}]", result);
    verify(mockSerializationWriter, times(1)).writeCollectionOfObjectValues(eq(""), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static SerializationWriter createMockSerializationWriter(ByteArrayInputStream serializedContent) {
    SerializationWriter mockSerializationWriter = mock(SerializationWriter.class);
    when(mockSerializationWriter.getSerializedContent()).thenReturn(serializedContent);
    return mockSerializationWriter;
}
```
</details>

---
## Mock Clone Instance #kiota-java_MCI_2
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.serialization.SerializationWriter`
- **Test Case Count**: 4
- **MO Count**: 4

### Reusable Method
```java
// === Declare in class scope ===
private SerializationWriter writer;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    writer = mock(SerializationWriter.class);
}

// === Replace local variable in test with ===
writer;

```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_2_1
#### Test Case Name: `requiresRequestAdapter`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\MultiPartBodyTest.java`)
#### Mock Object Variable Name: `writer`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void requiresRequestAdapter() {
     final MultipartBody multipartBody = new MultipartBody();
-    final SerializationWriter writer = mock(SerializationWriter.class);
+    // removed local mock; replaced with global field `writer`
     assertThrows(IllegalStateException.class, () -> multipartBody.serialize(writer));
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void requiresRequestAdapter() {
    final MultipartBody multipartBody = new MultipartBody();
    final SerializationWriter writer = mock(SerializationWriter.class);
    assertThrows(IllegalStateException.class, () -> multipartBody.serialize(writer));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private SerializationWriter writer;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    writer = mock(SerializationWriter.class);
}

// === Replace local variable in test with ===
writer;

```
</details>

---
#### Test Case ID #kiota-java_Test_2_2
#### Test Case Name: `requiresPartsForSerialization`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\MultiPartBodyTest.java`)
#### Mock Object Variable Name: `writer`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void requiresPartsForSerialization() {
     final MultipartBody multipartBody = new MultipartBody();
-    final SerializationWriter writer = mock(SerializationWriter.class);
+    // removed local mock; replaced with global field `writer`
     final RequestAdapter requestAdapter = mock(RequestAdapter.class);
     multipartBody.requestAdapter = requestAdapter;
     assertThrows(IllegalStateException.class, () -> multipartBody.serialize(writer));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void requiresPartsForSerialization() {
    final MultipartBody multipartBody = new MultipartBody();
    final SerializationWriter writer = mock(SerializationWriter.class);
    final RequestAdapter requestAdapter = mock(RequestAdapter.class);
    multipartBody.requestAdapter = requestAdapter;
    assertThrows(IllegalStateException.class, () -> multipartBody.serialize(writer));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private SerializationWriter writer;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    writer = mock(SerializationWriter.class);
}

// === Replace local variable in test with ===
writer;

```
</details>

---
#### Test Case ID #kiota-java_Test_2_3
#### Test Case Name: `notAddFilename`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\MultiPartBodyTest.java`)
#### Mock Object Variable Name: `writer`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void notAddFilename() {
     final MultipartBody multipartBody = new MultipartBody();
-    final SerializationWriter writer = mock(SerializationWriter.class);
+    // removed local mock; replaced with global field `writer`
     multipartBody.requestAdapter = mock(RequestAdapter.class);
     multipartBody.addOrReplacePart("foo", "bar", "baz");
-    multipartBody.serialize(writer);
+    multipartBody.serialize(writer);
-    verify(writer).writeStringValue("Content-Disposition", "form-data; name=\"foo\""");
+    verify(writer).writeStringValue("Content-Disposition", "form-data; name=\"foo\""");
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void notAddFilename() {
    final MultipartBody multipartBody = new MultipartBody();
    final SerializationWriter writer = mock(SerializationWriter.class);
    multipartBody.requestAdapter = mock(RequestAdapter.class);
    multipartBody.addOrReplacePart("foo", "bar", "baz");
    multipartBody.serialize(writer);
    verify(writer).writeStringValue("Content-Disposition", "form-data; name=\"foo\"");
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private SerializationWriter writer;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    writer = mock(SerializationWriter.class);
}

// === Replace local variable in test with ===
writer;

```
</details>

---
#### Test Case ID #kiota-java_Test_2_4
#### Test Case Name: `addFilename`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\MultiPartBodyTest.java`)
#### Mock Object Variable Name: `writer`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void addFilename() {
     final MultipartBody multipartBody = new MultipartBody();
-    final SerializationWriter writer = mock(SerializationWriter.class);
+    // removed local mock; replaced with global field `writer`
     multipartBody.requestAdapter = mock(RequestAdapter.class);
     multipartBody.addOrReplacePart("foo", "bar", "baz", "image.png");
-    multipartBody.serialize(writer);
+    multipartBody.serialize(writer);
-    verify(writer).writeStringValue("Content-Disposition", "form-data; name=\"foo\"; filename=\"image.png\"");
+    verify(writer).writeStringValue("Content-Disposition", "form-data; name=\"foo\"; filename=\"image.png\"");
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void addFilename() {
    final MultipartBody multipartBody = new MultipartBody();
    final SerializationWriter writer = mock(SerializationWriter.class);
    multipartBody.requestAdapter = mock(RequestAdapter.class);
    multipartBody.addOrReplacePart("foo", "bar", "baz", "image.png");
    multipartBody.serialize(writer);
    verify(writer).writeStringValue("Content-Disposition", "form-data; name=\"foo\"; filename=\"image.png\"");
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private SerializationWriter writer;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    writer = mock(SerializationWriter.class);
}

// === Replace local variable in test with ===
writer;

```
</details>

---
## Mock Clone Instance #kiota-java_MCI_3
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.serialization.SerializationWriter`
- **Test Case Count**: 5
- **MO Count**: 5

### Reusable Method
```java
// === Declare in class scope ===
private SerializationWriter writerMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    writerMock = mock(SerializationWriter.class);
}

// === Replace local variable in test with ===
writerMock;

```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_3_1
#### Test Case Name: `SetsParsableContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `writerMock`
<summary>Suggested Diff</summary>

```diff
@@
     requestInfo.httpMethod = HttpMethod.POST;
     requestInfo.urlTemplate = "http://localhost/users";
-    final SerializationWriter writerMock = mock(SerializationWriter.class);
+    // removed local mock; replaced with global field `writerMock`
     final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
     when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
     final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
     when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
     requestInfo.setContentFromParsable(requestAdapterMock, "application/json", new TestEntity());
-    verify(writerMock, times(1)).writeObjectValue(any(), any(TestEntity.class));
+    verify(writerMock, times(1)).writeObjectValue(any(), any(TestEntity.class));
-    verify(writerMock, never()).writeCollectionOfObjectValues(anyString(), any(ArrayList.class));
+    verify(writerMock, never()).writeCollectionOfObjectValues(anyString(), any(ArrayList.class));
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsParsableContent() {
    // Arrange as the request builders would
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/users";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    requestInfo.setContentFromParsable(requestAdapterMock, "application/json", new TestEntity());
    verify(writerMock, times(1)).writeObjectValue(any(), any(TestEntity.class));
    verify(writerMock, never()).writeCollectionOfObjectValues(anyString(), any(ArrayList.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private SerializationWriter writerMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    writerMock = mock(SerializationWriter.class);
}

// === Replace local variable in test with ===
writerMock;

```
</details>

---
#### Test Case ID #kiota-java_Test_3_2
#### Test Case Name: `SetsParsableContentCollection`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `writerMock`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void SetsParsableContentCollection() {
     // Arrange as the request builders would
     final RequestInformation requestInfo = new RequestInformation();
     requestInfo.httpMethod = HttpMethod.POST;
     requestInfo.urlTemplate = "http://localhost/users";
-    final SerializationWriter writerMock = mock(SerializationWriter.class);
+    // removed local mock; replaced with global field `writerMock`
     final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
     when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
     final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
     when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
     requestInfo.setContentFromParsable(requestAdapterMock, "application/json", new TestEntity[] { new TestEntity() });
     verify(writerMock, never()).writeObjectValue(any(), any(TestEntity.class));
     verify(writerMock, times(1)).writeCollectionOfObjectValues(any(), any(Iterable.class));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsParsableContentCollection() {
    // Arrange as the request builders would
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/users";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    requestInfo.setContentFromParsable(requestAdapterMock, "application/json", new TestEntity[] { new TestEntity() });
    verify(writerMock, never()).writeObjectValue(any(), any(TestEntity.class));
    verify(writerMock, times(1)).writeCollectionOfObjectValues(any(), any(Iterable.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private SerializationWriter writerMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    writerMock = mock(SerializationWriter.class);
}

// === Replace local variable in test with ===
writerMock;

```
</details>

---
#### Test Case ID #kiota-java_Test_3_3
#### Test Case Name: `SetsScalarContentCollection`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `writerMock`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void SetsScalarContentCollection() {
     // Arrange as the request builders would
     final RequestInformation requestInfo = new RequestInformation();
     requestInfo.httpMethod = HttpMethod.POST;
     requestInfo.urlTemplate = "http://localhost/users";
-    final SerializationWriter writerMock = mock(SerializationWriter.class);
+    // removed local mock; replaced with global field `writerMock`
     final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
     when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
     final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
     when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
     requestInfo.setContentFromScalarCollection(requestAdapterMock, "application/json", new String[] { "foo" });
     verify(writerMock, never()).writeStringValue(any(), anyString());
     verify(writerMock, times(1)).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsScalarContentCollection() {
    // Arrange as the request builders would
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/users";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    requestInfo.setContentFromScalarCollection(requestAdapterMock, "application/json", new String[] { "foo" });
    verify(writerMock, never()).writeStringValue(any(), anyString());
    verify(writerMock, times(1)).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private SerializationWriter writerMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    writerMock = mock(SerializationWriter.class);
}

// === Replace local variable in test with ===
writerMock;

```
</details>

---
#### Test Case ID #kiota-java_Test_3_4
#### Test Case Name: `SetsScalarContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `writerMock`
<summary>Suggested Diff</summary>

```diff
@@
     requestInfo.httpMethod = HttpMethod.POST;
     requestInfo.urlTemplate = "http://localhost/users";
-    final SerializationWriter writerMock = mock(SerializationWriter.class);
+    // removed local mock; replaced with global field `writerMock`
     final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
     when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
     final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
     when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
     requestInfo.setContentFromScalar(requestAdapterMock, "application/json", "foo");
-    verify(writerMock, times(1)).writeStringValue(any(), anyString());
+    verify(writerMock, times(1)).writeStringValue(any(), anyString());
-    verify(writerMock, never()).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
+    verify(writerMock, never()).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsScalarContent() {
    // Arrange as the request builders would
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/users";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    requestInfo.setContentFromScalar(requestAdapterMock, "application/json", "foo");
    verify(writerMock, times(1)).writeStringValue(any(), anyString());
    verify(writerMock, never()).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private SerializationWriter writerMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    writerMock = mock(SerializationWriter.class);
}

// === Replace local variable in test with ===
writerMock;

```
</details>

---
#### Test Case ID #kiota-java_Test_3_5
#### Test Case Name: `SetsBoundaryOnMultipartBody`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `writerMock`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void SetsBoundaryOnMultipartBody() {
-    final SerializationWriter writerMock = mock(SerializationWriter.class);
+    // removed local mock; replaced with global field `writerMock`
     final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
     when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
     final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
     when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
     final MultipartBody multipartBody = new MultipartBody();
     multipartBody.requestAdapter = requestAdapterMock;
     requestInfo.setContentFromParsable(requestAdapterMock, "multipart/form-data", multipartBody);
     assertNotNull(multipartBody.getBoundary());
     assertFalse(multipartBody.getBoundary().isEmpty());
     assertEquals("multipart/form-data; boundary=" + multipartBody.getBoundary(), requestInfo.headers.get("Content-Type").toArray()[0]);
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsBoundaryOnMultipartBody() {
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/{URITemplate}/ParameterMapping?IsCaseSensitive={IsCaseSensitive}";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    final MultipartBody multipartBody = new MultipartBody();
    multipartBody.requestAdapter = requestAdapterMock;
    requestInfo.setContentFromParsable(requestAdapterMock, "multipart/form-data", multipartBody);
    assertNotNull(multipartBody.getBoundary());
    assertFalse(multipartBody.getBoundary().isEmpty());
    assertEquals("multipart/form-data; boundary=" + multipartBody.getBoundary(), requestInfo.headers.get("Content-Type").toArray()[0]);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private SerializationWriter writerMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    writerMock = mock(SerializationWriter.class);
}

// === Replace local variable in test with ===
writerMock;

```
</details>

---
## Mock Clone Instance #kiota-java_MCI_4
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.authentication.AuthenticationProvider`
- **Test Case Count**: 11
- **MO Count**: 11

### Reusable Method
```java
// === Declare in class scope ===
private AuthenticationProvider authenticationProviderMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    authenticationProviderMock = mock(AuthenticationProvider.class);
}

// === Replace local variable in test with ===
authenticationProviderMock

```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_4_1
#### Test Case Name: `postRequestsShouldHaveEmptyBody`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `authenticationProviderMock`
<summary>Suggested Diff</summary>

```diff
@@
 @ParameterizedTest
 @EnumSource(value = HttpMethod.class, names = { "PUT", "POST", "PATCH" })
 void postRequestsShouldHaveEmptyBody(HttpMethod method) throws Exception {
     // Unexpected exception thrown: java.lang.IllegalArgumentException:
     // method POST must have a request body.
-    final AuthenticationProvider authenticationProviderMock = mock(AuthenticationProvider.class);
+    // removed local mock; replaced with global field `authenticationProviderMock`
     final var adapter = new OkHttpRequestAdapter(authenticationProviderMock) {

         public Request test() throws Exception {
             RequestInformation ri = new RequestInformation();
             ri.httpMethod = method;
             ri.urlTemplate = "http://localhost:1234";
             Span span1 = GlobalOpenTelemetry.getTracer("").spanBuilder("").startSpan();
             Span span2 = GlobalOpenTelemetry.getTracer("").spanBuilder("").startSpan();
             return this.getRequestFromRequestInformation(ri, span1, span2);
         }
     };
     final var request = assertDoesNotThrow(() -> adapter.test());
     assertNotNull(request.body());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@ParameterizedTest
@EnumSource(value = HttpMethod.class, names = { "PUT", "POST", "PATCH" })
void postRequestsShouldHaveEmptyBody(HttpMethod method) throws Exception {
    // Unexpected exception thrown: java.lang.IllegalArgumentException:
    // method POST must have a request body.
    final AuthenticationProvider authenticationProviderMock = mock(AuthenticationProvider.class);
    final var adapter = new OkHttpRequestAdapter(authenticationProviderMock) {

        public Request test() throws Exception {
            RequestInformation ri = new RequestInformation();
            ri.httpMethod = method;
            ri.urlTemplate = "http://localhost:1234";
            Span span1 = GlobalOpenTelemetry.getTracer("").spanBuilder("").startSpan();
            Span span2 = GlobalOpenTelemetry.getTracer("").spanBuilder("").startSpan();
            return this.getRequestFromRequestInformation(ri, span1, span2);
        }
    };
    final var request = assertDoesNotThrow(() -> adapter.test());
    assertNotNull(request.body());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AuthenticationProvider authenticationProviderMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    authenticationProviderMock = mock(AuthenticationProvider.class);
}

// === Replace local variable in test with ===
authenticationProviderMock

```
</details>

---
#### Test Case ID #kiota-java_Test_4_2
#### Test Case Name: `sendStreamReturnsUsableStream`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `authenticationProviderMock`
<summary>Suggested Diff</summary>

```diff
@@
 @ParameterizedTest
 @ValueSource(ints = { 200, 201, 202, 203, 206 })
 void sendStreamReturnsUsableStream(int statusCode) throws Exception {
-    final var authenticationProviderMock = mock(AuthenticationProvider.class);
+    // removed local mock; replaced with global field `authenticationProviderMock`
     authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
     final var text = "my-demo-text";
     final var bufferedSource = Okio.buffer(Okio.source(new ByteArrayInputStream(text.getBytes("UTF-8"))));
     final var client = getMockClient(new Response.Builder().code(statusCode).message("OK").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(ResponseBody.create(bufferedSource, MediaType.parse("application/binary"), text.getBytes("UTF-8").length)).build());
     final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, null, null, client);
     final var requestInformation = new RequestInformation() {

         {
             setUri(new URI("https://localhost"));
             httpMethod = HttpMethod.GET;
         }
     };
     InputStream response = null;
     try {
         response = requestAdapter.sendPrimitive(requestInformation, null, InputStream.class);
         assertNotNull(response);
         assertEquals(text, new String(response.readAllBytes(), StandardCharsets.UTF_8));
     } finally {
         if (response != null) {
             response.close();
         }
     }
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@ParameterizedTest
@ValueSource(ints = { 200, 201, 202, 203, 206 })
void sendStreamReturnsUsableStream(int statusCode) throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
    final var text = "my-demo-text";
    final var bufferedSource = Okio.buffer(Okio.source(new ByteArrayInputStream(text.getBytes("UTF-8"))));
    final var client = getMockClient(new Response.Builder().code(statusCode).message("OK").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(ResponseBody.create(bufferedSource, MediaType.parse("application/binary"), text.getBytes("UTF-8").length)).build());
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, null, null, client);
    final var requestInformation = new RequestInformation() {

        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
    InputStream response = null;
    try {
        response = requestAdapter.sendPrimitive(requestInformation, null, InputStream.class);
        assertNotNull(response);
        assertEquals(text, new String(response.readAllBytes(), StandardCharsets.UTF_8));
    } finally {
        if (response != null) {
            response.close();
        }
    }
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AuthenticationProvider authenticationProviderMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    authenticationProviderMock = mock(AuthenticationProvider.class);
}

// === Replace local variable in test with ===
authenticationProviderMock

```
</details>

---
#### Test Case ID #kiota-java_Test_4_3
#### Test Case Name: `sendStreamReturnsNullOnNoContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `authenticationProviderMock`
<summary>Suggested Diff</summary>

```diff
@@
 @ParameterizedTest
 @ValueSource(ints = { 200, 201, 202, 203, 204 })
 void sendStreamReturnsNullOnNoContent(int statusCode) throws Exception {
-    final var authenticationProviderMock = mock(AuthenticationProvider.class);
+    // removed local mock; replaced with global field `authenticationProviderMock`
     authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
     final var client = getMockClient(new Response.Builder().code(statusCode).message("OK").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(null).build());
     final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, null, null, client);
     final var requestInformation = new RequestInformation() {

         {
             setUri(new URI("https://localhost"));
             httpMethod = HttpMethod.GET;
         }
     };
     final var response = requestAdapter.sendPrimitive(requestInformation, null, InputStream.class);
     assertNull(response);
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@ParameterizedTest
@ValueSource(ints = { 200, 201, 202, 203, 204 })
void sendStreamReturnsNullOnNoContent(int statusCode) throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
    final var client = getMockClient(new Response.Builder().code(statusCode).message("OK").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(null).build());
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, null, null, client);
    final var requestInformation = new RequestInformation() {

        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
    final var response = requestAdapter.sendPrimitive(requestInformation, null, InputStream.class);
    assertNull(response);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AuthenticationProvider authenticationProviderMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    authenticationProviderMock = mock(AuthenticationProvider.class);
}

// === Replace local variable in test with ===
authenticationProviderMock

```
</details>

---
#### Test Case ID #kiota-java_Test_4_4
#### Test Case Name: `sendReturnsNullOnNoContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `authenticationProviderMock`
<summary>Suggested Diff</summary>

```diff
@@
 void sendReturnsNullOnNoContent(int statusCode) throws Exception {
-    final var authenticationProviderMock = mock(AuthenticationProvider.class);
+    // removed local mock; replaced with global field `authenticationProviderMock`
     authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
     final var client = getMockClient(new Response.Builder().code(statusCode).message("OK").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(null).build());
     final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, null, null, client);
     final var requestInformation = new RequestInformation() {

         {
             setUri(new URI("https://localhost"));
             httpMethod = HttpMethod.GET;
         }
     };
     final var mockEntity = mock(Parsable.class);
     when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
     final var response = requestAdapter.send(requestInformation, null, (node) -> mockEntity);
     assertNull(response);
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@ParameterizedTest
@ValueSource(ints = { 200, 201, 202, 203, 204, 205 })
void sendReturnsNullOnNoContent(int statusCode) throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
    final var client = getMockClient(new Response.Builder().code(statusCode).message("OK").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(null).build());
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, null, null, client);
    final var requestInformation = new RequestInformation() {

        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
    final var mockEntity = mock(Parsable.class);
    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
    final var response = requestAdapter.send(requestInformation, null, (node) -> mockEntity);
    assertNull(response);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AuthenticationProvider authenticationProviderMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    authenticationProviderMock = mock(AuthenticationProvider.class);
}

// === Replace local variable in test with ===
authenticationProviderMock

```
</details>

---
#### Test Case ID #kiota-java_Test_4_5
#### Test Case Name: `sendReturnsObjectOnContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `authenticationProviderMock`
<summary>Suggested Diff</summary>

```diff
@@
 @ParameterizedTest
 @ValueSource(ints = { 200, 201, 202, 203 })
 void sendReturnsObjectOnContent(int statusCode) throws Exception {
-    final var authenticationProviderMock = mock(AuthenticationProvider.class);
+    // removed local mock; replaced with global field `authenticationProviderMock`
     authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
     final var client = getMockClient(new Response.Builder().code(statusCode).message("OK").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(ResponseBody.create("test".getBytes("UTF-8"), MediaType.parse("application/json"))).build());
     final var requestInformation = new RequestInformation() {

         {
             setUri(new URI("https://localhost"));
             httpMethod = HttpMethod.GET;
         }
     };
     final var mockEntity = mock(Parsable.class);
     when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
     final var mockParseNode = mock(ParseNode.class);
     when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
     final var mockFactory = mock(ParseNodeFactory.class);
     when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
     when(mockFactory.getValidContentType()).thenReturn("application/json");
     final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
     final var response = requestAdapter.send(requestInformation, null, (node) -> mockEntity);
     assertNotNull(response);
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@ParameterizedTest
@ValueSource(ints = { 200, 201, 202, 203 })
void sendReturnsObjectOnContent(int statusCode) throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
    final var client = getMockClient(new Response.Builder().code(statusCode).message("OK").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(ResponseBody.create("test".getBytes("UTF-8"), MediaType.parse("application/json"))).build());
    final var requestInformation = new RequestInformation() {

        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
    final var mockEntity = mock(Parsable.class);
    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
    final var mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
    final var mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn("application/json");
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
    final var response = requestAdapter.send(requestInformation, null, (node) -> mockEntity);
    assertNotNull(response);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AuthenticationProvider authenticationProviderMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    authenticationProviderMock = mock(AuthenticationProvider.class);
}

// === Replace local variable in test with ===
authenticationProviderMock

```
</details>

---
#### Test Case ID #kiota-java_Test_4_6
#### Test Case Name: `throwsAPIException`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `authenticationProviderMock`
<summary>Suggested Diff</summary>

```diff
@@
 @SuppressWarnings("unchecked")
 @ParameterizedTest
 @MethodSource("providesErrorMappings")
 void throwsAPIException(int responseStatusCode, List<String> errorMappingCodes, boolean expectDeserializedException) throws Exception {
-    final var authenticationProviderMock = mock(AuthenticationProvider.class);
+    // removed local mock; replaced with global field `authenticationProviderMock`
     authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
     final var client = getMockClient(new Response.Builder().code(responseStatusCode).message("Not Found").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(ResponseBody.create("test".getBytes("UTF-8"), MediaType.parse("application/json")).header("request-id", "request-id-value").build());
     final var requestInformation = new RequestInformation() {

         {
             setUri(new URI("https://localhost"));
             httpMethod = HttpMethod.GET;
         }
     };
     final var mockEntity = mock(Parsable.class);
     when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
     final var mockParsableFactory = mock(ParsableFactory.class);
     when(mockParsableFactory.create(any(ParseNode.class))).thenReturn(mockEntity);
     final var mockParseNode = mock(ParseNode.class);
     when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
     final var mockFactory = mock(ParseNodeFactory.class);
     when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
     when(mockFactory.getValidContentType()).thenReturn("application/json");
-    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
+    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
     final var errorMappings = errorMappingCodes == null ? null : new HashMap<String, ParsableFactory<? extends Parsable>>();
     if (errorMappings != null)
         errorMappingCodes.forEach((mapping) -> errorMappings.put(mapping, mockParsableFactory));
     final var exception = assertThrows(ApiException.class, () -> requestAdapter.send(requestInformation, errorMappings, (node) -> mockEntity));
     assertNotNull(exception);
     if (expectDeserializedException)
         verify(mockParseNode, times(1)).getObjectValue(mockParsableFactory);
     assertEquals(responseStatusCode, exception.getResponseStatusCode());
     assertTrue(exception.getResponseHeaders().containsKey("request-id"));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@SuppressWarnings("unchecked")
@ParameterizedTest
@MethodSource("providesErrorMappings")
void throwsAPIException(int responseStatusCode, List<String> errorMappingCodes, boolean expectDeserializedException) throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
    final var client = getMockClient(new Response.Builder().code(responseStatusCode).message("Not Found").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(ResponseBody.create("test".getBytes("UTF-8"), MediaType.parse("application/json"))).header("request-id", "request-id-value").build());
    final var requestInformation = new RequestInformation() {

        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
    final var mockEntity = mock(Parsable.class);
    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
    final var mockParsableFactory = mock(ParsableFactory.class);
    when(mockParsableFactory.create(any(ParseNode.class))).thenReturn(mockEntity);
    final var mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
    final var mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn("application/json");
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
    final var errorMappings = errorMappingCodes == null ? null : new HashMap<String, ParsableFactory<? extends Parsable>>();
    if (errorMappings != null)
        errorMappingCodes.forEach((mapping) -> errorMappings.put(mapping, mockParsableFactory));
    final var exception = assertThrows(ApiException.class, () -> requestAdapter.send(requestInformation, errorMappings, (node) -> mockEntity));
    assertNotNull(exception);
    if (expectDeserializedException)
        verify(mockParseNode, times(1)).getObjectValue(mockParsableFactory);
    assertEquals(responseStatusCode, exception.getResponseStatusCode());
    assertTrue(exception.getResponseHeaders().containsKey("request-id"));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AuthenticationProvider authenticationProviderMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    authenticationProviderMock = mock(AuthenticationProvider.class);
}

// === Replace local variable in test with ===
authenticationProviderMock

```
</details>

---
#### Test Case ID #kiota-java_Test_4_7
#### Test Case Name: `getRequestFromRequestInformationHasCorrectContentLength_JsonPayload`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `authenticationProviderMock`
<summary>Suggested Diff</summary>

```diff
@@
@Test
void getRequestFromRequestInformationHasCorrectContentLength_JsonPayload() throws Exception {
-    final var authenticationProviderMock = mock(AuthenticationProvider.class);
+    // removed local mock; replaced with global field `authenticationProviderMock`
    final var requestInformation = new RequestInformation();
    requestInformation.setUri(new URI("https://localhost"));
    ByteArrayInputStream content = new ByteArrayInputStream("{\"name\":\"value\",\"array\":[\"1\",\"2\",\"3\"]}".getBytes(StandardCharsets.UTF_8));
    requestInformation.setStreamContent(content, "application/json");
    requestInformation.httpMethod = HttpMethod.PUT;
    final var contentLength = content.available();
    requestInformation.headers.tryAdd("Content-Length", String.valueOf(contentLength));
-    final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
+    final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
    final var request = adapter.getRequestFromRequestInformation(requestInformation, mock(Span.class), mock(Span.class));
    assertEquals(String.valueOf(contentLength), request.headers().get("Content-Length"));
    assertEquals("application/json", request.headers().get("Content-Type"));
    assertNotNull(request.body());
    assertEquals(request.body().contentLength(), contentLength);
    assertEquals(request.body().contentType(), MediaType.parse("application/json"));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void getRequestFromRequestInformationHasCorrectContentLength_JsonPayload() throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    final var requestInformation = new RequestInformation();
    requestInformation.setUri(new URI("https://localhost"));
    ByteArrayInputStream content = new ByteArrayInputStream("{\"name\":\"value\",\"array\":[\"1\",\"2\",\"3\"]}".getBytes(StandardCharsets.UTF_8));
    requestInformation.setStreamContent(content, "application/json");
    requestInformation.httpMethod = HttpMethod.PUT;
    final var contentLength = content.available();
    requestInformation.headers.tryAdd("Content-Length", String.valueOf(contentLength));
    final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
    final var request = adapter.getRequestFromRequestInformation(requestInformation, mock(Span.class), mock(Span.class));
    assertEquals(String.valueOf(contentLength), request.headers().get("Content-Length"));
    assertEquals("application/json", request.headers().get("Content-Type"));
    assertNotNull(request.body());
    assertEquals(request.body().contentLength(), contentLength);
    assertEquals(request.body().contentType(), MediaType.parse("application/json"));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AuthenticationProvider authenticationProviderMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    authenticationProviderMock = mock(AuthenticationProvider.class);
}

// === Replace local variable in test with ===
authenticationProviderMock

```
</details>

---
#### Test Case ID #kiota-java_Test_4_8
#### Test Case Name: `getRequestFromRequestInformationIncludesContentLength_FilePayload`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `authenticationProviderMock`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void getRequestFromRequestInformationIncludesContentLength_FilePayload() throws Exception {
-    final var authenticationProviderMock = mock(AuthenticationProvider.class);
+    // removed local mock; replaced with global field `authenticationProviderMock`
     final var testFile = new File("./src/test/resources/helloWorld.txt");
     final var requestInformation = new RequestInformation();
     requestInformation.setUri(new URI("https://localhost"));
     requestInformation.httpMethod = HttpMethod.PUT;
     final var contentLength = testFile.length();
     requestInformation.headers.add("Content-Length", String.valueOf(contentLength));
     try (FileInputStream content = new FileInputStream(testFile)) {
         requestInformation.setStreamContent(content, "application/octet-stream");
-        final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
+        final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
         final var request = adapter.getRequestFromRequestInformation(requestInformation, mock(Span.class), mock(Span.class));
         assertEquals(String.valueOf(contentLength), request.headers().get("Content-Length"));
         assertEquals("application/octet-stream", request.headers().get("Content-Type"));
         assertNotNull(request.body());
         assertEquals(request.body().contentLength(), contentLength);
         assertEquals(request.body().contentType(), MediaType.parse("application/octet-stream"));
     }
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void getRequestFromRequestInformationIncludesContentLength_FilePayload() throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    final var testFile = new File("./src/test/resources/helloWorld.txt");
    final var requestInformation = new RequestInformation();
    requestInformation.setUri(new URI("https://localhost"));
    requestInformation.httpMethod = HttpMethod.PUT;
    final var contentLength = testFile.length();
    requestInformation.headers.add("Content-Length", String.valueOf(contentLength));
    try (FileInputStream content = new FileInputStream(testFile)) {
        requestInformation.setStreamContent(content, "application/octet-stream");
        final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
        final var request = adapter.getRequestFromRequestInformation(requestInformation, mock(Span.class), mock(Span.class));
        assertEquals(String.valueOf(contentLength), request.headers().get("Content-Length"));
        assertEquals("application/octet-stream", request.headers().get("Content-Type"));
        assertNotNull(request.body());
        assertEquals(request.body().contentLength(), contentLength);
        assertEquals(request.body().contentType(), MediaType.parse("application/octet-stream"));
    }
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AuthenticationProvider authenticationProviderMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    authenticationProviderMock = mock(AuthenticationProvider.class);
}

// === Replace local variable in test with ===
authenticationProviderMock

```
</details>

---
#### Test Case ID #kiota-java_Test_4_9
#### Test Case Name: `getRequestFromRequestInformationWithoutContentLengthOverrideForStreamBody`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `authenticationProviderMock`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void getRequestFromRequestInformationWithoutContentLengthOverrideForStreamBody() throws Exception {
-    final var authenticationProviderMock = mock(AuthenticationProvider.class);
+    // removed local mock; replaced with global field `authenticationProviderMock`
     final var testFile = new File("./src/test/resources/helloWorld.txt");
     final var requestInformation = new RequestInformation();
     requestInformation.setUri(new URI("https://localhost"));
     requestInformation.httpMethod = HttpMethod.PUT;
     try (FileInputStream content = new FileInputStream(testFile)) {
         requestInformation.setStreamContent(content, "application/octet-stream");
         final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
         final var request = adapter.getRequestFromRequestInformation(requestInformation, mock(Span.class), mock(Span.class));
         assertEquals("application/octet-stream", request.headers().get("Content-Type"));
         assertNotNull(request.body());
         assertEquals(-1L, request.body().contentLength());
         assertEquals(request.body().contentType(), MediaType.parse("application/octet-stream"));
     }
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void getRequestFromRequestInformationWithoutContentLengthOverrideForStreamBody() throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    final var testFile = new File("./src/test/resources/helloWorld.txt");
    final var requestInformation = new RequestInformation();
    requestInformation.setUri(new URI("https://localhost"));
    requestInformation.httpMethod = HttpMethod.PUT;
    try (FileInputStream content = new FileInputStream(testFile)) {
        requestInformation.setStreamContent(content, "application/octet-stream");
        final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
        final var request = adapter.getRequestFromRequestInformation(requestInformation, mock(Span.class), mock(Span.class));
        assertEquals("application/octet-stream", request.headers().get("Content-Type"));
        assertNotNull(request.body());
        assertEquals(-1L, request.body().contentLength());
        assertEquals(request.body().contentType(), MediaType.parse("application/octet-stream"));
    }
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AuthenticationProvider authenticationProviderMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    authenticationProviderMock = mock(AuthenticationProvider.class);
}

// === Replace local variable in test with ===
authenticationProviderMock

```
</details>

---
#### Test Case ID #kiota-java_Test_4_10
#### Test Case Name: `getRequestFromRequestInformationWithoutContentLengthOverrideForJsonPayload`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `authenticationProviderMock`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void getRequestFromRequestInformationWithoutContentLengthOverrideForJsonPayload() throws Exception {
-    final var authenticationProviderMock = mock(AuthenticationProvider.class);
+    // removed local mock; replaced with global field `authenticationProviderMock`
     final var requestInformation = new RequestInformation();
     requestInformation.setUri(new URI("https://localhost"));
     ByteArrayInputStream content = new ByteArrayInputStream("{\"name\":\"value\",\"array\":[\"1\",\"2\",\"3\"]}".getBytes(StandardCharsets.UTF_8));
     requestInformation.setStreamContent(content, "application/json");
     requestInformation.httpMethod = HttpMethod.PUT;
     final var contentLength = content.available();
-    final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
+    final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
     final var request = adapter.getRequestFromRequestInformation(requestInformation, mock(Span.class), mock(Span.class));
     assertEquals("application/json", request.headers().get("Content-Type"));
     assertNotNull(request.body());
     assertEquals(contentLength, request.body().contentLength());
     assertEquals(MediaType.parse("application/json"), request.body().contentType());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void getRequestFromRequestInformationWithoutContentLengthOverrideForJsonPayload() throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    final var requestInformation = new RequestInformation();
    requestInformation.setUri(new URI("https://localhost"));
    ByteArrayInputStream content = new ByteArrayInputStream("{\"name\":\"value\",\"array\":[\"1\",\"2\",\"3\"]}".getBytes(StandardCharsets.UTF_8));
    requestInformation.setStreamContent(content, "application/json");
    requestInformation.httpMethod = HttpMethod.PUT;
    final var contentLength = content.available();
    final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
    final var request = adapter.getRequestFromRequestInformation(requestInformation, mock(Span.class), mock(Span.class));
    assertEquals("application/json", request.headers().get("Content-Type"));
    assertNotNull(request.body());
    assertEquals(contentLength, request.body().contentLength());
    assertEquals(MediaType.parse("application/json"), request.body().contentType());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AuthenticationProvider authenticationProviderMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    authenticationProviderMock = mock(AuthenticationProvider.class);
}

// === Replace local variable in test with ===
authenticationProviderMock

```
</details>

---
#### Test Case ID #kiota-java_Test_4_11
#### Test Case Name: `getRequestFromRequestInformationWithoutContentLengthOverrideWithEmptyPayload`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `authenticationProviderMock`
<summary>Suggested Diff</summary>

```diff
@@
@Test
void getRequestFromRequestInformationWithoutContentLengthOverrideWithEmptyPayload() throws Exception {
-    final var authenticationProviderMock = mock(AuthenticationProvider.class);
+    // removed local mock; replaced with global field `authenticationProviderMock`
    final var requestInformation = new RequestInformation();
    requestInformation.setUri(new URI("https://localhost"));
    ByteArrayInputStream content = new ByteArrayInputStream(new byte[0]);
    requestInformation.httpMethod = HttpMethod.PUT;
    requestInformation.content = content;
-    final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
+    final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
    final var request = adapter.getRequestFromRequestInformation(requestInformation, mock(Span.class), mock(Span.class));
    assertNull(request.headers().get("Content-Type"));
    assertEquals(0, request.body().contentLength());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void getRequestFromRequestInformationWithoutContentLengthOverrideWithEmptyPayload() throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    final var requestInformation = new RequestInformation();
    requestInformation.setUri(new URI("https://localhost"));
    ByteArrayInputStream content = new ByteArrayInputStream(new byte[0]);
    requestInformation.httpMethod = HttpMethod.PUT;
    requestInformation.content = content;
    final var adapter = new OkHttpRequestAdapter(authenticationProviderMock);
    final var request = adapter.getRequestFromRequestInformation(requestInformation, mock(Span.class), mock(Span.class));
    assertNull(request.headers().get("Content-Type"));
    assertEquals(0, request.body().contentLength());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private AuthenticationProvider authenticationProviderMock;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    authenticationProviderMock = mock(AuthenticationProvider.class);
}

// === Replace local variable in test with ===
authenticationProviderMock

```
</details>

---
## Mock Clone Instance #kiota-java_MCI_5
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.serialization.Parsable`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
private static Parsable createMockParsable() {
    Parsable mockParsable = mock(Parsable.class);
    when(mockParsable.getFieldDeserializers()).thenReturn(new HashMap<>());
    return mockParsable;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_5_1
#### Test Case Name: `sendReturnsNullOnNoContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `mockEntity`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
        }
    };
-    final var mockEntity = mock(Parsable.class);
-    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
+    final var mockEntity = createMockParsable();
    final var response = requestAdapter.send(requestInformation, null, (node) -> mockEntity);
    assertNull(response);
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@ParameterizedTest
@ValueSource(ints = { 200, 201, 202, 203, 204, 205 })
void sendReturnsNullOnNoContent(int statusCode) throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
    final var client = getMockClient(new Response.Builder().code(statusCode).message("OK").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(null).build());
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, null, null, client);
    final var requestInformation = new RequestInformation() {

        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
    final var mockEntity = mock(Parsable.class);
    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
    final var response = requestAdapter.send(requestInformation, null, (node) -> mockEntity);
    assertNull(response);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static Parsable createMockParsable() {
    Parsable mockParsable = mock(Parsable.class);
    when(mockParsable.getFieldDeserializers()).thenReturn(new HashMap<>());
    return mockParsable;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_5_2
#### Test Case Name: `sendReturnsObjectOnContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `mockEntity`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final var requestInformation = new RequestInformation() {
        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
-    final var mockEntity = mock(Parsable.class);
-    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
+    final var mockEntity = createMockParsable();
    final var mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
    final var mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn("application/json");
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
    final var response = requestAdapter.send(requestInformation, null, (node) -> mockEntity);
    assertNotNull(response);
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@ParameterizedTest
@ValueSource(ints = { 200, 201, 202, 203 })
void sendReturnsObjectOnContent(int statusCode) throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
    final var client = getMockClient(new Response.Builder().code(statusCode).message("OK").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(ResponseBody.create("test".getBytes("UTF-8"), MediaType.parse("application/json"))).build());
    final var requestInformation = new RequestInformation() {

        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
    final var mockEntity = mock(Parsable.class);
    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
    final var mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
    final var mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn("application/json");
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
    final var response = requestAdapter.send(requestInformation, null, (node) -> mockEntity);
    assertNotNull(response);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static Parsable createMockParsable() {
    Parsable mockParsable = mock(Parsable.class);
    when(mockParsable.getFieldDeserializers()).thenReturn(new HashMap<>());
    return mockParsable;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_5_3
#### Test Case Name: `throwsAPIException`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `mockEntity`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final var requestInformation = new RequestInformation() {
        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
-    final var mockEntity = mock(Parsable.class);
-    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
+    final var mockEntity = createMockParsable();
    final var mockParsableFactory = mock(ParsableFactory.class);
    when(mockParsableFactory.create(any(ParseNode.class))).thenReturn(mockEntity);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@SuppressWarnings("unchecked")
@ParameterizedTest
@MethodSource("providesErrorMappings")
void throwsAPIException(int responseStatusCode, List<String> errorMappingCodes, boolean expectDeserializedException) throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
    final var client = getMockClient(new Response.Builder().code(responseStatusCode).message("Not Found").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(ResponseBody.create("test".getBytes("UTF-8"), MediaType.parse("application/json"))).header("request-id", "request-id-value").build());
    final var requestInformation = new RequestInformation() {

        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
    final var mockEntity = mock(Parsable.class);
    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
    final var mockParsableFactory = mock(ParsableFactory.class);
    when(mockParsableFactory.create(any(ParseNode.class))).thenReturn(mockEntity);
    final var mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
    final var mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn("application/json");
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
    final var errorMappings = errorMappingCodes == null ? null : new HashMap<String, ParsableFactory<? extends Parsable>>();
    if (errorMappings != null)
        errorMappingCodes.forEach((mapping) -> errorMappings.put(mapping, mockParsableFactory));
    final var exception = assertThrows(ApiException.class, () -> requestAdapter.send(requestInformation, errorMappings, (node) -> mockEntity));
    assertNotNull(exception);
    if (expectDeserializedException)
        verify(mockParseNode, times(1)).getObjectValue(mockParsableFactory);
    assertEquals(responseStatusCode, exception.getResponseStatusCode());
    assertTrue(exception.getResponseHeaders().containsKey("request-id"));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static Parsable createMockParsable() {
    Parsable mockParsable = mock(Parsable.class);
    when(mockParsable.getFieldDeserializers()).thenReturn(new HashMap<>());
    return mockParsable;
}
```
</details>

---
## Mock Clone Instance #kiota-java_MCI_6
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.serialization.SerializationWriterFactory`
- **Test Case Count**: 5
- **MO Count**: 5

### Reusable Method
```java
private static SerializationWriterFactory createMockSerializationWriterFactory(SerializationWriter writerMock) {
    SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    return factoryMock;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_6_1
#### Test Case Name: `SetsParsableContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `factoryMock`
<summary>Suggested Diff</summary>

```diff
@@
     final SerializationWriter writerMock = mock(SerializationWriter.class);
-    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
-    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
+    final SerializationWriterFactory factoryMock = createMockSerializationWriterFactory(writerMock);
     final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
     when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
     requestInfo.setContentFromParsable(requestAdapterMock, "application/json", new TestEntity());
     verify(writerMock, times(1)).writeObjectValue(any(), any(TestEntity.class));
     verify(writerMock, never()).writeCollectionOfObjectValues(anyString(), any(ArrayList.class));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsParsableContent() {
    // Arrange as the request builders would
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/users";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    requestInfo.setContentFromParsable(requestAdapterMock, "application/json", new TestEntity());
    verify(writerMock, times(1)).writeObjectValue(any(), any(TestEntity.class));
    verify(writerMock, never()).writeCollectionOfObjectValues(anyString(), any(ArrayList.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static SerializationWriterFactory createMockSerializationWriterFactory(SerializationWriter writerMock) {
    SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    return factoryMock;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_6_2
#### Test Case Name: `SetsParsableContentCollection`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `factoryMock`
<summary>Suggested Diff</summary>

```diff
@@
     final SerializationWriter writerMock = mock(SerializationWriter.class);
-    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
-    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
+    final SerializationWriterFactory factoryMock = createMockSerializationWriterFactory(writerMock);
     final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
     when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
     requestInfo.setContentFromParsable(requestAdapterMock, "application/json", new TestEntity[] { new TestEntity() });
     verify(writerMock, never()).writeObjectValue(any(), any(TestEntity.class));
     verify(writerMock, times(1)).writeCollectionOfObjectValues(any(), any(Iterable.class));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsParsableContentCollection() {
    // Arrange as the request builders would
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/users";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    requestInfo.setContentFromParsable(requestAdapterMock, "application/json", new TestEntity[] { new TestEntity() });
    verify(writerMock, never()).writeObjectValue(any(), any(TestEntity.class));
    verify(writerMock, times(1)).writeCollectionOfObjectValues(any(), any(Iterable.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static SerializationWriterFactory createMockSerializationWriterFactory(SerializationWriter writerMock) {
    SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    return factoryMock;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_6_3
#### Test Case Name: `SetsScalarContentCollection`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `factoryMock`
<summary>Suggested Diff</summary>

```diff
@@
     final SerializationWriter writerMock = mock(SerializationWriter.class);
-    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
-    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
+    final SerializationWriterFactory factoryMock = createMockSerializationWriterFactory(writerMock);
     final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
     when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
     requestInfo.setContentFromScalarCollection(requestAdapterMock, "application/json", new String[] { "foo" });
     verify(writerMock, never()).writeStringValue(any(), anyString());
     verify(writerMock, times(1)).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsScalarContentCollection() {
    // Arrange as the request builders would
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/users";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    requestInfo.setContentFromScalarCollection(requestAdapterMock, "application/json", new String[] { "foo" });
    verify(writerMock, never()).writeStringValue(any(), anyString());
    verify(writerMock, times(1)).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static SerializationWriterFactory createMockSerializationWriterFactory(SerializationWriter writerMock) {
    SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    return factoryMock;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_6_4
#### Test Case Name: `SetsScalarContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `factoryMock`
<summary>Suggested Diff</summary>

```diff
@@
     final SerializationWriter writerMock = mock(SerializationWriter.class);
-    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
-    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
+    final SerializationWriterFactory factoryMock = createMockSerializationWriterFactory(writerMock);
     final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
     when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
     requestInfo.setContentFromScalar(requestAdapterMock, "application/json", "foo");
     verify(writerMock, times(1)).writeStringValue(any(), anyString());
     verify(writerMock, never()).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsScalarContent() {
    // Arrange as the request builders would
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/users";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    requestInfo.setContentFromScalar(requestAdapterMock, "application/json", "foo");
    verify(writerMock, times(1)).writeStringValue(any(), anyString());
    verify(writerMock, never()).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static SerializationWriterFactory createMockSerializationWriterFactory(SerializationWriter writerMock) {
    SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    return factoryMock;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_6_5
#### Test Case Name: `SetsBoundaryOnMultipartBody`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `factoryMock`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final SerializationWriter writerMock = mock(SerializationWriter.class);
-    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
-    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
+    final SerializationWriterFactory factoryMock = createMockSerializationWriterFactory(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    final MultipartBody multipartBody = new MultipartBody();
    multipartBody.requestAdapter = requestAdapterMock;
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsBoundaryOnMultipartBody() {
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/{URITemplate}/ParameterMapping?IsCaseSensitive={IsCaseSensitive}";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    final MultipartBody multipartBody = new MultipartBody();
    multipartBody.requestAdapter = requestAdapterMock;
    requestInfo.setContentFromParsable(requestAdapterMock, "multipart/form-data", multipartBody);
    assertNotNull(multipartBody.getBoundary());
    assertFalse(multipartBody.getBoundary().isEmpty());
    assertEquals("multipart/form-data; boundary=" + multipartBody.getBoundary(), requestInfo.headers.get("Content-Type").toArray()[0]);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static SerializationWriterFactory createMockSerializationWriterFactory(SerializationWriter writerMock) {
    SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    return factoryMock;
}
```
</details>

---
## Mock Clone Instance #kiota-java_MCI_7
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.serialization.SerializationWriterFactory`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
private static SerializationWriterFactory createMockSerializationWriterFactory(String contentType, SerializationWriter mockSerializationWriter) {
    SerializationWriterFactory mockSerializationWriterFactory = mock(SerializationWriterFactory.class);
    when(mockSerializationWriterFactory.getSerializationWriter(contentType)).thenReturn(mockSerializationWriter);
    return mockSerializationWriterFactory;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_7_1
#### Test Case Name: `serializesObject`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\serialization\SerializationHelpersTest.java`)
#### Mock Object Variable Name: `mockSerializationWriterFactory`
<summary>Suggested Diff</summary>

```diff
@@
    final var mockSerializationWriter = mock(SerializationWriter.class);
    when(mockSerializationWriter.getSerializedContent()).thenReturn(new ByteArrayInputStream("{'id':'123'}".getBytes(_charset)));
-    final var mockSerializationWriterFactory = mock(SerializationWriterFactory.class);
-    when(mockSerializationWriterFactory.getSerializationWriter(_jsonContentType)).thenReturn(mockSerializationWriter);
+    final var mockSerializationWriterFactory = createMockSerializationWriterFactory(_jsonContentType, mockSerializationWriter);
    SerializationWriterFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockSerializationWriterFactory);
    final var result = KiotaSerialization.serializeAsString(_jsonContentType, new TestEntity() {

        {
            setId("123");
        }
    });
    assertEquals("{'id':'123'}", result);
    verify(mockSerializationWriter, times(1)).writeObjectValue(eq(""), any(Parsable.class));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void serializesObject() throws IOException {
    final var mockSerializationWriter = mock(SerializationWriter.class);
    when(mockSerializationWriter.getSerializedContent()).thenReturn(new ByteArrayInputStream("{'id':'123'}".getBytes(_charset)));
    final var mockSerializationWriterFactory = mock(SerializationWriterFactory.class);
    when(mockSerializationWriterFactory.getSerializationWriter(_jsonContentType)).thenReturn(mockSerializationWriter);
    SerializationWriterFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockSerializationWriterFactory);
    final var result = KiotaSerialization.serializeAsString(_jsonContentType, new TestEntity() {

        {
            setId("123");
        }
    });
    assertEquals("{'id':'123'}", result);
    verify(mockSerializationWriter, times(1)).writeObjectValue(eq(""), any(Parsable.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static SerializationWriterFactory createMockSerializationWriterFactory(String contentType, SerializationWriter mockSerializationWriter) {
    SerializationWriterFactory mockSerializationWriterFactory = mock(SerializationWriterFactory.class);
    when(mockSerializationWriterFactory.getSerializationWriter(contentType)).thenReturn(mockSerializationWriter);
    return mockSerializationWriterFactory;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_7_2
#### Test Case Name: `serializesObjectCollection`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\serialization\SerializationHelpersTest.java`)
#### Mock Object Variable Name: `mockSerializationWriterFactory`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final var mockSerializationWriter = mock(SerializationWriter.class);
    when(mockSerializationWriter.getSerializedContent()).thenReturn(new ByteArrayInputStream("[{\'id\':\'123\'}]".getBytes(_charset)));
-    final var mockSerializationWriterFactory = mock(SerializationWriterFactory.class);
-    when(mockSerializationWriterFactory.getSerializationWriter(_jsonContentType)).thenReturn(mockSerializationWriter);
+    final var mockSerializationWriterFactory = createMockSerializationWriterFactory(_jsonContentType, mockSerializationWriter);
    SerializationWriterFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockSerializationWriterFactory);
    final var result = KiotaSerialization.serializeAsString(_jsonContentType, new ArrayList<>() {
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void serializesObjectCollection() throws IOException {
    final var mockSerializationWriter = mock(SerializationWriter.class);
    when(mockSerializationWriter.getSerializedContent()).thenReturn(new ByteArrayInputStream("[{'id':'123'}]".getBytes(_charset)));
    final var mockSerializationWriterFactory = mock(SerializationWriterFactory.class);
    when(mockSerializationWriterFactory.getSerializationWriter(_jsonContentType)).thenReturn(mockSerializationWriter);
    SerializationWriterFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockSerializationWriterFactory);
    final var result = KiotaSerialization.serializeAsString(_jsonContentType, new ArrayList<>() {

        {
            add(new TestEntity() {

                {
                    setId("123");
                }
            });
        }
    });
    assertEquals("[{'id':'123'}]", result);
    verify(mockSerializationWriter, times(1)).writeCollectionOfObjectValues(eq(""), any());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static SerializationWriterFactory createMockSerializationWriterFactory(String contentType, SerializationWriter mockSerializationWriter) {
    SerializationWriterFactory mockSerializationWriterFactory = mock(SerializationWriterFactory.class);
    when(mockSerializationWriterFactory.getSerializationWriter(contentType)).thenReturn(mockSerializationWriter);
    return mockSerializationWriterFactory;
}
```
</details>

---
## Mock Clone Instance #kiota-java_MCI_8
- **Scope**: method level
- **Mocked Class**: `com.azure.core.credential.TokenCredential`
- **Test Case Count**: 4
- **MO Count**: 4

### Reusable Method
```java
// === Declare in class scope ===
private TokenCredential tokenCredential;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    tokenCredential = mock(TokenCredential.class);
}

// === Replace local variable in test with ===
tokenCredential;

```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_8_1
#### Test Case Name: `testNonLocalhostHttpUrlIsInvalid`(File: `C:\Java_projects\Microsoft\kiota-java\components\authentication\azure\src\test\java\com\microsoft\kiota\authentication\AzureIdentityAccessTokenProviderTest.java`)
#### Mock Object Variable Name: `tokenCredential`
<summary>Suggested Diff</summary>

```diff
@@
 @ParameterizedTest
 @ValueSource(strings = { "http://graph.microsoft.com/me" })
 void testNonLocalhostHttpUrlIsInvalid(String urlString) {
-    var tokenCredential = mock(TokenCredential.class);
+    // removed local mock; replaced with global field `tokenCredential`
     var accessTokenProvider = new AzureIdentityAccessTokenProvider(tokenCredential, null, "");
     assertThrows(IllegalArgumentException.class, () -> accessTokenProvider.getAuthorizationToken(new URI(urlString), new HashMap<>()));
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@ParameterizedTest
@ValueSource(strings = { "http://graph.microsoft.com/me" })
void testNonLocalhostHttpUrlIsInvalid(String urlString) {
    var tokenCredential = mock(TokenCredential.class);
    var accessTokenProvider = new AzureIdentityAccessTokenProvider(tokenCredential, null, "");
    assertThrows(IllegalArgumentException.class, () -> accessTokenProvider.getAuthorizationToken(new URI(urlString), new HashMap<>()));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private TokenCredential tokenCredential;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    tokenCredential = mock(TokenCredential.class);
}

// === Replace local variable in test with ===
tokenCredential;

```
</details>

---
#### Test Case ID #kiota-java_Test_8_2
#### Test Case Name: `testKeepUserProvidedScopes`(File: `C:\Java_projects\Microsoft\kiota-java\components\authentication\azure\src\test\java\com\microsoft\kiota\authentication\AzureIdentityAccessTokenProviderTest.java`)
#### Mock Object Variable Name: `tokenCredential`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void testKeepUserProvidedScopes() throws URISyntaxException {
-    var tokenCredential = mock(TokenCredential.class);
+    // removed local mock; replaced with global field `tokenCredential`
     String[] userProvidedScopes = { "https://graph.microsoft.com/User.Read", "https://graph.microsoft.com/Application.Read" };
     var accessTokenProvider = new AzureIdentityAccessTokenProvider(tokenCredential, new String[] {}, userProvidedScopes);
     assertScopes(tokenCredential, accessTokenProvider, userProvidedScopes);
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void testKeepUserProvidedScopes() throws URISyntaxException {
    var tokenCredential = mock(TokenCredential.class);
    String[] userProvidedScopes = { "https://graph.microsoft.com/User.Read", "https://graph.microsoft.com/Application.Read" };
    var accessTokenProvider = new AzureIdentityAccessTokenProvider(tokenCredential, new String[] {}, userProvidedScopes);
    assertScopes(tokenCredential, accessTokenProvider, userProvidedScopes);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private TokenCredential tokenCredential;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    tokenCredential = mock(TokenCredential.class);
}

// === Replace local variable in test with ===
tokenCredential;

```
</details>

---
#### Test Case ID #kiota-java_Test_8_3
#### Test Case Name: `testConfigureDefaultScopeWhenScopesNotProvided`(File: `C:\Java_projects\Microsoft\kiota-java\components\authentication\azure\src\test\java\com\microsoft\kiota\authentication\AzureIdentityAccessTokenProviderTest.java`)
#### Mock Object Variable Name: `tokenCredential`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void testConfigureDefaultScopeWhenScopesNotProvided() throws URISyntaxException {
-    var tokenCredential = mock(TokenCredential.class);
+    // removed local mock; replaced with global field `tokenCredential`
     var accessTokenProvider = new AzureIdentityAccessTokenProvider(tokenCredential, new String[] {});
     assertScopes(tokenCredential, accessTokenProvider, new String[] { "https://graph.microsoft.com/.default" });
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void testConfigureDefaultScopeWhenScopesNotProvided() throws URISyntaxException {
    var tokenCredential = mock(TokenCredential.class);
    var accessTokenProvider = new AzureIdentityAccessTokenProvider(tokenCredential, new String[] {});
    assertScopes(tokenCredential, accessTokenProvider, new String[] { "https://graph.microsoft.com/.default" });
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private TokenCredential tokenCredential;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    tokenCredential = mock(TokenCredential.class);
}

// === Replace local variable in test with ===
tokenCredential;

```
</details>

---
#### Test Case ID #kiota-java_Test_8_4
#### Test Case Name: `testConfigureDefaultScopeWhenScopesNullOrEmpty`(File: `C:\Java_projects\Microsoft\kiota-java\components\authentication\azure\src\test\java\com\microsoft\kiota\authentication\AzureIdentityAccessTokenProviderTest.java`)
#### Mock Object Variable Name: `tokenCredential`
<summary>Suggested Diff</summary>

```diff
@@
@ParameterizedTest
@NullAndEmptySource
void testConfigureDefaultScopeWhenScopesNullOrEmpty(String[] nullOrEmptyUserProvidedScopes) throws URISyntaxException {
-    var tokenCredential = mock(TokenCredential.class);
+    // removed local mock; replaced with global field `tokenCredential`
    var accessTokenProvider = new AzureIdentityAccessTokenProvider(tokenCredential, new String[] {}, nullOrEmptyUserProvidedScopes);
    assertScopes(tokenCredential, accessTokenProvider, new String[] { "https://graph.microsoft.com/.default" });
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@ParameterizedTest
@NullAndEmptySource
void testConfigureDefaultScopeWhenScopesNullOrEmpty(String[] nullOrEmptyUserProvidedScopes) throws URISyntaxException {
    var tokenCredential = mock(TokenCredential.class);
    var accessTokenProvider = new AzureIdentityAccessTokenProvider(tokenCredential, new String[] {}, nullOrEmptyUserProvidedScopes);
    assertScopes(tokenCredential, accessTokenProvider, new String[] { "https://graph.microsoft.com/.default" });
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private TokenCredential tokenCredential;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    tokenCredential = mock(TokenCredential.class);
}

// === Replace local variable in test with ===
tokenCredential;

```
</details>

---
## Mock Clone Instance #kiota-java_MCI_9
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.RequestAdapter`
- **Test Case Count**: 5
- **MO Count**: 5

### Reusable Method
```java
private static RequestAdapter createMockRequestAdapter(SerializationWriterFactory factoryMock) {
    RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    return requestAdapterMock;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_9_1
#### Test Case Name: `SetsParsableContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `requestAdapterMock`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
-    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
-    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
+    final RequestAdapter requestAdapterMock = createMockRequestAdapter(factoryMock);
    requestInfo.setContentFromParsable(requestAdapterMock, "application/json", new TestEntity());
    verify(writerMock, times(1)).writeObjectValue(any(), any(TestEntity.class));
    verify(writerMock, never()).writeCollectionOfObjectValues(anyString(), any(ArrayList.class));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsParsableContent() {
    // Arrange as the request builders would
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/users";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    requestInfo.setContentFromParsable(requestAdapterMock, "application/json", new TestEntity());
    verify(writerMock, times(1)).writeObjectValue(any(), any(TestEntity.class));
    verify(writerMock, never()).writeCollectionOfObjectValues(anyString(), any(ArrayList.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static RequestAdapter createMockRequestAdapter(SerializationWriterFactory factoryMock) {
    RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    return requestAdapterMock;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_9_2
#### Test Case Name: `SetsParsableContentCollection`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `requestAdapterMock`
<summary>Suggested Diff</summary>

```diff
@@
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
-    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
-    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
+    final RequestAdapter requestAdapterMock = createMockRequestAdapter(factoryMock);
    requestInfo.setContentFromParsable(requestAdapterMock, "application/json", new TestEntity[] { new TestEntity() });
    verify(writerMock, never()).writeObjectValue(any(), any(TestEntity.class));
    verify(writerMock, times(1)).writeCollectionOfObjectValues(any(), any(Iterable.class));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsParsableContentCollection() {
    // Arrange as the request builders would
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/users";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    requestInfo.setContentFromParsable(requestAdapterMock, "application/json", new TestEntity[] { new TestEntity() });
    verify(writerMock, never()).writeObjectValue(any(), any(TestEntity.class));
    verify(writerMock, times(1)).writeCollectionOfObjectValues(any(), any(Iterable.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static RequestAdapter createMockRequestAdapter(SerializationWriterFactory factoryMock) {
    RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    return requestAdapterMock;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_9_3
#### Test Case Name: `SetsScalarContentCollection`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `requestAdapterMock`
<summary>Suggested Diff</summary>

```diff
@@
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
-    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
-    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
+    final RequestAdapter requestAdapterMock = createMockRequestAdapter(factoryMock);
    requestInfo.setContentFromScalarCollection(requestAdapterMock, "application/json", new String[] { "foo" });
    verify(writerMock, never()).writeStringValue(any(), anyString());
    verify(writerMock, times(1)).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsScalarContentCollection() {
    // Arrange as the request builders would
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/users";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    requestInfo.setContentFromScalarCollection(requestAdapterMock, "application/json", new String[] { "foo" });
    verify(writerMock, never()).writeStringValue(any(), anyString());
    verify(writerMock, times(1)).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static RequestAdapter createMockRequestAdapter(SerializationWriterFactory factoryMock) {
    RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    return requestAdapterMock;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_9_4
#### Test Case Name: `SetsScalarContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `requestAdapterMock`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
-    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
-    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
+    final RequestAdapter requestAdapterMock = createMockRequestAdapter(factoryMock);
    requestInfo.setContentFromScalar(requestAdapterMock, "application/json", "foo");
    verify(writerMock, times(1)).writeStringValue(any(), anyString());
    verify(writerMock, never()).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsScalarContent() {
    // Arrange as the request builders would
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/users";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    requestInfo.setContentFromScalar(requestAdapterMock, "application/json", "foo");
    verify(writerMock, times(1)).writeStringValue(any(), anyString());
    verify(writerMock, never()).writeCollectionOfPrimitiveValues(any(), any(Iterable.class));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static RequestAdapter createMockRequestAdapter(SerializationWriterFactory factoryMock) {
    RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    return requestAdapterMock;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_9_5
#### Test Case Name: `SetsBoundaryOnMultipartBody`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\RequestInformationTest.java`)
#### Mock Object Variable Name: `requestAdapterMock`
<summary>Suggested Diff</summary>

```diff
@@
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
-    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
-    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
+    final RequestAdapter requestAdapterMock = createMockRequestAdapter(factoryMock);
    final MultipartBody multipartBody = new MultipartBody();
    multipartBody.requestAdapter = requestAdapterMock;
    requestInfo.setContentFromParsable(requestAdapterMock, "multipart/form-data", multipartBody);
    assertNotNull(multipartBody.getBoundary());
    assertFalse(multipartBody.getBoundary().isEmpty());
    assertEquals("multipart/form-data; boundary=" + multipartBody.getBoundary(), requestInfo.headers.get("Content-Type").toArray()[0]);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void SetsBoundaryOnMultipartBody() {
    final RequestInformation requestInfo = new RequestInformation();
    requestInfo.httpMethod = HttpMethod.POST;
    requestInfo.urlTemplate = "http://localhost/{URITemplate}/ParameterMapping?IsCaseSensitive={IsCaseSensitive}";
    final SerializationWriter writerMock = mock(SerializationWriter.class);
    final SerializationWriterFactory factoryMock = mock(SerializationWriterFactory.class);
    when(factoryMock.getSerializationWriter(anyString())).thenReturn(writerMock);
    final RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    final MultipartBody multipartBody = new MultipartBody();
    multipartBody.requestAdapter = requestAdapterMock;
    requestInfo.setContentFromParsable(requestAdapterMock, "multipart/form-data", multipartBody);
    assertNotNull(multipartBody.getBoundary());
    assertFalse(multipartBody.getBoundary().isEmpty());
    assertEquals("multipart/form-data; boundary=" + multipartBody.getBoundary(), requestInfo.headers.get("Content-Type").toArray()[0]);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static RequestAdapter createMockRequestAdapter(SerializationWriterFactory factoryMock) {
    RequestAdapter requestAdapterMock = mock(RequestAdapter.class);
    when(requestAdapterMock.getSerializationWriterFactory()).thenReturn(factoryMock);
    return requestAdapterMock;
}
```
</details>

---
## Mock Clone Instance #kiota-java_MCI_10
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.RequestAdapter`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
// === Declare in class scope ===
private RequestAdapter requestAdapter;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    requestAdapter = mock(RequestAdapter.class);
}

// === Replace local variable in test with ===
requestAdapter;

```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_10_1
#### Test Case Name: `requiresPartsForSerialization`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\MultiPartBodyTest.java`)
#### Mock Object Variable Name: `requestAdapter`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void requiresPartsForSerialization() {
     final MultipartBody multipartBody = new MultipartBody();
     final SerializationWriter writer = mock(SerializationWriter.class);
-    final RequestAdapter requestAdapter = mock(RequestAdapter.class);
+    // removed local mock; replaced with global field `requestAdapter`
     multipartBody.requestAdapter = requestAdapter;
     assertThrows(IllegalStateException.class, () -> multipartBody.serialize(writer));
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void requiresPartsForSerialization() {
    final MultipartBody multipartBody = new MultipartBody();
    final SerializationWriter writer = mock(SerializationWriter.class);
    final RequestAdapter requestAdapter = mock(RequestAdapter.class);
    multipartBody.requestAdapter = requestAdapter;
    assertThrows(IllegalStateException.class, () -> multipartBody.serialize(writer));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private RequestAdapter requestAdapter;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    requestAdapter = mock(RequestAdapter.class);
}

// === Replace local variable in test with ===
requestAdapter;

```
</details>

---
#### Test Case ID #kiota-java_Test_10_2
#### Test Case Name: `addsPart`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\MultiPartBodyTest.java`)
#### Mock Object Variable Name: `requestAdapter`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void addsPart() {
     final MultipartBody multipartBody = new MultipartBody();
-    final RequestAdapter requestAdapter = mock(RequestAdapter.class);
+    // removed local mock; replaced with global field `requestAdapter`
     multipartBody.requestAdapter = requestAdapter;
     multipartBody.addOrReplacePart("foo", "bar", "baz");
     final Object result = multipartBody.getPartValue("foo");
     assertNotNull(result);
     assertTrue(result instanceof String);
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void addsPart() {
    final MultipartBody multipartBody = new MultipartBody();
    final RequestAdapter requestAdapter = mock(RequestAdapter.class);
    multipartBody.requestAdapter = requestAdapter;
    multipartBody.addOrReplacePart("foo", "bar", "baz");
    final Object result = multipartBody.getPartValue("foo");
    assertNotNull(result);
    assertTrue(result instanceof String);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private RequestAdapter requestAdapter;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    requestAdapter = mock(RequestAdapter.class);
}

// === Replace local variable in test with ===
requestAdapter;

```
</details>

---
#### Test Case ID #kiota-java_Test_10_3
#### Test Case Name: `removesPart`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\MultiPartBodyTest.java`)
#### Mock Object Variable Name: `requestAdapter`
<summary>Suggested Diff</summary>

```diff
@@
 @Test
 void removesPart() {
     final MultipartBody multipartBody = new MultipartBody();
-    final RequestAdapter requestAdapter = mock(RequestAdapter.class);
+    // removed local mock; replaced with global field `requestAdapter`
     multipartBody.requestAdapter = requestAdapter;
     multipartBody.addOrReplacePart("foo", "bar", "baz");
     multipartBody.removePart("FOO");
     final Object result = multipartBody.getPartValue("foo");
     assertNull(result);
 }
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void removesPart() {
    final MultipartBody multipartBody = new MultipartBody();
    final RequestAdapter requestAdapter = mock(RequestAdapter.class);
    multipartBody.requestAdapter = requestAdapter;
    multipartBody.addOrReplacePart("foo", "bar", "baz");
    multipartBody.removePart("FOO");
    final Object result = multipartBody.getPartValue("foo");
    assertNull(result);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
// === Declare in class scope ===
private RequestAdapter requestAdapter;

// === Add to @BeforeEach method ===
@BeforeEach
public void setUp() {
    requestAdapter = mock(RequestAdapter.class);
}

// === Replace local variable in test with ===
requestAdapter;

```
</details>

---
## Mock Clone Instance #kiota-java_MCI_11
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.serialization.ParseNodeFactory`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
private static ParseNodeFactory createMockParseNodeFactory(ParseNode mockParseNode, String validContentType) {
    ParseNodeFactory mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn(validContentType);
    return mockFactory;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_11_1
#### Test Case Name: `sendReturnsObjectOnContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `mockFactory`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final var mockEntity = mock(Parsable.class);
    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
    final var mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
-    final var mockFactory = mock(ParseNodeFactory.class);
-    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
-    when(mockFactory.getValidContentType()).thenReturn("application/json");
+    final var mockFactory = createMockParseNodeFactory(mockParseNode, "application/json");
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
    final var response = requestAdapter.send(requestInformation, null, (node) -> mockEntity);
    assertNotNull(response);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@ParameterizedTest
@ValueSource(ints = { 200, 201, 202, 203 })
void sendReturnsObjectOnContent(int statusCode) throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
    final var client = getMockClient(new Response.Builder().code(statusCode).message("OK").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(ResponseBody.create("test".getBytes("UTF-8"), MediaType.parse("application/json"))).build());
    final var requestInformation = new RequestInformation() {

        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
    final var mockEntity = mock(Parsable.class);
    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
    final var mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
    final var mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn("application/json");
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
    final var response = requestAdapter.send(requestInformation, null, (node) -> mockEntity);
    assertNotNull(response);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ParseNodeFactory createMockParseNodeFactory(ParseNode mockParseNode, String validContentType) {
    ParseNodeFactory mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn(validContentType);
    return mockFactory;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_11_2
#### Test Case Name: `throwsAPIException`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `mockFactory`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final var mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
-    final var mockFactory = mock(ParseNodeFactory.class);
-    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
-    when(mockFactory.getValidContentType()).thenReturn("application/json");
+    final var mockFactory = createMockParseNodeFactory(mockParseNode, "application/json");
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
    final var errorMappings = errorMappingCodes == null ? null : new HashMap<String, ParsableFactory<? extends Parsable>>();
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@SuppressWarnings("unchecked")
@ParameterizedTest
@MethodSource("providesErrorMappings")
void throwsAPIException(int responseStatusCode, List<String> errorMappingCodes, boolean expectDeserializedException) throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
    final var client = getMockClient(new Response.Builder().code(responseStatusCode).message("Not Found").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(ResponseBody.create("test".getBytes("UTF-8"), MediaType.parse("application/json"))).header("request-id", "request-id-value").build());
    final var requestInformation = new RequestInformation() {

        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
    final var mockEntity = mock(Parsable.class);
    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
    final var mockParsableFactory = mock(ParsableFactory.class);
    when(mockParsableFactory.create(any(ParseNode.class))).thenReturn(mockEntity);
    final var mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
    final var mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn("application/json");
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
    final var errorMappings = errorMappingCodes == null ? null : new HashMap<String, ParsableFactory<? extends Parsable>>();
    if (errorMappings != null)
        errorMappingCodes.forEach((mapping) -> errorMappings.put(mapping, mockParsableFactory));
    final var exception = assertThrows(ApiException.class, () -> requestAdapter.send(requestInformation, errorMappings, (node) -> mockEntity));
    assertNotNull(exception);
    if (expectDeserializedException)
        verify(mockParseNode, times(1)).getObjectValue(mockParsableFactory);
    assertEquals(responseStatusCode, exception.getResponseStatusCode());
    assertTrue(exception.getResponseHeaders().containsKey("request-id"));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ParseNodeFactory createMockParseNodeFactory(ParseNode mockParseNode, String validContentType) {
    ParseNodeFactory mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn(validContentType);
    return mockFactory;
}
```
</details>

---
## Mock Clone Instance #kiota-java_MCI_12
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.serialization.ParseNodeFactory`
- **Test Case Count**: 3
- **MO Count**: 3

### Reusable Method
```java
private static ParseNodeFactory createMockParseNodeFactory(ParseNode mockParseNode) {
    ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
    return mockParseNodeFactory;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_12_1
#### Test Case Name: `deserializesObjectWithoutReflection`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\serialization\DeserializationHelpersTest.java`)
#### Mock Object Variable Name: `mockParseNodeFactory`
<summary>Suggested Diff</summary>

```diff
@@
    final ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any())).thenReturn(new TestEntity() {

        {
            setId("123");
        }
    });
-    final ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
-    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
+    final ParseNodeFactory mockParseNodeFactory = createMockParseNodeFactory(mockParseNode);
    ParseNodeFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockParseNodeFactory);
    final var result = KiotaSerialization.deserialize(_jsonContentType, strValue, TestEntity::createFromDiscriminatorValue);
    assertEquals("123", result.getId());
}
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void deserializesObjectWithoutReflection() throws IOException {
    final String strValue = "{'id':'123'}";
    final ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any())).thenReturn(new TestEntity() {

        {
            setId("123");
        }
    });
    final ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
    ParseNodeFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockParseNodeFactory);
    final var result = KiotaSerialization.deserialize(_jsonContentType, strValue, TestEntity::createFromDiscriminatorValue);
    assertEquals("123", result.getId());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ParseNodeFactory createMockParseNodeFactory(ParseNode mockParseNode) {
    ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
    return mockParseNodeFactory;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_12_2
#### Test Case Name: `deserializesObjectWithReflection`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\serialization\DeserializationHelpersTest.java`)
#### Mock Object Variable Name: `mockParseNodeFactory`
<summary>Suggested Diff</summary>

```diff
@@
    final ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any())).thenReturn(new TestEntity() {

        {
            setId("123");
        }
    });
-    final ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
-    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
+    final ParseNodeFactory mockParseNodeFactory = createMockParseNodeFactory(mockParseNode);
    ParseNodeFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockParseNodeFactory);
    final var result = KiotaSerialization.deserialize(_jsonContentType, strValue, TestEntity::createFromDiscriminatorValue);
    assertEquals("123", result.getId());
}
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void deserializesObjectWithReflection() throws IOException {
    final String strValue = "{'id':'123'}";
    final ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any())).thenReturn(new TestEntity() {

        {
            setId("123");
        }
    });
    final ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
    ParseNodeFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockParseNodeFactory);
    final var result = KiotaSerialization.deserialize(_jsonContentType, strValue, TestEntity::createFromDiscriminatorValue);
    assertEquals("123", result.getId());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ParseNodeFactory createMockParseNodeFactory(ParseNode mockParseNode) {
    ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
    return mockParseNodeFactory;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_12_3
#### Test Case Name: `deserializesCollectionOfObjects`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\serialization\DeserializationHelpersTest.java`)
#### Mock Object Variable Name: `mockParseNodeFactory`
<summary>Suggested Diff</summary>

```diff
@@
    final ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getCollectionOfObjectValues(any())).thenReturn(new ArrayList<Parsable>() {
        {
            add(new TestEntity() {
                {
                    setId("123");
                }
            });
        }
    });
-    final ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
-    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
+    final ParseNodeFactory mockParseNodeFactory = createMockParseNodeFactory(mockParseNode);
    ParseNodeFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockParseNodeFactory);
    final var result = KiotaSerialization.deserializeCollection(_jsonContentType, strValue, TestEntity::createFromDiscriminatorValue);
    assertEquals("123", result.get(0).getId());
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void deserializesCollectionOfObjects() throws IOException {
    final String strValue = "{'id':'123'}";
    final ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getCollectionOfObjectValues(any())).thenReturn(new ArrayList<Parsable>() {

        {
            add(new TestEntity() {

                {
                    setId("123");
                }
            });
        }
    });
    final ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
    ParseNodeFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockParseNodeFactory);
    final var result = KiotaSerialization.deserializeCollection(_jsonContentType, strValue, TestEntity::createFromDiscriminatorValue);
    assertEquals("123", result.get(0).getId());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ParseNodeFactory createMockParseNodeFactory(ParseNode mockParseNode) {
    ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
    return mockParseNodeFactory;
}
```
</details>

---
## Mock Clone Instance #kiota-java_MCI_13
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.serialization.ParseNode`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
private static ParseNode createMockParseNode() {
    ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any())).thenReturn(new TestEntity() {
        {
            setId("123");
        }
    });
    return mockParseNode;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_13_1
#### Test Case Name: `deserializesObjectWithoutReflection`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\serialization\DeserializationHelpersTest.java`)
#### Mock Object Variable Name: `mockParseNode`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final String strValue = "{'id':'123'}";
-    final ParseNode mockParseNode = mock(ParseNode.class);
-    when(mockParseNode.getObjectValue(any())).thenReturn(new TestEntity() {
-
-        {
-            setId("123");
-        }
-    });
+    final ParseNode mockParseNode = createMockParseNode();
    final ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
    ParseNodeFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockParseNodeFactory);
    final var result = KiotaSerialization.deserialize(_jsonContentType, strValue, TestEntity::createFromDiscriminatorValue);
    assertEquals("123", result.getId());
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void deserializesObjectWithoutReflection() throws IOException {
    final String strValue = "{'id':'123'}";
    final ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any())).thenReturn(new TestEntity() {

        {
            setId("123");
        }
    });
    final ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
    ParseNodeFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockParseNodeFactory);
    final var result = KiotaSerialization.deserialize(_jsonContentType, strValue, TestEntity::createFromDiscriminatorValue);
    assertEquals("123", result.getId());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ParseNode createMockParseNode() {
    ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any())).thenReturn(new TestEntity() {
        {
            setId("123");
        }
    });
    return mockParseNode;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_13_2
#### Test Case Name: `deserializesObjectWithReflection`(File: `C:\Java_projects\Microsoft\kiota-java\components\abstractions\src\test\java\com\microsoft\kiota\serialization\DeserializationHelpersTest.java`)
#### Mock Object Variable Name: `mockParseNode`
<summary>Suggested Diff</summary>

```diff
--- original
+++ refactored
@@
    final String strValue = "{'id':'123'}";
-    final ParseNode mockParseNode = mock(ParseNode.class);
-    when(mockParseNode.getObjectValue(any())).thenReturn(new TestEntity() {
-
-        {
-            setId("123");
-        }
-    });
+    final ParseNode mockParseNode = createMockParseNode();
    final ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
    ParseNodeFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockParseNodeFactory);
    final var result = KiotaSerialization.deserialize(_jsonContentType, strValue, TestEntity::createFromDiscriminatorValue);
    assertEquals("123", result.getId());
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@Test
void deserializesObjectWithReflection() throws IOException {
    final String strValue = "{'id':'123'}";
    final ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any())).thenReturn(new TestEntity() {

        {
            setId("123");
        }
    });
    final ParseNodeFactory mockParseNodeFactory = mock(ParseNodeFactory.class);
    when(mockParseNodeFactory.getParseNode(any(), any())).thenReturn(mockParseNode);
    ParseNodeFactoryRegistry.defaultInstance.contentTypeAssociatedFactories.put(_jsonContentType, mockParseNodeFactory);
    final var result = KiotaSerialization.deserialize(_jsonContentType, strValue, TestEntity::createFromDiscriminatorValue);
    assertEquals("123", result.getId());
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ParseNode createMockParseNode() {
    ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any())).thenReturn(new TestEntity() {
        {
            setId("123");
        }
    });
    return mockParseNode;
}
```
</details>

---
## Mock Clone Instance #kiota-java_MCI_14
- **Scope**: method level
- **Mocked Class**: `com.microsoft.kiota.serialization.ParseNode`
- **Test Case Count**: 2
- **MO Count**: 2

### Reusable Method
```java
private static ParseNode createMockParseNode(Parsable mockEntity) {
    ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
    return mockParseNode;
}
```

### The refactoring details in each test cases
---
#### Test Case ID #kiota-java_Test_14_1
#### Test Case Name: `sendReturnsObjectOnContent`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `mockParseNode`
<summary>Suggested Diff</summary>

```diff
@@
    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
-    final var mockParseNode = mock(ParseNode.class);
-    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
+    final var mockParseNode = createMockParseNode(mockEntity);
    final var mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn("application/json");
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@ParameterizedTest
@ValueSource(ints = { 200, 201, 202, 203 })
void sendReturnsObjectOnContent(int statusCode) throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
    final var client = getMockClient(new Response.Builder().code(statusCode).message("OK").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(ResponseBody.create("test".getBytes("UTF-8"), MediaType.parse("application/json"))).build());
    final var requestInformation = new RequestInformation() {

        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
    final var mockEntity = mock(Parsable.class);
    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
    final var mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
    final var mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn("application/json");
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
    final var response = requestAdapter.send(requestInformation, null, (node) -> mockEntity);
    assertNotNull(response);
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ParseNode createMockParseNode(Parsable mockEntity) {
    ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
    return mockParseNode;
}
```
</details>

---
#### Test Case ID #kiota-java_Test_14_2
#### Test Case Name: `throwsAPIException`(File: `C:\Java_projects\Microsoft\kiota-java\components\http\okHttp\src\test\java\com\microsoft\kiota\http\OkHttpRequestAdapterTest.java`)
#### Mock Object Variable Name: `mockParseNode`
<summary>Suggested Diff</summary>

```diff
@@
    when(mockParsableFactory.create(any(ParseNode.class))).thenReturn(mockEntity);
-    final var mockParseNode = mock(ParseNode.class);
-    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
+    final var mockParseNode = createMockParseNode(mockEntity);
    final var mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn("application/json");
@@
```

<details><summary>Original Test Code (click to expand)</summary>

```java
@SuppressWarnings("unchecked")
@ParameterizedTest
@MethodSource("providesErrorMappings")
void throwsAPIException(int responseStatusCode, List<String> errorMappingCodes, boolean expectDeserializedException) throws Exception {
    final var authenticationProviderMock = mock(AuthenticationProvider.class);
    authenticationProviderMock.authenticateRequest(any(RequestInformation.class), any(Map.class));
    final var client = getMockClient(new Response.Builder().code(responseStatusCode).message("Not Found").protocol(Protocol.HTTP_1_1).request(new Request.Builder().url("http://localhost").build()).body(ResponseBody.create("test".getBytes("UTF-8"), MediaType.parse("application/json"))).header("request-id", "request-id-value").build());
    final var requestInformation = new RequestInformation() {

        {
            setUri(new URI("https://localhost"));
            httpMethod = HttpMethod.GET;
        }
    };
    final var mockEntity = mock(Parsable.class);
    when(mockEntity.getFieldDeserializers()).thenReturn(new HashMap<>());
    final var mockParsableFactory = mock(ParsableFactory.class);
    when(mockParsableFactory.create(any(ParseNode.class))).thenReturn(mockEntity);
    final var mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
    final var mockFactory = mock(ParseNodeFactory.class);
    when(mockFactory.getParseNode(any(String.class), any(InputStream.class))).thenReturn(mockParseNode);
    when(mockFactory.getValidContentType()).thenReturn("application/json");
    final var requestAdapter = new OkHttpRequestAdapter(authenticationProviderMock, mockFactory, null, client);
    final var errorMappings = errorMappingCodes == null ? null : new HashMap<String, ParsableFactory<? extends Parsable>>();
    if (errorMappings != null)
        errorMappingCodes.forEach((mapping) -> errorMappings.put(mapping, mockParsableFactory));
    final var exception = assertThrows(ApiException.class, () -> requestAdapter.send(requestInformation, errorMappings, (node) -> mockEntity));
    assertNotNull(exception);
    if (expectDeserializedException)
        verify(mockParseNode, times(1)).getObjectValue(mockParsableFactory);
    assertEquals(responseStatusCode, exception.getResponseStatusCode());
    assertTrue(exception.getResponseHeaders().containsKey("request-id"));
}
```
</details>
<details><summary>Reusable Method for MCI (click to expand)</summary>

```java
private static ParseNode createMockParseNode(Parsable mockEntity) {
    ParseNode mockParseNode = mock(ParseNode.class);
    when(mockParseNode.getObjectValue(any(ParsableFactory.class))).thenReturn(mockEntity);
    return mockParseNode;
}
```
</details>

---