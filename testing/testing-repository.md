# Testing Repositories

> **Test custom queries and complex database operations**

---

## 🎯 When to Test?

✅ **Only test custom queries** - Skip simple CRUD (Spring Data tests those)

## 📝 What to Test?

- Custom finder methods
- Complex JPQL/native queries
- Query methods with multiple parameters
- @Query annotations
- Database constraints

❌ **Skip testing:**
- Simple methods like `findById()`, `save()`, `delete()` (Spring Data handles these)

---

## 🔧 Setup

**Annotation:** `@DataJpaTest`

**What it does:**
- Configures in-memory database (H2)
- Provides `TestEntityManager`
- Rolls back after each test

---

## 📖 Example: User Repository

### Repository Interface

**Java:**
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    Optional<User> findByEmail(String email);
    
    List<User> findByActiveTrue();
    
    @Query("SELECT u FROM User u WHERE u.name LIKE %:keyword%")
    List<User> searchByName(@Param("keyword") String keyword);
}
```

**Kotlin:**
```kotlin
@Repository
interface UserRepository : JpaRepository<User, Long> {
    
    fun findByEmail(email: String): Optional<User>
    
    fun findByActiveTrue(): List<User>
    
    @Query("SELECT u FROM User u WHERE u.name LIKE %:keyword%")
    fun searchByName(@Param("keyword") keyword: String): List<User>
}
```

---

## 🧪 Repository Tests

### Java Example

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
    void shouldReturnEmpty_WhenEmailNotFound() {
        // WHEN
        Optional<User> found = userRepository.findByEmail("nonexistent@example.com");
        
        // THEN
        assertFalse(found.isPresent());
    }
    
    @Test
    void shouldFindOnlyActiveUsers() {
        // GIVEN
        entityManager.persist(new User("active@test.com", "John", true));
        entityManager.persist(new User("inactive@test.com", "Jane", false));
        entityManager.flush();
        
        // WHEN
        List<User> activeUsers = userRepository.findByActiveTrue();
        
        // THEN
        assertEquals(1, activeUsers.size());
        assertEquals("John", activeUsers.get(0).getName());
    }
    
    @Test
    void shouldSearchByNameKeyword() {
        // GIVEN
        entityManager.persist(new User("user1@test.com", "John Doe"));
        entityManager.persist(new User("user2@test.com", "John Smith"));
        entityManager.persist(new User("user3@test.com", "Jane Doe"));
        entityManager.flush();
        
        // WHEN
        List<User> results = userRepository.searchByName("John");
        
        // THEN
        assertEquals(2, results.size());
        assertTrue(results.stream().allMatch(u -> u.getName().contains("John")));
    }
}
```

### Kotlin Example

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
        val user = User("test@example.com", "John")
        entityManager.persistAndFlush(user)
        
        // WHEN
        val found = userRepository.findByEmail("test@example.com")
        
        // THEN
        assertTrue(found.isPresent)
        assertEquals("John", found.get().name)
    }
    
    @Test
    fun `should return empty when email not found`() {
        // WHEN
        val found = userRepository.findByEmail("nonexistent@example.com")
        
        // THEN
        assertFalse(found.isPresent)
    }
    
    @Test
    fun `should find only active users`() {
        // GIVEN
        entityManager.persist(User("active@test.com", "John", active = true))
        entityManager.persist(User("inactive@test.com", "Jane", active = false))
        entityManager.flush()
        
        // WHEN
        val activeUsers = userRepository.findByActiveTrue()
        
        // THEN
        assertEquals(1, activeUsers.size)
        assertEquals("John", activeUsers[0].name)
    }
    
    @Test
    fun `should search by name keyword`() {
        // GIVEN
        entityManager.persist(User("user1@test.com", "John Doe"))
        entityManager.persist(User("user2@test.com", "John Smith"))
        entityManager.persist(User("user3@test.com", "Jane Doe"))
        entityManager.flush()
        
        // WHEN
        val results = userRepository.searchByName("John")
        
        // THEN
        assertEquals(2, results.size)
        assertTrue(results.all { it.name.contains("John") })
    }
}
```

---

## 🎯 Common Test Scenarios

### For Custom Query Methods, Test:

1. **✅ Found Case** - Query returns expected data
2. **✅ Not Found Case** - Query returns empty
3. **✅ Multiple Results** - Query returns correct list
4. **✅ Filtering Logic** - WHERE clauses work correctly
5. **✅ Sorting** - ORDER BY works as expected
6. **✅ Parameters** - Query handles parameters correctly

---

## 💡 Pro Tips

- Use `TestEntityManager` to set up test data
- Use `persistAndFlush()` to ensure data is in DB
- Test queries return correct data, not just "something"
- Test edge cases (empty results, multiple matches)
- Don't test Spring Data generated methods
- Keep test data simple and focused
