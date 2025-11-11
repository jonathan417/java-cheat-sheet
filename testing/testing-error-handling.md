# Testing Error Handling

> **Test exceptions, validation, and error responses**

---

## 🎯 Why Test Error Handling?

Error handling is **critical** - users see these messages!

## 📝 What to Test?

- Custom exceptions are thrown
- Global exception handlers return correct responses
- Validation errors return proper messages
- HTTP status codes are correct
- Error response structure is consistent

---

## 📖 Example: Custom Exceptions

### Exception Classes

**Java:**
```java
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}

public class UserAlreadyExistsException extends RuntimeException {
    public UserAlreadyExistsException(String message) {
        super(message);
    }
}
```

**Kotlin:**
```kotlin
class UserNotFoundException(message: String) : RuntimeException(message)

class UserAlreadyExistsException(message: String) : RuntimeException(message)
```

---

## 📖 Example: Global Exception Handler

### Exception Handler

**Java:**
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleUserNotFound(UserNotFoundException ex) {
        return new ErrorResponse("USER_NOT_FOUND", ex.getMessage());
    }
    
    @ExceptionHandler(UserAlreadyExistsException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleUserAlreadyExists(UserAlreadyExistsException ex) {
        return new ErrorResponse("USER_ALREADY_EXISTS", ex.getMessage());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidationErrors(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(FieldError::getDefaultMessage)
            .collect(Collectors.toList());
        
        return new ErrorResponse("VALIDATION_ERROR", errors.toString());
    }
}
```

**Kotlin:**
```kotlin
@RestControllerAdvice
class GlobalExceptionHandler {
    
    @ExceptionHandler(UserNotFoundException::class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    fun handleUserNotFound(ex: UserNotFoundException): ErrorResponse {
        return ErrorResponse("USER_NOT_FOUND", ex.message ?: "User not found")
    }
    
    @ExceptionHandler(UserAlreadyExistsException::class)
    @ResponseStatus(HttpStatus.CONFLICT)
    fun handleUserAlreadyExists(ex: UserAlreadyExistsException): ErrorResponse {
        return ErrorResponse("USER_ALREADY_EXISTS", ex.message ?: "User already exists")
    }
    
    @ExceptionHandler(MethodArgumentNotValidException::class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    fun handleValidationErrors(ex: MethodArgumentNotValidException): ErrorResponse {
        val errors = ex.bindingResult.fieldErrors
            .map { it.defaultMessage }
            .joinToString()
        
        return ErrorResponse("VALIDATION_ERROR", errors)
    }
}
```

---

## 🧪 Testing Service Layer Exceptions

### Java Example

```java
@ExtendWith(MockitoExtension.class)
class UserServiceErrorTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void shouldThrowUserNotFoundException_WhenUserNotFound() {
        // GIVEN
        when(userRepository.findById(999L)).thenReturn(Optional.empty());
        
        // WHEN & THEN
        UserNotFoundException exception = assertThrows(
            UserNotFoundException.class,
            () -> userService.getUserById(999L)
        );
        
        assertTrue(exception.getMessage().contains("999"));
    }
    
    @Test
    void shouldThrowUserAlreadyExistsException_WhenEmailExists() {
        // GIVEN
        User existingUser = new User("test@example.com", "Jane");
        when(userRepository.findByEmail("test@example.com"))
            .thenReturn(Optional.of(existingUser));
        
        // WHEN & THEN
        UserAlreadyExistsException exception = assertThrows(
            UserAlreadyExistsException.class,
            () -> userService.createUser(new CreateUserRequest("test@example.com", "John"))
        );
        
        assertTrue(exception.getMessage().contains("exists"));
    }
}
```

### Kotlin Example

```kotlin
@ExtendWith(MockitoExtension::class)
class UserServiceErrorTest {
    
    @Mock
    private lateinit var userRepository: UserRepository
    
    @InjectMocks
    private lateinit var userService: UserService
    
    @Test
    fun `should throw UserNotFoundException when user not found`() {
        // GIVEN
        whenever(userRepository.findById(999L)).thenReturn(Optional.empty())
        
        // WHEN & THEN
        val exception = assertThrows<UserNotFoundException> {
            userService.getUserById(999L)
        }
        
        assertTrue(exception.message!!.contains("999"))
    }
    
    @Test
    fun `should throw UserAlreadyExistsException when email exists`() {
        // GIVEN
        val existingUser = User("test@example.com", "Jane")
        whenever(userRepository.findByEmail("test@example.com"))
            .thenReturn(Optional.of(existingUser))
        
        // WHEN & THEN
        val exception = assertThrows<UserAlreadyExistsException> {
            userService.createUser(CreateUserRequest("test@example.com", "John"))
        }
        
        assertTrue(exception.message!!.contains("exists"))
    }
}
```

---

## 🧪 Testing Controller Error Responses

### Java Example

```java
@WebMvcTest(UserController.class)
class UserControllerErrorTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void shouldReturn404_WhenUserNotFound() throws Exception {
        // GIVEN
        when(userService.getUserById(999L))
            .thenThrow(new UserNotFoundException("User not found: 999"));
        
        // WHEN & THEN
        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.code").value("USER_NOT_FOUND"))
            .andExpect(jsonPath("$.message").value("User not found: 999"));
    }
    
    @Test
    void shouldReturn409_WhenUserAlreadyExists() throws Exception {
        // GIVEN
        when(userService.createUser(any()))
            .thenThrow(new UserAlreadyExistsException("Email already exists"));
        
        // WHEN & THEN
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"email\":\"test@example.com\",\"name\":\"John\"}"))
            .andExpect(status().isConflict())
            .andExpect(jsonPath("$.code").value("USER_ALREADY_EXISTS"));
    }
    
    @Test
    void shouldReturn400_WhenValidationFails() throws Exception {
        // WHEN & THEN
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"email\":\"invalid-email\",\"name\":\"\"}"))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value("VALIDATION_ERROR"));
    }
}
```

### Kotlin Example

```kotlin
@WebMvcTest(UserController::class)
class UserControllerErrorTest {
    
