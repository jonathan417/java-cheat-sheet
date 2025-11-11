# Testing Mappers

> **Test data transformation logic - simple and fast!**

---

## 🎯 When to Test?

✅ **Always test mappers** - They transform data between layers/external APIs

## 📝 What to Test?

- Field mapping correctness
- Null handling
- Data transformation logic
- List/collection mapping
- Default values

---

## 🔧 Setup

**Annotation:** `@ExtendWith(MockitoExtension.class)` or no annotation

**No mocks needed!** - Mappers are pure functions

---

## 📖 Example: Weather Mapper

### Mapper Implementation

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
    
    fun toWeatherResponseList(externalList: List<ExternalWeatherResponse>): List<WeatherResponse> {
        return externalList.mapNotNull { toWeatherResponse(it) }
    }
}
```

---

## 🧪 Mapper Tests

### Java Example

```java
class WeatherMapperTest {
    
    private WeatherMapper mapper;
    
    @BeforeEach
    void setUp() {
        mapper = new WeatherMapper();
    }
    
    @Test
    void shouldMapExternalResponseToWeatherResponse() {
        // GIVEN
        ExternalWeatherResponse external = new ExternalWeatherResponse(
            "Tokyo", 25.5, 60, "Sunny"
        );
        
        // WHEN
        WeatherResponse result = mapper.toWeatherResponse(external);
        
        // THEN
        assertNotNull(result);
        assertEquals("Tokyo", result.getCity());
        assertEquals(25.5, result.getTemperature());
        assertEquals("Sunny", result.getCondition());
    }
    
    @Test
    void shouldReturnNull_WhenInputIsNull() {
        // WHEN
        WeatherResponse result = mapper.toWeatherResponse(null);
        
        // THEN
        assertNull(result);
    }
    
    @Test
    void shouldMapList() {
        // GIVEN
        List<ExternalWeatherResponse> externalList = Arrays.asList(
            new ExternalWeatherResponse("Tokyo", 25.5, 60, "Sunny"),
            new ExternalWeatherResponse("London", 15.0, 80, "Rainy")
        );
        
        // WHEN
        List<WeatherResponse> results = mapper.toWeatherResponseList(externalList);
        
        // THEN
        assertEquals(2, results.size());
        assertEquals("Tokyo", results.get(0).getCity());
        assertEquals("London", results.get(1).getCity());
    }
    
    @Test
    void shouldHandleEmptyList() {
        // WHEN
        List<WeatherResponse> results = mapper.toWeatherResponseList(Collections.emptyList());
        
        // THEN
        assertTrue(results.isEmpty());
    }
}
```

### Kotlin Example

```kotlin
class WeatherMapperTest {
    
    private lateinit var mapper: WeatherMapper
    
    @BeforeEach
    fun setUp() {
        mapper = WeatherMapper()
    }
    
    @Test
    fun `should map external response to weather response`() {
        // GIVEN
        val external = ExternalWeatherResponse("Tokyo", 25.5, 60, "Sunny")
        
        // WHEN
        val result = mapper.toWeatherResponse(external)
        
        // THEN
        assertNotNull(result)
        assertEquals("Tokyo", result?.city)
        assertEquals(25.5, result?.temperature)
        assertEquals("Sunny", result?.condition)
    }
    
    @Test
    fun `should return null when input is null`() {
        // WHEN
        val result = mapper.toWeatherResponse(null)
        
        // THEN
        assertNull(result)
    }
    
    @Test
    fun `should map list of responses`() {
        // GIVEN
        val externalList = listOf(
            ExternalWeatherResponse("Tokyo", 25.5, 60, "Sunny"),
            ExternalWeatherResponse("London", 15.0, 80, "Rainy")
        )
        
        // WHEN
        val results = mapper.toWeatherResponseList(externalList)
        
        // THEN
        assertEquals(2, results.size)
        assertEquals("Tokyo", results[0].city)
        assertEquals("London", results[1].city)
    }
    
    @Test
    fun `should handle empty list`() {
        // WHEN
        val results = mapper.toWeatherResponseList(emptyList())
        
        // THEN
        assertTrue(results.isEmpty())
    }
}
```

---

## 🎯 Common Test Scenarios

### For Mappers, Test:

1. **✅ Correct Field Mapping** - All fields map correctly
2. **✅ Null Input Handling** - Returns null or empty safely
3. **✅ List Mapping** - Multiple items transform correctly
4. **✅ Empty Collection** - Handles empty lists/sets
5. **✅ Data Transformation** - Calculations or formatting work
6. **✅ Default Values** - Missing fields get proper defaults

---

## 💡 Pro Tips

- **No mocks needed** - Just instantiate and test
- **Test all fields** - Don't miss any mapping
- **Test null safety** - Important for external APIs
- **Keep it simple** - Mappers should be pure functions
- **Fast tests** - No Spring, no DB, just logic
- Use `@BeforeEach` to create fresh mapper instance
