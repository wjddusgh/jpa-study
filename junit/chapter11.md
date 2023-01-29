1. 프로덕션 코드에서는 유용하지만, JUNIT의 도움을 받는 테스트 코드에서는 필요없는 것은 무엇이 있을까요?
 <details>
<summary> 정답 </summary>
  1. try-catch 블록 </br>
  2. not-null 단언 </br>
</details>
 
2. 다음 테스트는 어떤 냄새가 나나요?
```java
...

@Test
public void bookPushHelperPushTest() thorws IOException {
  Book book1 = new Book("101", "성진의 스프링", "어디출판사");  // 훌륭한 책 객체
  Book book2 = new Book(null, "연수의 스프링", "출판성공기원");   // 책 id 없음
  Book book3 = new Book("103", "연호의 스프링", "이상한출판사");    // 책 리스트
  ArrayList<Book> books = new ArrayList<>(book1, book3); 
  String result1 = bookPushHelper.push(book1);
  String result2 = bookPushHelper.push(book2);
  String result3 = bookPushHelper.push(book3);
  assertEquals(result1, "어디출판사");  // 성공시 출판사 이름 리턴
  assertNotEquals(result2, "출판성공기원");
  assertEquals(result3, List.Of("어디출판사", "이상한출판사")); // 거지같은 리턴값
}
```
 
 <details>
<summary> 정답 </summary>
1. AAA, 혹은 given-when-then 에 맞게 빈 줄을 넣어서 구분해주자 </br>
2. 출판성공기원 이름은 너무 어지럽다. 출판실패 객체 처럼 암시 시켜주자 </br>
3. 한번에 세개의 단언이 있다. 한 테스트당 한 단언으로 할 수 있게끔 테스트를 쪼개주자
</details>

3. 진심 낼게없네요 무엇을 하면 좋을까요
<details>
<summary> 정답 </summary>
String.getBytes() 첨봤습니다. </br>
@Before, @After 는 static만 되는 이유 알아보기
</details>
