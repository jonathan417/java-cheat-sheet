# Testing Decision Guide

> **Quick reference: Which test should I write?**

---

## 🎯 The Golden Rule

**Write tests IMMEDIATELY after writing the code, not at the end!**

---

## 🚀 Quick Decision Tree

```
Created new code?
    │
    ├─ Service/Business Logic? 
    │   → @ExtendWith(MockitoExtension) ⚡ FASTEST
    │   → 90% coverage goal (MOST IMPORTANT!)
    │
    ├─ Controller/REST API?
    │   → @WebMvcTest ⚡ FAST
    │   → Test: HTTP status codes + validation
    │
    ├─ Repository (custom query)?
    │   → @DataJpaTest ⚡ FAST
    │   → Skip simple CRUD (Spring tests these)
    │
    ├─ Mapper/Util?
    │   → @ExtendWith(MockitoExtension) ⚡ FASTEST
    │   → No mocks needed (pure functions)
    │
    └─ Full Integration?
        → @SpringBootTest 🐢 SLOW
        → Use sparingly (5-10 critical flows only)
```

---

## 📊 Test Type Quick Reference

| What I'm Testing | Annotation | Speed | Coverage Goal |
|------------------|------------|-------|---------------|
| **Service** (Business Logic) | `@ExtendWith(MockitoExtension)` | ⚡ Milliseconds | **90%+** ⭐ |
| **Controller** (REST API) | `@WebMvcTest` | ⚡ 1-2 sec | 80%+ |
| **Repository** (Custom Query) | `@DataJpaTest` | ⚡ 1-2 sec | Custom queries only |
| **Mapper/Util** | `@ExtendWith(MockitoExtension)` | ⚡ Milliseconds | 80%+ |
| **Config** | Usually skip | - | - |
| **Full Integration** | `@SpringBootTest` | 🐢 5-10 sec | 5-10 critical flows |

---

## 🎯 Test Distribution (Healthy Test Suite)

```
📊 Recommended Mix:
├─ 80% Unit Tests (@ExtendWith)     ⚡⚡⚡
├─ 15% Slice Tests (@WebMvcTest, @DataJpaTest)  ⚡⚡
└─  5% Integration Tests (@SpringBootTest)  🐢
```

---

## ✅ Testing Checklist (For Each New Feature)

- [ ] **1. Write the code**
- [ ] **2. Write tests immediately:**
  - [ ] Happy path (success case)
  - [ ] Error cases (exceptions, validation)
  - [ ] Edge cases (null, empty, boundaries)
- [ ] **3. Run tests** - All pass ✅
- [ ] **4. Refactor** - Tests still pass
- [ ] **5. Commit** - Code + tests together

---

## 💡 Quick Rules

### ✅ DO:
- Start with `@ExtendWith(MockitoExtension)` by default
- Test Services thoroughly (your business logic)
- Test both success AND failure scenarios
- Keep tests fast (mock dependencies)
- Use descriptive names: `shouldCreateUser_ThrowsException_WhenEmailExists`

### ❌ DON'T:
- Don't use `@SpringBootTest` everywhere (too slow!)
- Don't test framework code (Spring Data basic CRUD)
- Don't test getters/setters (unless they have logic)
- Don't skip error case testing
- Don't write tests after finishing all code

---

## 🎓 When to Skip Tests

❌ **Skip testing these:**
- Simple getters/setters (no logic)
- Basic CRUD methods from Spring Data (findById, save)
- Configuration classes (unless complex logic)
- DTOs/Entities without business logic

✅ **Always test these:**
- Business logic in Services
- Custom repository queries
- REST API endpoints
- Mappers with transformation logic
- Error handling
- Validation logic
