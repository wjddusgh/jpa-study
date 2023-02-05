1. TDD에서 가장 우선적으로 만드는것은 테스트인데 이거의 장점은?

 <details>
<summary> 정답 </summary>
  테스트를 통과하기 위해 구현한 코드는 만든 즉시 검증, 리팩토링, 정리가 가능해진다
</details>
 
```java
~~~
Boolean result = Car.isSuvModel();

assertTrue(result);
---

assertTrue(Car.isSuvModel());
~~~
```
2. 두 방식은 무슨 차이점이 있죠? 
 <details>
<summary> 정답 </summary>
첫번째는 실행과 단언을 분리하여 AAA원칙 지킨다. 하지만 AAA는 만고불변의 진리가 아니기에 쓸때없는 임시변수를 줄이는 두번째 방식도 좋은 리팩토링이 될 수 있다
</details>

3. 테스트 클래스들을 그룹화 시키면 장점은?
 <details>
<summary> 정답 </summary>
  공통된 메서드를 테스트 그룹 클래스의 이름에 넣고, 테스트자체에는 이름을 빼 좀더 명확한 해석이 가능한 이름을 만들수 있다
</details>


