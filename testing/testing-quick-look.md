# Complete Mockito & JUnit Testing Reference

> **For Beginners**: This guide covers all essential testing functions in Java and Kotlin. Use this as your complete reference when writing tests.

---

## 🔧 Test Class Setup Annotations

| Annotation | Java Example | Kotlin Example | Description |
|------------|-------------|----------------|-------------|
| `@ExtendWith(MockitoExtension.class)` | `@ExtendWith(MockitoExtension.class) class MyTest {}` | `@ExtendWith(MockitoExtension::class) class MyTest` | Enable Mockito for JUnit 5 (pure unit tests, ⚡ very fast) |
| `@SpringBootTest` | `@SpringBootTest class MyTest {}` | `@SpringBootTest class MyTest` | Load full Spring application context for integration tests (🐢 slow, use sparingly) |
| `@WebMvcTest` | `@WebMvcTest(UserController.class) class MyTest {}` | `@WebMvcTest(UserController::class) class MyTest` | Load only web layer (controllers) for testing, lighter than @SpringBootTest |
| `@DataJpaTest` | `@DataJpaTest class MyTest {}` | `@DataJpaTest class MyTest` | Load only JPA components for repository testing |
| `@TestConfiguration` | `@TestConfiguration class TestConfig {}` | `@TestConfiguration class TestConfig` | Define additional beans for testing |

### 🎯 Decision Guide - Which Annotation Should I Use?

```
┌─────────────────────────────────────────────────────────────┐
│ What are you testing?                                       │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
    Service/          Controller/          Repository/
    Business Logic      REST API             Database
        │                   │                   │
        ↓                   ↓                   ↓
  @ExtendWith           @WebMvcTest         @DataJpaTest
  (MockitoExtension)    
                      
  ⚡ FASTEST            ⚡ FAST              ⚡ FAST
  (milliseconds)       (1-2 seconds)       (1-2 seconds)
  
  Use @Mock            Use @MockBean        Use TestEntityManager
  + @InjectMocks       for dependencies     Auto-configures DB
  
  80% of tests         15% of tests         4% of tests
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                Need to test everything together?
                (Full Spring context + all beans)
                            │
                            ↓
                    @SpringBootTest
                    
                    🐢 SLOWEST
                    (5-10 seconds per test)
                    
                    1% of tests
                    (End-to-end integration)
```

**💡 Quick Rules:**
- **Default choice**: `@ExtendWith(MockitoExtension.class)` - Fast unit tests
- **Testing REST endpoints**: `@WebMvcTest` - Only loads web layer
- **Testing database queries**: `@DataJpaTest` - Only loads JPA layer
- **Last resort**: `@SpringBootTest` - Full integration (use sparingly!)

**📊 Recommended Test Distribution:**
- 80% Unit tests (`@ExtendWith(MockitoExtension.class)`)
- 15% Slice tests (`@WebMvcTest`, `@DataJpaTest`)
- 5% Integration tests (`@SpringBootTest`)

---

## 🔄 Test Method Lifecycle Annotations

| Annotation | Java Example | Kotlin Example | Description |
|------------|-------------|----------------|-------------|
| `@Test` | `@Test void testMethod() {}` | `@Test fun testMethod() {}` | Mark test method |
| `@BeforeEach` | `@BeforeEach void setup() {}` | `@BeforeEach fun setup() {}` | Run before each test |
| `@AfterEach` | `@AfterEach void teardown() {}` | `@AfterEach fun teardown() {}` | Run after each test |
| `@BeforeAll` | `@BeforeAll static void init() {}` | `@BeforeAll @JvmStatic fun init() {}` | Run once before all tests (static) |
| `@AfterAll` | `@AfterAll static void cleanup() {}` | `@AfterAll @JvmStatic fun cleanup() {}` | Run once after all tests (static) |

---

## 🎯 GIVEN Stage - Mock Creation & Setup

| Function | Java Example | Kotlin Example | Description |
|----------|-------------|----------------|-------------|
| `mock(Class)` | `UserService service = mock(UserService.class);` | `val service = mock<UserService>()` | Create a mock object |
| `spy(Object)` | `List<String> list = spy(new ArrayList<>());` | `val list = spy(ArrayList<String>())` | Create a partial mock (keeps real methods) |
| `@Mock` | `@Mock private UserService service;` | `@Mock private lateinit var service: UserService` | Annotation to auto-create mock |
| `@Spy` | `@Spy private List<String> list = new ArrayList<>();` | `@Spy private var list = ArrayList<String>()` | Annotation to create spy object |
| `@InjectMocks` | `@InjectMocks private UserController controller;` | `@InjectMocks private lateinit var controller: UserController` | Auto-inject mocks into the class under test (e.g., inject mocked UserService into UserController). ⚡ Fast, no Spring context. Use with `@ExtendWith(MockitoExtension.class)` |
| `@Autowired` | `@Autowired private UserController controller;` | `@Autowired private lateinit var controller: UserController` | Inject from Spring context. 🐢 Slower (loads Spring). Use with `@SpringBootTest` or `@WebMvcTest` + `@MockBean` for dependencies |
| `@MockBean` | `@MockBean private UserService service;` | `@MockBean private lateinit var service: UserService` | Spring-specific: Replace a Spring bean with a mock in the application context |
| `@SpyBean` | `@SpyBean private UserService service;` | `@SpyBean private lateinit var service: UserService` | Spring-specific: Wrap an existing Spring bean with a spy |
| `@Captor` | `@Captor private ArgumentCaptor<String> captor;` | `@Captor private lateinit var captor: ArgumentCaptor<String>` | Create ArgumentCaptor |

