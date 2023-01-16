1. 람다 와 익명내부클래스 의 차이점은 무엇인가요??

 <details>
<summary> 정답 </summary>
  둘 다 용도는 같지만, 람다는 주어진 식에 맞는 메소드를 만들어내고, 익명내부클래스는 주어진 식에 맞는 새로운 클래스를 만들어낸다.
  그래서 둘의 변수 스코프는 다르다.
</details>
 
2. stub 을 주입하는 방법을 3가지 이상 말해주세요
 
 <details>
<summary> 정답 </summary>
생성자 주입, 세터 메소드, 팩토리 메소드 오버라이딩, 구글 주스나 스프링 주입 도구, 추상 팩토리
</details>

3. stub 을 mock 으로 변환하기 위해서 필요한 기능들은?
 <details>
<summary> 정답 </summary>
1. 요구하는 인자를 stub 내부가 아닌 테스트 자체에 명시하기 </br>
2. 메소드의 인자들을 저장하기 </br>
3. 저장한 인자들이 기대한 인자들인지 검증하는 능력 가지기 
</details>

4. 이렇게 복잡하고 많은 기능을 가진 mock을 만들면 stub 보다 어떤 장점이 있나요?
 <details>
<summary> 정답 </summary>
하나의 테스트로는 낭비로 보일수 있지만, 여러 테스트에 같은 Mock 객체를 쓰게된다면 코드 중복이 줄고, </br>
  다른 번거로운 의존성들에 대해 더 많은 목을 구현한다면 이들 사이의 중복을 제거하는 방법을 찾을수도 있다
</details>

5.Mockito 의 의존 주입 절차를 아시나요?
 <details>
<summary> 정답 </summary>
1. @Mock 어노테이션을 사용하여 목 인스턴스 생성 </br>
2. @InjectMocks 어노테이션을 붙인 대상 인스턴스 변수 선언 </br>
3. 대상 인스턴스를 인스턴스화(new) 한후에 MockitoAnnotations.initMocks(this) 호출 
</details>
