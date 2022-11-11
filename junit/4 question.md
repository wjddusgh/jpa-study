# 1번
AAA중에서 이미 필요한 상태이기 때문에 생략 가능한 부분은?

# 2번
테스트를 별도 디렉터리로 분리하지만 프로덕션 코드와 같은 패키지에 넣는다면 생기는 문제점은?

# 3번
```kotlin
data class Data(id: Long, feel: String)

class StudyTests {
  
    @Autowired private val repository : Repository;
  
    @BeforeClass  //junit5 는 @BeforeAll
    fun createData() {
      (0..4).forEach { repository.save(Data(it, "good")) }
    }
    
    @Before //junit5 는 @BeforeEach
    fun deleteData() {
      repository.deleteAll()
    }
    
    @Test
    fun `find id 4's feel`() {
      
      val result = repository.findById(4).feel
      
      assertEquals(result, "good")
    }
    
```
- 실패할까 성공할까!!!

# 4번
given-when-then 은 무슨 개발에서 말한다고 하나요? 풀네임도 맞추세요
1. ADD
2. BDD
3. DDD
4. TDD

# 5번
실패한 테스트를 나중에 고칠려고 넘어가고 싶어요, 어떤 어노테이션 사용할까요?
1. @Avoid
2. @Pass
3. @Forget
4. @Ignore