---

## 🎬 GIVEN Stage - Stubbing Behavior

| Function | Java Example | Kotlin Example | Description |
|----------|-------------|----------------|-------------|
| `when().thenReturn()` | `when(service.getUser()).thenReturn(user);` | `whenever(service.getUser()).thenReturn(user)` | Set mock method return value |
| `when().thenThrow()` | `when(service.getUser()).thenThrow(new RuntimeException());` | `whenever(service.getUser()).thenThrow(RuntimeException())` | Set mock method to throw exception |
| `when().thenAnswer()` | `when(service.process(any())).thenAnswer(inv -> inv.getArgument(0));` | `whenever(service.process(any())).thenAnswer { it.arguments[0] }` | Custom return logic or complex behavior |
| `when().thenCallRealMethod()` | `when(spy.method()).thenCallRealMethod();` | `whenever(spy.method()).thenCallRealMethod()` | Call the real method |
| `doReturn().when()` | `doReturn(user).when(service).getUser();` | `doReturn(user).whenever(service).getUser()` | Alternative when() syntax, for void methods |
| `doThrow().when()` | `doThrow(new RuntimeException()).when(service).delete();` | `doThrow(RuntimeException()).whenever(service).delete()` | Set void method to throw exception |
| `doNothing().when()` | `doNothing().when(service).update();` | `doNothing().whenever(service).update()` | Set void method to do nothing |
| `doAnswer().when()` | `doAnswer(inv -> { /*logic*/ return null; }).when(service).save();` | `doAnswer { /*logic*/ }.whenever(service).save()` | Custom void method behavior |
| `doCallRealMethod().when()` | `doCallRealMethod().when(spy).method();` | `doCallRealMethod().whenever(spy).method()` | Call real method instead of mock |

---

## 🔍 GIVEN Stage - Argument Matchers

| Function | Java Example | Kotlin Example | Description |
|----------|-------------|----------------|-------------|
| `any()` | `when(service.save(any())).thenReturn(true);` | `whenever(service.save(any())).thenReturn(true)` | Match any object |
| `anyInt()`, `anyLong()`, `anyDouble()` | `when(service.calculate(anyInt())).thenReturn(42);` | `whenever(service.calculate(anyInt())).thenReturn(42)` | Match any numeric type |
| `anyString()` | `when(service.find(anyString())).thenReturn(user);` | `whenever(service.find(anyString())).thenReturn(user)` | Match any string |
| `anyBoolean()` | `when(service.toggle(anyBoolean())).thenReturn(true);` | `whenever(service.toggle(anyBoolean())).thenReturn(true)` | Match any boolean |
| `anyList()`, `anyMap()`, `anySet()` | `when(service.process(anyList())).thenReturn(result);` | `whenever(service.process(anyList())).thenReturn(result)` | Match any collection |
| `eq()` | `when(service.get(eq("id"), any())).thenReturn(user);` | `whenever(service.get(eq("id"), any())).thenReturn(user)` | Match exact parameter value |
| `isNull()`, `isNotNull()` | `when(service.find(isNull())).thenReturn(null);` | `whenever(service.find(isNull())).thenReturn(null)` | Match null value |
| `contains()` | `when(service.search(contains("test"))).thenReturn(list);` | `whenever(service.search(contains("test"))).thenReturn(list)` | String contains match |
| `startsWith()`, `endsWith()` | `when(service.find(startsWith("user"))).thenReturn(user);` | `whenever(service.find(startsWith("user"))).thenReturn(user)` | String prefix/suffix match |
| `argThat()` | `when(service.save(argThat(u -> u.getId() > 0))).thenReturn(true);` | `whenever(service.save(argThat { it.id > 0 })).thenReturn(true)` | Custom argument matcher |

---

## ⚡ WHEN Stage - Execute Test

| Action | Java Example | Kotlin Example | Description |
|--------|-------------|----------------|-------------|
| Execute test method | `User result = controller.getUser("123");` | `val result = controller.getUser("123")` | Call the business logic to test |

---

## ✅ THEN Stage - Verification

