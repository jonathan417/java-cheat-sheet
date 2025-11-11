# Spring Boot Testing Workflow Guide

> **A step-by-step guide for when, what, and how to test your Spring Boot application**

---

## 🎯 The Golden Rule: Test As You Build

**Write tests IMMEDIATELY after writing the code, not at the end!**

---

## 📋 Testing Decision Workflow

### Step 1: What Layer Are You Building?

```
┌─────────────────────────────────────────────────────────┐
│  I just created/modified...                             │
└─────────────────────────────────────────────────────────┘
                          │
         ┌────────────────┼────────────────┬──────────────┐
         │                │                │              │
      Entity         Repository        Service       Controller
         │                │                │              │
         ↓                ↓                ↓              ↓
   [Step 2A]         [Step 2B]        [Step 2C]      [Step 2D]
```

---

## Step 2A: Testing Entities (Domain Models)

### When to Test?
- ✅ If entity has business logic (methods, validation)
- ❌ Skip if it's just a plain data class (getters/setters only)

### What to Test?
- Custom validation logic
- Business rules in entity methods
- Relationships (if complex)

### How to Test?

**Java Example:**
```java
@ExtendWith(MockitoExtension.class)
class UserEntityTest {
    
    @Test
    void shouldValidateEmail() {
        User user = new User();
        user.setEmail("invalid-email");
        
        assertFalse(user.isEmailValid());
    }
    
    @Test
    void shouldCalculateAge() {
        User user = new User();
        user.setBirthDate(LocalDate.of(2000, 1, 1));
        
        assertTrue(user.getAge() >= 24);
    }
}
```

**Kotlin Example:**
```kotlin
@ExtendWith(MockitoExtension::class)
class UserEntityTest {
    
    @Test
    fun `should validate email`() {
        val user = User()
        user.email = "invalid-email"
        
        assertFalse(user.isEmailValid())
    }
    
    @Test
    fun `should calculate age`() {
        val user = User(birthDate = LocalDate.of(2000, 1, 1))
        
        assertTrue(user.age >= 24)
    }
}
```

---

## Step 2B: Testing Repositories

### When to Test?
- ✅ Custom query methods
- ✅ Complex JPQL/native queries
- ❌ Skip simple CRUD (findById, save, etc.) - Spring Data already tests these

### What to Test?
- Custom finder methods work correctly
- Query methods return expected results
- Database constraints are enforced

### How to Test?

**Annotation:** `@DataJpaTest`

**Java Example:**
```java
@DataJpaTest
class UserRepositoryTest {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Test
    void shouldFindUserByEmail() {
        // GIVEN
        User user = new User("test@example.com", "John");
        entityManager.persistAndFlush(user);
        
        // WHEN
        Optional<User> found = userRepository.findByEmail("test@example.com");
        
        // THEN
        assertTrue(found.isPresent());
        assertEquals("John", found.get().getName());
    }
    
    @Test
    void shouldFindActiveUsers() {
        // GIVEN
        entityManager.persist(new User("user1@test.com", true));
        entityManager.persist(new User("user2@test.com", false));
        entityManager.flush();
        
        // WHEN
        List<User> activeUsers = userRepository.findByActiveTrue();
        
        // THEN
        assertEquals(1, activeUsers.size());
    }
}
```

**Kotlin Example:**
```kotlin
@DataJpaTest
class UserRepositoryTest {
    
    @Autowired
    private lateinit var userRepository: UserRepository
    
    @Autowired
    private lateinit var entityManager: TestEntityManager
    
    @Test
    fun `should find user by email`() {
        // GIVEN
        val user = User(email = "test@example.com", name = "John")
        entityManager.persistAndFlush(user)
        
        // WHEN
        val found = userRepository.findByEmail("test@example.com")
        
        // THEN
        assertTrue(found.isPresent)
        assertEquals("John", found.get().name)
    }
    
    @Test
    fun `should find active users`() {
        // GIVEN
        entityManager.persist(User("user1@test.com", active = true))
        entityManager.persist(User("user2@test.com", active = false))
        entityManager.flush()
        
        // WHEN
        val activeUsers = userRepository.findByActiveTrue()
        
        // THEN
        assertEquals(1, activeUsers.size)
    }
}
```

