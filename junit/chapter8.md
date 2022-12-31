1. 리팩토링을 해보자!(3개 예상)

```kotlin

data class Person(
  private val name: Name,
  private val alive: Boolean,
  private val friends: List<Person> = emptyList()
) {
  data class Name(
    val fullName: String,
    val firstName: String,
    val lastName: String,
  )

  fun isNotAlive(): Boolean {
    //TODO
    return true
  }
  
  fun hello() {
    if(!alive) { 
    print("죽은자의 말 :")
    }
    println("Hello my friend!")
  }
  
  fun Bye() {
    f(!alive) { 
    print("죽은자의 말 :")
    }
    println("Good bye my friend!")
  }  
  
  fun goodGoodMake: Name(val f: String, val l: String) {
    val goodGood = f + l
    return Name(firstName = f, lastName = l, fullName = goodGood)
  }
  
  fun printMyFriends(): List<Person> {
    val myFriends: List<String> = friends.map { it.firstName }
    myFriends.forEach { myFriend -> println("My friend ${myFriend}!! Good!!") }
  }
}  
 ```
 <details>
<summary> 정답 </summary>
  1. isNotAlive에다가 "Die" 외치는걸 구현하자! </br>
  2. goodGoodMake 이름미쳤다 파라미터도 이름을 잘지어주자 </br>
  3. printMyFriends 에서 인라인 시켜주자</br>
</details>
 
2. 꾸준한 리팩토링을 통해 거대한 한개였던 메소드가 4개정도로 쪼개졌습니다. 그에 따른 장점은 무엇이 있나요? 
 
 <details>
<summary> 정답 </summary>
쪼개진 메소드마다 각각 명확한 내용이 생기고 고립성이 생긴다</br>
가독성이 증가한다</br>
유닛테스트하기에 용이해진다
</details>
