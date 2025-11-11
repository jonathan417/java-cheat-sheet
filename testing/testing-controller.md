# Testing Controllers (REST APIs)

> **Test HTTP endpoints, status codes, and request/response handling**

---

## 🎯 When to Test?

✅ **Always test REST controllers** - Test all endpoints

## 📝 What to Test?

- HTTP status codes (200, 404, 400, 500)
- Request validation
- Response body structure
- Path variables and query parameters
- Authentication/Authorization (if applicable)

---

## 🔧 Setup

**Annotation:** `@WebMvcTest(YourController.class)`

**Dependencies to Mock:** Use `@MockBean` for services

---

## 📖 Example: User Controller

### Controller Implementation

**Java:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
        User user = userService.getUserById(id);
        return ResponseEntity.ok(new UserResponse(user));
    }
    
    @PostMapping
    public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
        User user = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(new UserResponse(user));
    }
}
```

**Kotlin:**
```kotlin
@RestController
@RequestMapping("/api/users")
class UserController(
    private val userService: UserService
) {
    
    @GetMapping("/{id}")
    fun getUser(@PathVariable id: Long): ResponseEntity<UserResponse> {
        val user = userService.getUserById(id)
        return ResponseEntity.ok(UserResponse(user))
    }
    
    @PostMapping
    fun createUser(@Valid @RequestBody request: CreateUserRequest): ResponseEntity<UserResponse> {
        val user = userService.createUser(request)
        return ResponseEntity.status(HttpStatus.CREATED).body(UserResponse(user))
    }
}
```

---

## 🧪 Controller Tests

### Java Example

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void shouldGetUser_Returns200_WhenExists() throws Exception {
        // GIVEN
        User user = new User(1L, "test@example.com", "John");
        when(userService.getUserById(1L)).thenReturn(user);
        
        // WHEN & THEN
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("John"))
            .andExpect(jsonPath("$.email").value("test@example.com"));
        
        verify(userService).getUserById(1L);
    }
    
    @Test
    void shouldGetUser_Returns404_WhenNotFound() throws Exception {
        // GIVEN
        when(userService.getUserById(999L))
            .thenThrow(new UserNotFoundException("User not found"));
        
        // WHEN & THEN
        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound());
    }
    
    @Test
    void shouldCreateUser_Returns201_WhenValid() throws Exception {
        // GIVEN
        User user = new User(1L, "test@example.com", "John");
        when(userService.createUser(any())).thenReturn(user);
        
        // WHEN & THEN
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"email\":\"test@example.com\",\"name\":\"John\"}"))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.name").value("John"));
    }
    
    @Test
    void shouldCreateUser_Returns400_WhenInvalidEmail() throws Exception {
        // WHEN & THEN
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"email\":\"invalid-email\",\"name\":\"John\"}"))
            .andExpect(status().isBadRequest());
    }
}
```

### Kotlin Example

```kotlin
@WebMvcTest(UserController::class)
class UserControllerTest {
    
    @Autowired
    private lateinit var mockMvc: MockMvc
    
    @MockBean
    private lateinit var userService: UserService
    
    @Test
    fun `should return 200 when user exists`() {
        // GIVEN
        val user = User(1L, "test@example.com", "John")
        whenever(userService.getUserById(1L)).thenReturn(user)
        
        // WHEN & THEN
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk)
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("John"))
            .andExpect(jsonPath("$.email").value("test@example.com"))
        
        verify(userService).getUserById(1L)
    }
    
    @Test
    fun `should return 404 when user not found`() {
        // GIVEN
        whenever(userService.getUserById(999L))
            .thenThrow(UserNotFoundException("User not found"))
        
        // WHEN & THEN
        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound)
    }
    
    @Test
    fun `should return 201 when creating valid user`() {
        // GIVEN
        val user = User(1L, "test@example.com", "John")
        whenever(userService.createUser(any())).thenReturn(user)
        
        // WHEN & THEN
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""{"email":"test@example.com","name":"John"}"""))
            .andExpect(status().isCreated)
            .andExpect(jsonPath("$.name").value("John"))
    }
    
    @Test
    fun `should return 400 when email is invalid`() {
        // WHEN & THEN
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""{"email":"invalid-email","name":"John"}"""))
            .andExpect(status().isBadRequest)
    }
}
```

---

## 🎯 Common Test Scenarios

### 1. GET Endpoint Tests
- ✅ 200 OK - Resource exists
- ✅ 404 Not Found - Resource doesn't exist
- ✅ Path variables work correctly

### 2. POST Endpoint Tests
- ✅ 201 Created - Valid request
- ✅ 400 Bad Request - Invalid data
- ✅ 409 Conflict - Duplicate resource

### 3. PUT/PATCH Endpoint Tests
- ✅ 200 OK - Update successful
- ✅ 404 Not Found - Resource doesn't exist
- ✅ 400 Bad Request - Invalid data

### 4. DELETE Endpoint Tests
- ✅ 204 No Content - Delete successful
- ✅ 404 Not Found - Resource doesn't exist

---

## 💡 Pro Tips

- Use `jsonPath()` to verify response structure
- Test both success and error cases
- Mock service layer completely
- Don't test business logic here (test in service)
- Keep tests focused on HTTP layer
