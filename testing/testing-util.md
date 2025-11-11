# Testing Utility Classes

> **Test helper methods and common utilities**

---

## 🎯 When to Test?

✅ **Always test utility methods** - They're reused everywhere

## 📝 What to Test?

- String manipulation
- Date/time utilities
- Validation helpers
- Formatting functions
- Calculation methods
- Collection utilities

---

## 🔧 Setup

**No special annotation needed** - Just instantiate and test

---

## 📖 Example: String Utilities

### Util Implementation

**Java:**
```java
public class StringUtils {
    
    public static boolean isValidEmail(String email) {
        if (email == null || email.isEmpty()) {
            return false;
        }
        return email.matches("^[A-Za-z0-9+_.-]+@(.+)$");
    }
    
    public static String maskEmail(String email) {
        if (!isValidEmail(email)) {
            return email;
        }
        
        String[] parts = email.split("@");
        String username = parts[0];
        String masked = username.charAt(0) + "***" + username.charAt(username.length() - 1);
        return masked + "@" + parts[1];
    }
    
    public static String capitalize(String str) {
        if (str == null || str.isEmpty()) {
            return str;
        }
        return str.substring(0, 1).toUpperCase() + str.substring(1).toLowerCase();
    }
}
```

**Kotlin:**
```kotlin
object StringUtils {
    
    fun isValidEmail(email: String?): Boolean {
        if (email.isNullOrEmpty()) {
            return false
        }
        return email.matches(Regex("^[A-Za-z0-9+_.-]+@(.+)$"))
    }
    
    fun maskEmail(email: String): String {
        if (!isValidEmail(email)) {
            return email
        }
        
        val parts = email.split("@")
        val username = parts[0]
        val masked = "${username.first()}***${username.last()}"
        return "$masked@${parts[1]}"
    }
    
    fun capitalize(str: String?): String? {
        if (str.isNullOrEmpty()) {
            return str
        }
        return str.replaceFirstChar { it.uppercase() } + str.substring(1).lowercase()
    }
}
```

---

## 🧪 Util Tests

### Java Example

```java
class StringUtilsTest {
    
    @Test
    void shouldValidateEmail_ReturnsTrue_WhenValid() {
        assertTrue(StringUtils.isValidEmail("test@example.com"));
        assertTrue(StringUtils.isValidEmail("user.name+tag@domain.co.uk"));
    }
    
    @Test
    void shouldValidateEmail_ReturnsFalse_WhenInvalid() {
        assertFalse(StringUtils.isValidEmail("invalid-email"));
        assertFalse(StringUtils.isValidEmail("@example.com"));
        assertFalse(StringUtils.isValidEmail("test@"));
        assertFalse(StringUtils.isValidEmail(null));
        assertFalse(StringUtils.isValidEmail(""));
    }
    
    @Test
    void shouldMaskEmail() {
        assertEquals("t***t@example.com", StringUtils.maskEmail("test@example.com"));
        assertEquals("j***n@domain.co", StringUtils.maskEmail("john@domain.co"));
    }
    
    @Test
    void shouldNotMaskInvalidEmail() {
        assertEquals("invalid-email", StringUtils.maskEmail("invalid-email"));
    }
    
    @Test
    void shouldCapitalize() {
        assertEquals("Hello", StringUtils.capitalize("hello"));
        assertEquals("Hello", StringUtils.capitalize("HELLO"));
        assertEquals("H", StringUtils.capitalize("h"));
    }
    
    @Test
    void shouldHandleNullOrEmpty_InCapitalize() {
        assertNull(StringUtils.capitalize(null));
        assertEquals("", StringUtils.capitalize(""));
    }
}
```

### Kotlin Example

```kotlin
class StringUtilsTest {
    
    @Test
    fun `should validate email returns true when valid`() {
        assertTrue(StringUtils.isValidEmail("test@example.com"))
        assertTrue(StringUtils.isValidEmail("user.name+tag@domain.co.uk"))
    }
    
    @Test
    fun `should validate email returns false when invalid`() {
        assertFalse(StringUtils.isValidEmail("invalid-email"))
        assertFalse(StringUtils.isValidEmail("@example.com"))
        assertFalse(StringUtils.isValidEmail("test@"))
        assertFalse(StringUtils.isValidEmail(null))
        assertFalse(StringUtils.isValidEmail(""))
    }
    
    @Test
    fun `should mask email`() {
        assertEquals("t***t@example.com", StringUtils.maskEmail("test@example.com"))
        assertEquals("j***n@domain.co", StringUtils.maskEmail("john@domain.co"))
    }
    
    @Test
    fun `should not mask invalid email`() {
        assertEquals("invalid-email", StringUtils.maskEmail("invalid-email"))
    }
    
    @Test
    fun `should capitalize`() {
        assertEquals("Hello", StringUtils.capitalize("hello"))
        assertEquals("Hello", StringUtils.capitalize("HELLO"))
        assertEquals("H", StringUtils.capitalize("h"))
    }
    
    @Test
    fun `should handle null or empty in capitalize`() {
        assertNull(StringUtils.capitalize(null))
        assertEquals("", StringUtils.capitalize(""))
    }
}
```

---

## 🎯 Common Test Scenarios

### For Utility Methods, Test:

1. **✅ Happy Path** - Method works with valid input
2. **✅ Edge Cases** - Empty, null, single character, etc.
3. **✅ Boundary Values** - Min/max values
4. **✅ Invalid Input** - How method handles bad data
5. **✅ Multiple Formats** - Different valid input formats
6. **✅ Special Characters** - Unicode, symbols, etc.

---

## 📖 Example: Date Utilities

**Java:**
```java
class DateUtilsTest {
    
    @Test
    void shouldFormatDate() {
        LocalDate date = LocalDate.of(2025, 11, 12);
        assertEquals("2025-11-12", DateUtils.formatDate(date));
    }
    
    @Test
    void shouldCalculateDaysBetween() {
        LocalDate start = LocalDate.of(2025, 11, 1);
        LocalDate end = LocalDate.of(2025, 11, 12);
        assertEquals(11, DateUtils.daysBetween(start, end));
    }
    
    @Test
    void shouldCheckIsWeekend() {
        assertTrue(DateUtils.isWeekend(LocalDate.of(2025, 11, 9))); // Saturday
        assertFalse(DateUtils.isWeekend(LocalDate.of(2025, 11, 10))); // Monday
    }
}
```

**Kotlin:**
```kotlin
class DateUtilsTest {
    
    @Test
    fun `should format date`() {
        val date = LocalDate.of(2025, 11, 12)
        assertEquals("2025-11-12", DateUtils.formatDate(date))
    }
    
    @Test
    fun `should calculate days between`() {
        val start = LocalDate.of(2025, 11, 1)
        val end = LocalDate.of(2025, 11, 12)
        assertEquals(11, DateUtils.daysBetween(start, end))
    }
    
    @Test
    fun `should check is weekend`() {
        assertTrue(DateUtils.isWeekend(LocalDate.of(2025, 11, 9))) // Saturday
        assertFalse(DateUtils.isWeekend(LocalDate.of(2025, 11, 10))) // Monday
    }
}
```

---

## 💡 Pro Tips

- **Test thoroughly** - Utils are used everywhere, bugs multiply!
- **No Spring needed** - Pure unit tests, super fast
- **Test edge cases** - Null, empty, boundaries
- **Use descriptive names** - Clear what's being tested
- **Group related tests** - Use nested test classes if needed
- **Static methods** - No instance needed, just call directly