---

## Step 2C: Testing Services (Business Logic)

### When to Test?
- ✅ **ALWAYS!** This is where most of your tests should be
- Services contain your core business logic

### What to Test?
- Business logic correctness
- Error handling
- Different input scenarios (happy path + edge cases)
- Method interactions with dependencies

### How to Test?

**Annotation:** `@ExtendWith(MockitoExtension.class)` - Pure unit test

**Java Example:**
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
        User user = new User("test@example.com", "John");
        when(userRepository.findByEmail(anyString())).thenReturn(Optional.empty());
        when(userRepository.save(any(User.class))).thenReturn(user);
        
        // WHEN
        User created = userService.createUser(user);
        
        // THEN
        assertNotNull(created);
        assertEquals("John", created.getName());
        verify(userRepository).save(user);
        verify(emailService).sendWelcomeEmail(user.getEmail());
    }
    
    @Test
    void shouldCreateUser_ThrowsException_WhenEmailExists() {
        // GIVEN
        User existingUser = new User("test@example.com", "Jane");
        when(userRepository.findByEmail("test@example.com"))
            .thenReturn(Optional.of(existingUser));
        
        // WHEN & THEN
        assertThrows(UserAlreadyExistsException.class, () -> {
            userService.createUser(new User("test@example.com", "John"));
        });
        
        verify(userRepository, never()).save(any());
    }
    
    @Test
    void shouldGetUser_ReturnsUser_WhenExists() {
        // GIVEN
        User user = new User("test@example.com", "John");
        when(userRepository.findById(1L)).thenReturn(Optional.of(user));
        
        // WHEN
        User found = userService.getUserById(1L);
        
        // THEN
        assertNotNull(found);
        assertEquals("John", found.getName());
    }
    
    @Test
    void shouldGetUser_ThrowsException_WhenNotFound() {
        // GIVEN
        when(userRepository.findById(999L)).thenReturn(Optional.empty());
        
        // WHEN & THEN
        assertThrows(UserNotFoundException.class, () -> {
            userService.getUserById(999L);
        });
    }
}
```

**Kotlin Example:**
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
        val user = User(email = "test@example.com", name = "John")
        whenever(userRepository.findByEmail(anyString())).thenReturn(Optional.empty())
        whenever(userRepository.save(any())).thenReturn(user)
        
        // WHEN
        val created = userService.createUser(user)
        
        // THEN
        assertNotNull(created)
        assertEquals("John", created.name)
        verify(userRepository).save(user)
        verify(emailService).sendWelcomeEmail(user.email)
    }
    
    @Test
    fun `should throw exception when email exists`() {
        // GIVEN
        val existingUser = User(email = "test@example.com", name = "Jane")
        whenever(userRepository.findByEmail("test@example.com"))
            .thenReturn(Optional.of(existingUser))
        
        // WHEN & THEN
        assertThrows<UserAlreadyExistsException> {
            userService.createUser(User(email = "test@example.com", name = "John"))
        }
        
        verify(userRepository, never()).save(any())
    }
    
    @Test
    fun `should get user when exists`() {
        // GIVEN
        val user = User(email = "test@example.com", name = "John")
        whenever(userRepository.findById(1L)).thenReturn(Optional.of(user))
        
        // WHEN
        val found = userService.getUserById(1L)
        
        // THEN
        assertNotNull(found)
        assertEquals("John", found.name)
    }
    
    @Test
    fun `should throw exception when user not found`() {
        // GIVEN
        whenever(userRepository.findById(999L)).thenReturn(Optional.empty())
        
        // WHEN & THEN
        assertThrows<UserNotFoundException> {
            userService.getUserById(999L)
        }
    }
}
```

---

## Step 2D: Testing Controllers (REST APIs)

### When to Test?
- ✅ For all REST endpoints
- Test request/response mapping, validation, status codes

### What to Test?
- HTTP status codes (200, 404, 400, etc.)
- Request validation
- Response body structure
- Authentication/Authorization (if applicable)

### How to Test?

