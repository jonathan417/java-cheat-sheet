# Testing Services (Business Logic)

> **Most important tests! Test your core business logic thoroughly**

---

## 🎯 When to Test?

✅ **ALWAYS!** Services contain your core business logic - **90%+ coverage goal**

## 📝 What to Test?

- Business logic correctness
- Error handling and exceptions
- Different input scenarios (happy path + edge cases)
- Interactions with dependencies
- Validation rules
- Transaction behavior

---

## 🔧 Setup

**Annotation:** `@ExtendWith(MockitoExtension.class)` - Pure unit test

**Dependencies:** Mock with `@Mock`, inject with `@InjectMocks`

---

## 📖 Example: User Service

### Service Implementation

**Java:**
```java
@Service
public class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    public User createUser(CreateUserRequest request) {
        // Check if email exists
        if (userRepository.findByEmail(request.getEmail()).isPresent()) {
            throw new UserAlreadyExistsException("Email already exists");
        }
        
        // Create user
        User user = new User(request.getEmail(), request.getName());
        User saved = userRepository.save(user);
        
        // Send welcome email
        emailService.sendWelcomeEmail(user.getEmail());
        
        return saved;
    }
    
    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
    }
}
```

**Kotlin:**
```kotlin
@Service
class UserService(
    private val userRepository: UserRepository,
    private val emailService: EmailService
) {
    
    fun createUser(request: CreateUserRequest): User {
        // Check if email exists
        if (userRepository.findByEmail(request.email).isPresent) {
            throw UserAlreadyExistsException("Email already exists")
        }
        
        // Create user
        val user = User(request.email, request.name)
        val saved = userRepository.save(user)
        
        // Send welcome email
        emailService.sendWelcomeEmail(user.email)
        
        return saved
    }
    
    fun getUserById(id: Long): User {
        return userRepository.findById(id)
            .orElseThrow { UserNotFoundException("User not found: $id") }
    }
}
```

---

## 🧪 Service Tests

### Java Example

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void shouldCreateUser_Success() {
        // GIVEN
        CreateUserRequest request = new CreateUserRequest("test@example.com", "John");
        User user = new User("test@example.com", "John");
        
        when(userRepository.findByEmail("test@example.com")).thenReturn(Optional.empty());
        when(userRepository.save(any(User.class))).thenReturn(user);
        
        // WHEN
        User result = userService.createUser(request);
        
        // THEN
        assertNotNull(result);
        assertEquals("John", result.getName());
        verify(userRepository).save(any(User.class));
        verify(emailService).sendWelcomeEmail("test@example.com");
    }
    
    @Test
    void shouldCreateUser_ThrowsException_WhenEmailExists() {
        // GIVEN
        CreateUserRequest request = new CreateUserRequest("test@example.com", "John");
        User existingUser = new User("test@example.com", "Jane");
        
        when(userRepository.findByEmail("test@example.com"))
            .thenReturn(Optional.of(existingUser));
        
        // WHEN & THEN
        assertThrows(UserAlreadyExistsException.class, () -> {
            userService.createUser(request);
        });
        
        verify(userRepository, never()).save(any());
        verify(emailService, never()).sendWelcomeEmail(anyString());
    }
    
    @Test
    void shouldGetUser_Success() {
        // GIVEN
        User user = new User("test@example.com", "John");
        when(userRepository.findById(1L)).thenReturn(Optional.of(user));
        
        // WHEN
        User result = userService.getUserById(1L);
        
        // THEN
        assertNotNull(result);
        assertEquals("John", result.getName());
        verify(userRepository).findById(1L);
    }
    
    @Test
    void shouldGetUser_ThrowsException_WhenNotFound() {
        // GIVEN
        when(userRepository.findById(999L)).thenReturn(Optional.empty());
        
        // WHEN & THEN
        UserNotFoundException exception = assertThrows(UserNotFoundException.class, () -> {
            userService.getUserById(999L);
        });
        
        assertTrue(exception.getMessage().contains("999"));
    }
}
```

### Kotlin Example

```kotlin
@ExtendWith(MockitoExtension::class)
class UserServiceTest {
    
    @Mock
    private lateinit var userRepository: UserRepository
    
    @Mock
    private lateinit var emailService: EmailService
    
    @InjectMocks
    private lateinit var userService: UserService
    
    @Test
    fun `should create user successfully`() {
        // GIVEN
        val request = CreateUserRequest("test@example.com", "John")
        val user = User("test@example.com", "John")
        
        whenever(userRepository.findByEmail("test@example.com")).thenReturn(Optional.empty())
        whenever(userRepository.save(any())).thenReturn(user)
        
        // WHEN
        val result = userService.createUser(request)
        
        // THEN
        assertNotNull(result)
        assertEquals("John", result.name)
        verify(userRepository).save(any())
        verify(emailService).sendWelcomeEmail("test@example.com")
    }
    
    @Test
    fun `should throw exception when email already exists`() {
        // GIVEN
        val request = CreateUserRequest("test@example.com", "John")
        val existingUser = User("test@example.com", "Jane")
        
        whenever(userRepository.findByEmail("test@example.com"))
            .thenReturn(Optional.of(existingUser))
        
        // WHEN & THEN
        assertThrows<UserAlreadyExistsException> {
            userService.createUser(request)
        }
        
        verify(userRepository, never()).save(any())
        verify(emailService, never()).sendWelcomeEmail(anyString())
    }
    
    @Test
    fun `should get user successfully`() {
        // GIVEN
        val user = User("test@example.com", "John")
        whenever(userRepository.findById(1L)).thenReturn(Optional.of(user))
        
        // WHEN
        val result = userService.getUserById(1L)
        
        // THEN
        assertNotNull(result)
        assertEquals("John", result.name)
        verify(userRepository).findById(1L)
    }
    
    @Test
    fun `should throw exception when user not found`() {
        // GIVEN
        whenever(userRepository.findById(999L)).thenReturn(Optional.empty())
        
        // WHEN & THEN
        val exception = assertThrows<UserNotFoundException> {
            userService.getUserById(999L)
        }
        
        assertTrue(exception.message!!.contains("999"))
    }
}
```

---

## 🎯 Common Test Scenarios

### For Each Service Method, Test:

1. **✅ Happy Path** - Method succeeds with valid input
2. **✅ Validation Failures** - Invalid input throws proper exceptions
3. **✅ Business Rule Violations** - Rules are enforced (e.g., duplicate email)
4. **✅ Not Found Cases** - Resource doesn't exist
5. **✅ Dependency Interactions** - Other services are called correctly
6. **✅ Edge Cases** - Null, empty, boundary values

---

## 💡 Pro Tips

- **Mock all dependencies** - Repository, other services, external APIs
- **Verify interactions** - Use `verify()` to ensure dependencies are called
- **Test exceptions** - Use `assertThrows()` for error cases
- **Use `never()`** - Verify something wasn't called in error cases
- **Keep it fast** - No Spring context needed!
- **Test one thing per test** - Clear, focused tests
