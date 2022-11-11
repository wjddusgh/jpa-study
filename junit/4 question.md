# 1번
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


    
  
