# Testing Configuration Classes

> **Usually skip testing configs, but here's how when needed**

---

## 🎯 When to Test?

❌ **Usually skip** - Most configs don't need testing

✅ **Test only when:**
- Complex bean creation logic
- Conditional configuration (@ConditionalOnProperty)
- Custom auto-configuration
- Bean initialization with complex logic

---

## 📝 What to Test?

- Bean creation works correctly
- Conditional beans load properly
- Configuration properties bind correctly
- Dependencies are wired properly

---

## 🔧 Setup

**Annotation:** `@SpringBootTest` or `@ContextConfiguration`

---

## 📖 Example: Simple Config (Usually Skip)

### Config That Doesn't Need Testing

**Java:**
```java
@Configuration
public class AppConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper()
            .registerModule(new JavaTimeModule());
    }
}
```

**❌ Don't test this** - It's too simple, Spring handles it

---

## 📖 Example: Complex Config (Test This)

### Config with Complex Logic

**Java:**
```java
@Configuration
public class WeatherClientConfig {
    
    @Value("${weather.api.url}")
    private String apiUrl;
    
    @Value("${weather.api.timeout:5000}")
    private int timeout;
    
    @Bean
    public RestTemplate weatherRestTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        
        // Complex timeout configuration
        HttpComponentsClientHttpRequestFactory factory = 
            new HttpComponentsClientHttpRequestFactory();
        factory.setConnectTimeout(timeout);
        factory.setReadTimeout(timeout);
        
        restTemplate.setRequestFactory(factory);
        return restTemplate;
    }
    
    @Bean
    @ConditionalOnProperty(name = "weather.cache.enabled", havingValue = "true")
    public CacheManager weatherCacheManager() {
        return new ConcurrentMapCacheManager("weather");
    }
}
```

**Kotlin:**
```kotlin
@Configuration
class WeatherClientConfig {
    
    @Value("\${weather.api.url}")
    private lateinit var apiUrl: String
    
    @Value("\${weather.api.timeout:5000}")
    private var timeout: Int = 5000
    
    @Bean
    fun weatherRestTemplate(): RestTemplate {
        val restTemplate = RestTemplate()
        
        // Complex timeout configuration
        val factory = HttpComponentsClientHttpRequestFactory()
        factory.setConnectTimeout(timeout)
        factory.setReadTimeout(timeout)
        
        restTemplate.requestFactory = factory
        return restTemplate
    }
    
    @Bean
    @ConditionalOnProperty(name = "weather.cache.enabled", havingValue = "true")
    fun weatherCacheManager(): CacheManager {
        return ConcurrentMapCacheManager("weather")
    }
}
```

---

## 🧪 Config Tests

### Java Example

```java
@SpringBootTest
@TestPropertySource(properties = {
    "weather.api.url=https://api.weather.com",
    "weather.api.timeout=3000",
    "weather.cache.enabled=true"
})
class WeatherClientConfigTest {
    
    @Autowired
    private RestTemplate weatherRestTemplate;
    
    @Autowired(required = false)
    private CacheManager weatherCacheManager;
    
    @Test
    void shouldCreateRestTemplateWithCorrectTimeout() {
        // THEN
        assertNotNull(weatherRestTemplate);
        HttpComponentsClientHttpRequestFactory factory = 
            (HttpComponentsClientHttpRequestFactory) weatherRestTemplate.getRequestFactory();
        
        // Verify timeout is set (you might need reflection to check this)
        assertNotNull(factory);
    }
    
    @Test
    void shouldCreateCacheManager_WhenEnabled() {
        // THEN
        assertNotNull(weatherCacheManager);
        assertNotNull(weatherCacheManager.getCache("weather"));
    }
}

@SpringBootTest
@TestPropertySource(properties = {
    "weather.api.url=https://api.weather.com",
    "weather.cache.enabled=false"
})
class WeatherClientConfigWithoutCacheTest {
    
    @Autowired(required = false)
    private CacheManager weatherCacheManager;
    
    @Test
    void shouldNotCreateCacheManager_WhenDisabled() {
        // THEN
        assertNull(weatherCacheManager);
    }
}
```

### Kotlin Example

```kotlin
@SpringBootTest
@TestPropertySource(properties = [
    "weather.api.url=https://api.weather.com",
    "weather.api.timeout=3000",
    "weather.cache.enabled=true"
])
class WeatherClientConfigTest {
    
    @Autowired
    private lateinit var weatherRestTemplate: RestTemplate
    
    @Autowired(required = false)
    private var weatherCacheManager: CacheManager? = null
    
    @Test
    fun `should create rest template with correct timeout`() {
        // THEN
        assertNotNull(weatherRestTemplate)
        val factory = weatherRestTemplate.requestFactory as HttpComponentsClientHttpRequestFactory
        
        assertNotNull(factory)
    }
    
    @Test
    fun `should create cache manager when enabled`() {
        // THEN
        assertNotNull(weatherCacheManager)
        assertNotNull(weatherCacheManager?.getCache("weather"))
    }
}

@SpringBootTest
@TestPropertySource(properties = [
    "weather.api.url=https://api.weather.com",
    "weather.cache.enabled=false"
])
class WeatherClientConfigWithoutCacheTest {
    
    @Autowired(required = false)
    private var weatherCacheManager: CacheManager? = null
    
    @Test
    fun `should not create cache manager when disabled`() {
        // THEN
        assertNull(weatherCacheManager)
    }
}
```

---

## 🎯 What to Test in Configs

### Test These:

1. **✅ Conditional Beans** - @ConditionalOnProperty logic works
2. **✅ Complex Bean Creation** - Custom initialization logic
3. **✅ Property Binding** - @ConfigurationProperties work correctly
4. **✅ Bean Dependencies** - Beans get correct dependencies injected

### Skip These:

1. **❌ Simple @Bean methods** - Just returns `new SomeClass()`
2. **❌ Standard Spring configs** - Spring tests these
3. **❌ Property files** - No logic to test

---

## 💡 Pro Tips

- **Most configs don't need tests** - Keep it simple!
- Use `@TestPropertySource` to test with different properties
- Use `@Autowired(required = false)` for optional beans
- Test conditional beans with multiple test classes
- Focus on **custom logic**, not Spring framework behavior
- If config is complex, maybe refactor it!

---

## 🎓 Rule of Thumb

**If your config has an `if` statement or complex logic → Test it**

**If your config just creates beans → Skip testing**