**Annotation:** `@WebMvcTest(YourController.class)`

**Java Example:**
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
        User user = new User("test@example.com", "John");
        when(userService.getUserById(1L)).thenReturn(user);
        
        // WHEN & THEN
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
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
        User user = new User("test@example.com", "John");
        when(userService.createUser(any(User.class))).thenReturn(user);
        
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

**Kotlin Example:**
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
        val user = User(email = "test@example.com", name = "John")
        whenever(userService.getUserById(1L)).thenReturn(user)
        
        // WHEN & THEN
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk)
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
        val user = User(email = "test@example.com", name = "John")
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

## 🎯 Complete Testing Checklist

### For Each New Feature/Method:

- [ ] **1. Write the code**
- [ ] **2. Immediately write tests:**
  - [ ] Happy path (success case)
  - [ ] Error cases (exceptions, validation failures)
  - [ ] Edge cases (null, empty, boundary values)
  - [ ] Business rule validation
- [ ] **3. Run tests** - All should pass ✅
- [ ] **4. Refactor** if needed, tests should still pass
- [ ] **5. Commit code + tests together**

---

## 📊 Test Coverage Goals

| Layer | Test Type | Annotation | Coverage Goal |
|-------|-----------|------------|---------------|
| Entity | Unit | `@ExtendWith(MockitoExtension)` | 80%+ (if has logic) |
| Repository | Integration | `@DataJpaTest` | Custom queries only |
| **Service** | **Unit** | **`@ExtendWith(MockitoExtension)`** | **90%+** ⭐ Most important! |
| Controller | Integration | `@WebMvcTest` | 80%+ |
| Full App | Integration | `@SpringBootTest` | 5-10 critical flows |

---

## 🚀 Quick Decision Tree

```
Created new method?
    │
    ├─ In Service? 
    │   → Write @ExtendWith(MockitoExtension) test NOW
    │   → Test: success case + all error cases
    │
    ├─ In Controller?
    │   → Write @WebMvcTest test NOW
    │   → Test: all HTTP status codes
    │
    ├─ In Repository (custom query)?
    │   → Write @DataJpaTest test NOW
    │   → Test: query returns correct data
    │
    └─ In Entity (business logic)?
        → Write @ExtendWith(MockitoExtension) test NOW
        → Test: validation + calculation logic
```

---

## 💡 Pro Tips

### ✅ DO:
- Write tests as you code (not later!)
- Test business logic thoroughly (Services)
- Use descriptive test names: `shouldCreateUser_ThrowsException_WhenEmailExists`
- Test both success and failure scenarios
- Keep tests fast (use mocks, avoid @SpringBootTest)
- Follow GIVEN-WHEN-THEN structure

### ❌ DON'T:
- Don't test framework code (Spring Data basic CRUD)
- Don't test getters/setters (unless they have logic)
- Don't use @SpringBootTest everywhere (slow!)
- Don't skip error case testing
- Don't write tests after finishing all code

---

## 📝 Test Naming Convention

### Java:
```java
// Pattern: should[Action]_[ExpectedResult]_[Condition]
shouldCreateUser_Success_WhenValidData()
shouldCreateUser_ThrowsException_WhenEmailExists()
shouldGetUser_ReturnsNull_WhenNotFound()
```

### Kotlin:
```kotlin
// Pattern: Use backticks for readable names
`should create user successfully when data is valid`()
`should throw exception when email already exists`()
`should return null when user not found`()
```

---

## 🌐 Common Pattern: API Proxy/Gateway Controller

### Scenario: Controller receives request → Calls external API → Maps response → Returns result

This is very common in microservices or when integrating with external APIs.

#### Architecture:
```
Client Request → Controller → Service → External API/Client → Map Response → Return to Client
```

### Mapper Example

Mappers transform external API responses to your domain models.

**Java:**
```java
@Component
public class WeatherMapper {
    
    public WeatherResponse toWeatherResponse(ExternalWeatherResponse external) {
        if (external == null) {
            return null;
        }
        
        return new WeatherResponse(
            external.getCityName(),
            external.getTemp(),
            external.getCondition()
        );
    }
    
    // Optional: Map list of responses
    public List<WeatherResponse> toWeatherResponseList(List<ExternalWeatherResponse> externalList) {
        return externalList.stream()
            .map(this::toWeatherResponse)
            .collect(Collectors.toList());
    }
}
```

