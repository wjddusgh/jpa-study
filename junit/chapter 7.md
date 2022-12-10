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

```



### ans
1. @After 의 메소드가 putInSuperTallHuman()을 실패하게 만드므로 필요한 곳에 넣어준다
2. putInRightHuman()의 단언이 Int형으로 키를 넣고있다
3. liveIn에서의 null과 ""을 구분하자


