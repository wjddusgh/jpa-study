### 인덱스를 다룰 때 고려해야할 부분이 아닌것은?
1. 시작과 마지막 인덱스가 같으면 안된다
2. 시작이 마지막보다 크면 안된다
3. 인덱스는 자연수이어야 한다
4. 인덱스가 허용된 것보다 크면 안된다
5. 개수가 실제 항목의 개수와 맞아야 한다

### ZOM, ZOI(Zero, One, Many(Infinity)) 개념으로 파일 시스템을 설명 가능한가요?

### 다음중 테스트가 실패한다면 고쳐야할 곳을 모두 찾으세요 (3개)

```java
public class Human {
  private HumanSize humanSize;  // 필드 접근 가능하다 쳐줍시다
  private String address;
  
  public Boolean isValidHuman() {
    if (humansSize.height == null || humanSize.weight == null) return false;
    
    return (humanSize.height < 300.0 && humanSize.height > 100.0 
    && humanSize.weight > 20.0);
  }
  
  public String liveIn() {
      if (address == null) return "City";
      return address;
  }
}

public class HumanTest {
  private Human human;
  
  @After
  public void isValidHuman() {
    assertEquals(human.isValid(), true);
  }
  
  @Test
  public void putInNormalHuman() {
    human = new Human(new HumanSize(180.0, 90.0), "Uijeongbu");
    assertEquals(human.getHeight(), 180);
  }
  
  @Test
  public void putInSuperTallHuman() {
    human = new Human(new HumanSize(380.0, 125.0), "Pusan");
    assertEquals(human.getWeight(), 125.0);
    assertEquals(human.isValid(), false);
  }
  
  @Test
  public void putInSuperLighttHuman() {
    human = new Human(new HumanSize(150.0, 30.0), "");
    assertEquals(human.isValid(), true);
    assertEquals(human.liveIn(), "City"); 
  }
  
  @Test
  public void 

```



### ans
1. 3번 0도됨
2. 파일시스템
  - 0 -> 최상위 디렉토리를 포함하는 디렉토리는 없다
  - 1 -> 각 하위 디렉토리는 정확히 하나의 상위 디렉토리를 가진다
  - Many(Infinity) -> 각 디렉토리는 파일 시스템의 규칙에 따라 최상위 디렉토리 또는 하위 디렉토리에 관계없이 파일 또는 하위 디렉토리를 포함할 수 있다
    
4. @After 의 메소드가 putInSuperTallHuman()을 실패하게 만드므로 필요한 곳에 넣어준다
5. putInRightHuman()의 단언이 Int형으로 키를 넣고있다
6. liveIn에서의 null과 ""을 구분하자