**Kotlin:**
```kotlin
@Component
class WeatherMapper {
    
    fun toWeatherResponse(external: ExternalWeatherResponse?): WeatherResponse? {
        return external?.let {
            WeatherResponse(
                city = it.cityName,
                temperature = it.temp,
                condition = it.condition
            )
        }
    }
    
    // Optional: Map list of responses
    fun toWeatherResponseList(externalList: List<ExternalWeatherResponse>): List<WeatherResponse> {
        return externalList.mapNotNull { toWeatherResponse(it) }
    }
}
```

### Mapper Test

**Java:**
```java
@ExtendWith(MockitoExtension.class)
class WeatherMapperTest {
    
    private WeatherMapper weatherMapper;
    
    @BeforeEach
    void setUp() {
        weatherMapper = new WeatherMapper();
    }
    
    @Test
    void shouldMapExternalResponseToWeatherResponse() {
        // GIVEN
        ExternalWeatherResponse external = new ExternalWeatherResponse(
            "Tokyo",
            25.5,
            60,
            "Sunny"
        );
        
        // WHEN
        WeatherResponse result = weatherMapper.toWeatherResponse(external);
        
        // THEN
        assertNotNull(result);
        assertEquals("Tokyo", result.getCity());
        assertEquals(25.5, result.getTemperature());
        assertEquals("Sunny", result.getCondition());
    }
    
    @Test
    void shouldReturnNull_WhenExternalResponseIsNull() {
        // WHEN
        WeatherResponse result = weatherMapper.toWeatherResponse(null);
        
        // THEN
        assertNull(result);
    }
    
    @Test
    void shouldMapListOfExternalResponses() {
        // GIVEN
        List<ExternalWeatherResponse> externalList = Arrays.asList(
            new ExternalWeatherResponse("Tokyo", 25.5, 60, "Sunny"),
            new ExternalWeatherResponse("London", 15.0, 80, "Rainy")
        );
        
        // WHEN
        List<WeatherResponse> results = weatherMapper.toWeatherResponseList(externalList);
        
        // THEN
        assertEquals(2, results.size());
        assertEquals("Tokyo", results.get(0).getCity());
        assertEquals("London", results.get(1).getCity());
    }
}
```

**Kotlin:**
```kotlin
@ExtendWith(MockitoExtension::class)
class WeatherMapperTest {
    
    private lateinit var weatherMapper: WeatherMapper
    
    @BeforeEach
    fun setUp() {
        weatherMapper = WeatherMapper()
    }
    
    @Test
    fun `should map external response to weather response`() {
        // GIVEN
        val external = ExternalWeatherResponse(
            cityName = "Tokyo",
            temp = 25.5,
            humidity = 60,
            condition = "Sunny"
        )
        
        // WHEN
        val result = weatherMapper.toWeatherResponse(external)
        
        // THEN
        assertNotNull(result)
        assertEquals("Tokyo", result?.city)
        assertEquals(25.5, result?.temperature)
        assertEquals("Sunny", result?.condition)
    }
    
    @Test
    fun `should return null when external response is null`() {
        // WHEN
        val result = weatherMapper.toWeatherResponse(null)
        
        // THEN
        assertNull(result)
    }
    
    @Test
    fun `should map list of external responses`() {
        // GIVEN
        val externalList = listOf(
            ExternalWeatherResponse("Tokyo", 25.5, 60, "Sunny"),
            ExternalWeatherResponse("London", 15.0, 80, "Rainy")
        )
        
        // WHEN
        val results = weatherMapper.toWeatherResponseList(externalList)
        
        // THEN
        assertEquals(2, results.size)
        assertEquals("Tokyo", results[0].city)
        assertEquals("London", results[1].city)
    }
}
```

---

### Service Layer Example