    @Autowired
    private lateinit var mockMvc: MockMvc
    
    @MockBean
    private lateinit var userService: UserService
    
    @Test
    fun `should return 404 when user not found`() {
        // GIVEN
        whenever(userService.getUserById(999L))
            .thenThrow(UserNotFoundException("User not found: 999"))
        
        // WHEN & THEN
        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound)
            .andExpect(jsonPath("$.code").value("USER_NOT_FOUND"))
            .andExpect(jsonPath("$.message").value("User not found: 999"))
    }
    
    @Test
    fun `should return 409 when user already exists`() {
        // GIVEN
        whenever(userService.createUser(any()))
            .thenThrow(UserAlreadyExistsException("Email already exists"))
        
        // WHEN & THEN
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""{"email":"test@example.com","name":"John"}"""))
            .andExpect(status().isConflict)
            .andExpect(jsonPath("$.code").value("USER_ALREADY_EXISTS"))
    }
    
    @Test
    fun `should return 400 when validation fails`() {
        // WHEN & THEN
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""{"email":"invalid-email","name":""}"""))
            .andExpect(status().isBadRequest)
            .andExpect(jsonPath("$.code").value("VALIDATION_ERROR"))
    }
}
```

---

## 🎯 Error Testing Checklist

### For Each Error Case:

1. **✅ Service throws correct exception**
2. **✅ Controller returns correct HTTP status**
3. **✅ Error response has correct structure**
4. **✅ Error message is user-friendly**
5. **✅ Validation errors include field details**

---

## 📊 Common HTTP Status Codes

| Error Type | Status Code | When to Use |
|------------|-------------|-------------|
| **Not Found** | 404 | Resource doesn't exist |
| **Bad Request** | 400 | Invalid input/validation error |
| **Conflict** | 409 | Duplicate resource (e.g., email exists) |
| **Unauthorized** | 401 | Authentication required |
| **Forbidden** | 403 | No permission for this action |
| **Internal Server Error** | 500 | Unexpected server error |

---

## 💡 Pro Tips

- **Test exceptions in services** - Use `assertThrows()`
- **Test error responses in controllers** - Check status + body
- **Verify error messages** - Make sure they're helpful
- **Test validation errors** - @Valid annotations work
- **Use descriptive exception messages** - Include IDs, names, etc.
- **Keep error responses consistent** - Same structure everywhere