| Function | Java Example | Kotlin Example | Description |
|----------|-------------|----------------|-------------|
| `verify()` | `verify(service).getUser();` | `verify(service).getUser()` | Verify method was called |
| `verify(times(n))` | `verify(service, times(3)).save(any());` | `verify(service, times(3)).save(any())` | Verify called n times |
| `verify(never())` | `verify(service, never()).delete();` | `verify(service, never()).delete()` | Verify never called |
| `verify(atLeastOnce())` | `verify(service, atLeastOnce()).update();` | `verify(service, atLeastOnce()).update()` | Verify called at least once |
| `verify(atLeast(n))` | `verify(service, atLeast(2)).process();` | `verify(service, atLeast(2)).process()` | Verify called at least n times |
| `verify(atMost(n))` | `verify(service, atMost(5)).log();` | `verify(service, atMost(5)).log()` | Verify called at most n times |
| `verifyNoMoreInteractions()` | `verifyNoMoreInteractions(service);` | `verifyNoMoreInteractions(service)` | Verify no other interactions |
| `verifyNoInteractions()` | `verifyNoInteractions(service);` | `verifyNoInteractions(service)` | Verify no interactions at all |
| `inOrder().verify()` | `InOrder order = inOrder(service); order.verify(service).create(); order.verify(service).update();` | `inOrder(service) { verify(service).create(); verify(service).update() }` | Verify method call order |
| `ArgumentCaptor.capture()` | `verify(service).save(captor.capture()); String value = captor.getValue();` | `verify(service).save(captor.capture()); val value = captor.value` | Capture method argument value |

---

## 🎯 THEN Stage - JUnit Assertions

| Function | Java Example | Kotlin Example | Description |
|----------|-------------|----------------|-------------|
| `assertEquals()` | `assertEquals(expected, actual);` | `assertEquals(expected, actual)` | Verify two values are equal |
| `assertNotEquals()` | `assertNotEquals(expected, actual);` | `assertNotEquals(expected, actual)` | Verify values are not equal |
| `assertTrue()`, `assertFalse()` | `assertTrue(result);` | `assertTrue(result)` | Verify boolean condition |
| `assertNull()`, `assertNotNull()` | `assertNull(value);` | `assertNull(value)` | Verify null value |
| `assertSame()`, `assertNotSame()` | `assertSame(obj1, obj2);` | `assertSame(obj1, obj2)` | Verify same object reference |
| `assertThrows()` | `assertThrows(RuntimeException.class, () -> service.fail());` | `assertThrows<RuntimeException> { service.fail() }` | Verify exception is thrown |
| `assertArrayEquals()` | `assertArrayEquals(new int[]{1,2}, actual);` | `assertArrayEquals(intArrayOf(1,2), actual)` | Verify arrays are equal |
| `assertIterableEquals()` | `assertIterableEquals(list1, list2);` | `assertIterableEquals(list1, list2)` | Verify Iterable deep equality |
| `assertLinesMatch()` | `assertLinesMatch(expected, actual);` | `assertLinesMatch(expected, actual)` | Verify multi-line text match |
| `assertAll()` | `assertAll(() -> assertEquals(1, a), () -> assertEquals(2, b));` | `assertAll({ assertEquals(1, a) }, { assertEquals(2, b) })` | Execute multiple assertions |
| `assertTimeout()` | `assertTimeout(Duration.ofSeconds(1), () -> service.run());` | `assertTimeout(Duration.ofSeconds(1)) { service.run() }` | Verify execution time |
| `assertTimeoutPreemptively()` | `assertTimeoutPreemptively(Duration.ofSeconds(1), () -> service.run());` | `assertTimeoutPreemptively(Duration.ofSeconds(1)) { service.run() }` | Verify execution time (terminates early) |

---

## 📝 Quick Start Template

### Java
```java
@ExtendWith(MockitoExtension.class)
class UserControllerTest {
    @Mock private UserService service;          // Mock dependency
    @InjectMocks private UserController controller;  // Class under test (injects service)
    
    @Test
    void shouldGetUserById() {
        // GIVEN
        User mockUser = new User("123", "John");
        when(service.getUserById(anyString())).thenReturn(mockUser);
        
        // WHEN
        User result = controller.getUser("123");
        
        // THEN
        assertEquals("John", result.getName());
        verify(service).getUserById("123");
    }
}
```

### Kotlin
```kotlin
@ExtendWith(MockitoExtension::class)
class UserControllerTest {
    @Mock private lateinit var service: UserService          // Mock dependency
    @InjectMocks private lateinit var controller: UserController  // Class under test (injects service)
    
    @Test
    fun `should get user by id`() {
        // GIVEN
        val mockUser = User("123", "John")
        whenever(service.getUserById(anyString())).thenReturn(mockUser)
        
        // WHEN
        val result = controller.getUser("123")
        
        // THEN
        assertEquals("John", result.name)
        verify(service).getUserById("123")
    }
}
```

---

**💡 Tip**: Use `whenever()` in Kotlin instead of `when()` to avoid keyword conflicts!

**💡 Recommendation**: Use `@InjectMocks` for fast unit tests, `@Autowired` only when you need Spring features!