**Java:**
```java
@Service
public class WeatherService {
    
    private final RestTemplate restTemplate;
    private final WeatherMapper weatherMapper;
    
    public WeatherService(RestTemplate restTemplate, WeatherMapper weatherMapper) {
        this.restTemplate = restTemplate;
        this.weatherMapper = weatherMapper;
    }
    
    public WeatherResponse getWeather(String city) {
        // Call external API
        String url = "https://api.weather.com/v1/current?city=" + city;
        ExternalWeatherResponse externalResponse = restTemplate.getForObject(
            url, 
            ExternalWeatherResponse.class
        );
        
        if (externalResponse == null) {
            throw new WeatherNotFoundException("Weather data not available");
        }
        
        // Map external response to our domain model
        return weatherMapper.toWeatherResponse(externalResponse);
    }
}
```

**Kotlin:**
```kotlin
@Service
class WeatherService(
    private val restTemplate: RestTemplate,
    private val weatherMapper: WeatherMapper
) {
    
    fun getWeather(city: String): WeatherResponse {
        // Call external API
        val url = "https://api.weather.com/v1/current?city=$city"
        val externalResponse = restTemplate.getForObject(
            url,
            ExternalWeatherResponse::class.java
        ) ?: throw WeatherNotFoundException("Weather data not available")
        
        // Map external response to our domain model
        return weatherMapper.toWeatherResponse(externalResponse)
    }
}
```

### Service Test

**Java:**
```java
@ExtendWith(MockitoExtension.class)
class WeatherServiceTest {
    
    @Mock
    private RestTemplate restTemplate;
    
    @Mock
    private WeatherMapper weatherMapper;
    
    @InjectMocks
    private WeatherService weatherService;
    
    @Test
    void shouldGetWeather_Success_WhenApiReturnsData() {
        // GIVEN
        String city = "Tokyo";
        ExternalWeatherResponse externalResponse = new ExternalWeatherResponse(
            "Tokyo", 25.5, 60, "Sunny"
        );
        WeatherResponse expectedResponse = new WeatherResponse(
            "Tokyo", 25.5, "Sunny"
        );
        
        when(restTemplate.getForObject(anyString(), eq(ExternalWeatherResponse.class)))
            .thenReturn(externalResponse);
        when(weatherMapper.toWeatherResponse(externalResponse))
            .thenReturn(expectedResponse);
        
        // WHEN
        WeatherResponse result = weatherService.getWeather(city);
        
        // THEN
        assertNotNull(result);
        assertEquals("Tokyo", result.getCity());
        assertEquals(25.5, result.getTemperature());
        assertEquals("Sunny", result.getCondition());
        
        verify(restTemplate).getForObject(contains(city), eq(ExternalWeatherResponse.class));
        verify(weatherMapper).toWeatherResponse(externalResponse);
    }
    
    @Test
    void shouldGetWeather_ThrowsException_WhenApiReturnsNull() {
        // GIVEN
        when(restTemplate.getForObject(anyString(), eq(ExternalWeatherResponse.class)))
            .thenReturn(null);
        
        // WHEN & THEN
        assertThrows(WeatherNotFoundException.class, () -> {
            weatherService.getWeather("InvalidCity");
        });
        
        verify(weatherMapper, never()).toWeatherResponse(any());
    }
    
    @Test
    void shouldGetWeather_ThrowsException_WhenApiThrowsException() {
        // GIVEN
        when(restTemplate.getForObject(anyString(), eq(ExternalWeatherResponse.class)))
            .thenThrow(new RestClientException("API unavailable"));
        
        // WHEN & THEN
        assertThrows(RestClientException.class, () -> {
            weatherService.getWeather("Tokyo");
        });
    }
}
```

**Kotlin:**
```kotlin
@ExtendWith(MockitoExtension::class)
class WeatherServiceTest {
    
    @Mock
    private lateinit var restTemplate: RestTemplate
    
    @Mock
    private lateinit var weatherMapper: WeatherMapper
    
    @InjectMocks
    private lateinit var weatherService: WeatherService
    
    @Test
    fun `should get weather successfully when API returns data`() {
        // GIVEN
        val city = "Tokyo"
        val externalResponse = ExternalWeatherResponse(
            city = "Tokyo",
            temp = 25.5,
            humidity = 60,
            condition = "Sunny"
        )
        val expectedResponse = WeatherResponse(
            city = "Tokyo",
            temperature = 25.5,
            condition = "Sunny"
        )
        
        whenever(restTemplate.getForObject(anyString(), eq(ExternalWeatherResponse::class.java)))
            .thenReturn(externalResponse)
        whenever(weatherMapper.toWeatherResponse(externalResponse))
            .thenReturn(expectedResponse)
        
        // WHEN
        val result = weatherService.getWeather(city)
        
        // THEN
        assertNotNull(result)
        assertEquals("Tokyo", result.city)
        assertEquals(25.5, result.temperature)
        assertEquals("Sunny", result.condition)
        
        verify(restTemplate).getForObject(contains(city), eq(ExternalWeatherResponse::class.java))
        verify(weatherMapper).toWeatherResponse(externalResponse)
    }
    
    @Test
    fun `should throw exception when API returns null`() {
        // GIVEN
        whenever(restTemplate.getForObject(anyString(), eq(ExternalWeatherResponse::class.java)))
            .thenReturn(null)
        
        // WHEN & THEN
        assertThrows<WeatherNotFoundException> {
            weatherService.getWeather("InvalidCity")
        }
        
        verify(weatherMapper, never()).toWeatherResponse(any())
    }
    
    @Test
    fun `should throw exception when API throws exception`() {
        // GIVEN
        whenever(restTemplate.getForObject(anyString(), eq(ExternalWeatherResponse::class.java)))
            .thenThrow(RestClientException("API unavailable"))
        
        // WHEN & THEN
        assertThrows<RestClientException> {
            weatherService.getWeather("Tokyo")
        }
    }
}
```

### Controller Layer Example

**Java:**
```java
@RestController
@RequestMapping("/api/weather")
public class WeatherController {
    
    private final WeatherService weatherService;
    
    public WeatherController(WeatherService weatherService) {
        this.weatherService = weatherService;
    }
    
    @GetMapping("/{city}")
    public ResponseEntity<WeatherResponse> getWeather(@PathVariable String city) {
        WeatherResponse weather = weatherService.getWeather(city);
        return ResponseEntity.ok(weather);
    }
}
```

**Kotlin:**
```kotlin
@RestController
@RequestMapping("/api/weather")
class WeatherController(
    private val weatherService: WeatherService
) {
    
    @GetMapping("/{city}")
    fun getWeather(@PathVariable city: String): ResponseEntity<WeatherResponse> {
        val weather = weatherService.getWeather(city)
        return ResponseEntity.ok(weather)
    }
}
```

### Controller Test

**Java:**
```java
@WebMvcTest(WeatherController.class)
class WeatherControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private WeatherService weatherService;
    
    @Test
    void shouldGetWeather_Returns200_WhenCityExists() throws Exception {
        // GIVEN
        WeatherResponse response = new WeatherResponse("Tokyo", 25.5, "Sunny");
        when(weatherService.getWeather("Tokyo")).thenReturn(response);
        
        // WHEN & THEN
        mockMvc.perform(get("/api/weather/Tokyo"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.city").value("Tokyo"))
            .andExpect(jsonPath("$.temperature").value(25.5))
            .andExpect(jsonPath("$.condition").value("Sunny"));
        
        verify(weatherService).getWeather("Tokyo");
    }
    
    @Test
    void shouldGetWeather_Returns404_WhenCityNotFound() throws Exception {
        // GIVEN
        when(weatherService.getWeather("InvalidCity"))
            .thenThrow(new WeatherNotFoundException("Weather data not available"));
        
        // WHEN & THEN
        mockMvc.perform(get("/api/weather/InvalidCity"))
            .andExpect(status().isNotFound());
    }
    
    @Test
    void shouldGetWeather_Returns500_WhenExternalApiDown() throws Exception {
        // GIVEN
        when(weatherService.getWeather("Tokyo"))
            .thenThrow(new RestClientException("API unavailable"));
        
        // WHEN & THEN
        mockMvc.perform(get("/api/weather/Tokyo"))
            .andExpect(status().isInternalServerError());
    }
}
```

**Kotlin:**
```kotlin
@WebMvcTest(WeatherController::class)
class WeatherControllerTest {
    
    @Autowired
    private lateinit var mockMvc: MockMvc
    
    @MockBean
    private lateinit var weatherService: WeatherService
    
    @Test
    fun `should return 200 when city exists`() {
        // GIVEN
        val response = WeatherResponse("Tokyo", 25.5, "Sunny")
        whenever(weatherService.getWeather("Tokyo")).thenReturn(response)
        
        // WHEN & THEN
        mockMvc.perform(get("/api/weather/Tokyo"))
            .andExpect(status().isOk)
            .andExpect(jsonPath("$.city").value("Tokyo"))
            .andExpect(jsonPath("$.temperature").value(25.5))
            .andExpect(jsonPath("$.condition").value("Sunny"))
        
        verify(weatherService).getWeather("Tokyo")
    }
    
    @Test
    fun `should return 404 when city not found`() {
        // GIVEN
        whenever(weatherService.getWeather("InvalidCity"))
            .thenThrow(WeatherNotFoundException("Weather data not available"))
        
        // WHEN & THEN
        mockMvc.perform(get("/api/weather/InvalidCity"))
            .andExpect(status().isNotFound)
    }
    
    @Test
    fun `should return 500 when external API is down`() {
        // GIVEN
        whenever(weatherService.getWeather("Tokyo"))
            .thenThrow(RestClientException("API unavailable"))
        
        // WHEN & THEN
        mockMvc.perform(get("/api/weather/Tokyo"))
            .andExpect(status().isInternalServerError)
    }
}
```

### 🎯 Key Testing Points for API Proxy Pattern:

1. **Mock External Dependencies** (`RestTemplate`, `WebClient`, etc.)
2. **Test Mapping Logic** - Verify data transformation is correct
3. **Test Error Scenarios**:
   - External API returns null
   - External API throws exception
   - Network timeout
   - Invalid response format
4. **Verify Interactions** - Ensure external API is called correctly
5. **Test HTTP Status Codes** - 200, 404, 500, etc.

---

## 🎓 Example: Full Testing Flow for New Feature

### Scenario: Add "Reset Password" feature

#### 1. **Create Entity Method** (if needed)

---

## 🎓 Example: Full Testing Flow for New Feature

### Scenario: Add "Reset Password" feature

#### 1. **Create Entity Method** (if needed)
```java
// User.java
public void setPassword(String password) {
    if (password.length() < 8) {
        throw new IllegalArgumentException("Password too short");
    }
    this.password = hashPassword(password);
}
```
**→ Write Entity Test** ✅

#### 2. **Create Repository Method**
```java
// UserRepository.java
Optional<User> findByResetToken(String token);
```
**→ Write @DataJpaTest** ✅

#### 3. **Create Service Method**
```java
// UserService.java
public void resetPassword(String token, String newPassword) {
    User user = userRepository.findByResetToken(token)
        .orElseThrow(() -> new InvalidTokenException());
    user.setPassword(newPassword);
    userRepository.save(user);
}
```
**→ Write @ExtendWith(MockitoExtension) Test** ✅
- Test: valid token
- Test: invalid token → exception
- Test: password validation

#### 4. **Create Controller Endpoint**
```java
// UserController.java
@PostMapping("/reset-password")
public ResponseEntity<?> resetPassword(@RequestBody ResetPasswordRequest request) {
    userService.resetPassword(request.getToken(), request.getPassword());
    return ResponseEntity.ok().build();
}
```
**→ Write @WebMvcTest** ✅
- Test: 200 OK with valid token
- Test: 400 Bad Request with invalid token
- Test: 400 Bad Request with weak password

---

## 🎯 Summary: The Testing Mindset

**Think of tests as documentation that proves your code works!**

1. **Write code** → **Write test** → **See green ✅** → **Move on**
2. Start with **Service tests** (most important)
3. Add **Controller tests** for APIs
4. Add **Repository tests** for custom queries
5. Keep tests **fast** (mock dependencies)
6. Test **what can break**, not what's trivial

**Remember:** Good tests = confidence to refactor = better code! 🚀
