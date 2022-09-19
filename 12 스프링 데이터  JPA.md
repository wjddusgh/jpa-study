# 12.1 스프링 데이터 JPA 소개

- 데이터 접근 계층 개발 시 지루하게 반복되는 CRUD 문제를 세련된 방법으로 해결
- CRUD 처리를 위한 공통 인터페이스 제공 -> 실행 시점에 구현 객체를 동적으로 제공해줌
- 동적으로 제공해주므로 개발 시 구현 클래스 없어도 됨
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
  Member findByUsername(String username); // pk 가 아닐 때 선언 만으로 메소드 이름을 통해 JPQL 메소드 뚝딱
}

public interface ItemRepository extends JpaRepository<Item, Long> {
}
```
# 12.1.1 스프링 데이터 프로젝트
- 스프링 데이터 JPA 는 스프링 데이터 프로젝트의 하위 프로젝트중 하나이다
- 스프링 데이터 프로젝트는 JPA, 몽고DB, REDIS, HADOOP 등의 데이터 저장소 접근을 추상화해주려고 노력한다

# 12.2 스프링 데이터 JPA 설정

- 필요 라이브러리 : spring-data-jpa (spring-data-commons에 의존)
- 환경 설정 : jpa:repositories 혹은 JavaConfig로 `@EnableJpaRepositoreis(basePackages = "레포지토리이름")` 어노테이션 추가
- 이러면 자동으로 레포지토리를 동적으로 생성한 뒤 스프링 빈에 등록한다

# 12.3 공통 인터페이스 기능

- 공통 인터페이스 기능을 사용하기 위해 상속을 이용한다
- 제너릭 타입은 첫번째 : 엔티티 타입, 두번째 : 식별자 타입 을 제공해야 한다
- 주요 메소드
  - save() : 식별자 값이 없으면 persist() 호출, 있으면 merge() 호출
  - delete()
  - findOne()
  - getOne() : 프록시 조회, 내부에서 getReference() 호출
  - findAll() 

# 12.4 쿼리 메소드 기능
- 메소드 이름으로 쿼리 생성
- 메소드 이름으로 JPA NamedQuery 호출
- @Query 어노테이션을 사용해서 레포지토리 인터페이스에 쿼리 직접 정의

## 12.4.1 메소드 이름으로 쿼리 생성
- 말 그대로 메소드 이름을 통해 스프링 데이터 JPA 가 JPQL 만들어낸다
- 주요 키워드
  - And
  - Is, Equals
  - Between
  - LessThanEqual
  - GreaterThan
  - After, Before
  - isNull
  - Like
  - Containing
  - StartingWith
  - OrderBy
  - True
  - IgnoreCase

## 12.4.2 JPA NamedQuery
```java
@Entity
@NamedQuery(
  name="Member.findByUsername",
  query="select m from Member m where m.username = :username")
public class Member {
  ...
}
```
- 이후 `em.createNamedQuery()` 사용해야 했다
- 근데 JPA NamedQuery는 놀랍다
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
  List<Member> findByUsername(@Param("username") String username);
```
- 선언한 도메인 클래스 + . + 메소드이름으로 Named Query 찾아서 실행한다
- 만약 Named Query 없다면 메소드이름으로 쿼리 생성 전략 사용

## 12.4.3 @Query, 레포지토리 메소드에 쿼리 정의
```java
@Query("select m from Member m where m.username = ?1")
Member findByUsername(String username);
```
- Native SQL 을 사용하고 싶을땐 @Query 의 속성으로 `nativeQuery = true` 를 설정하면 된다 ( 네이티브는 파라미터가 ?1이아닌 ?0부터 시작)

## 12.4.4 파라미터 바인딩
- 위치기반, 이름기반 파라미터 바인딩 모두 지원
- 코드 가독성과 유지보수를 위해 이름기반 바인딩 추천

## 12.4.5 벌크성 수정 쿼리
- 수정이니까 `@Modifying` 애노테이션을 메소드에 붙이면 된다
- 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트를 초기화하고 싶으면 `clearAutomatically = true` 속성을 설정하면 된다

## 12.4.6 반환 타입
- 결과가 단건이면 반환타입 지정, 한건 이상이면 컬렉션 인터페이스 사용
- 단건 메소드 호출시 내부에서 `Query.getSingleResult()` 메소드 호출, 예외 발생시 무시하고 Null 반환

## 12.4.7 페이징과 정렬
- Sort, Pageable 제공
- Pageable 사용시 Page<> 인터페이스로 반환 가능
  - count 쿼리를 사용 가능해짐 
- Page 인터페이스도 다양한 메소드 제공해줌 ( getSize, isFirstPage 등)

## 12.4.8 힌트
- `@QueryHints` 사용, SQL 힌트는 아님

## 12.4.9 Lock
- 쿼리시 락을 걸려면 @Lock 사용, LockModeType 속성으로 락 단계 설정

# 12.5 명세

- JpaSpecificationExecutor 인터페이스 상속을 통해 사용
- 명세 정의는 `toPredicate()` 메소드 사용

# 12.6 사용자 정의 레포지토리 구현

- 전체를 구현하면 머리아프니 필요한것만 개발자가 구현시킬수 있게끔 할 수 있다
- 사용자 정의 인터페이스 구현 클래스는 이름 뒤에 `Impl` 를 붙여야 한다 ( postfix는 변경 가능)

# 12.7 Web 확장
- 스프링 MVC 에서 사용가능한 편리한 기능 제공

## 12.7.1 설정
- SpringDataWebConfiguration 빈으로 등록하는 법
- @EnableSpringDataWebSupport 사용

## 12.7.2 도메인 클래스 컨버터 기능
- pk나 다른 필드로 받아도 컨버터가 엔티티로 찾아와줘서 컨트롤러에 전달

## 12.7.3 페이징과 정렬 기능
- Pageable 을 파라미터로 받아 정렬, 페이징 사용 가능

# 12.10 스프링 데이터 JPA와 QueryDSL 통합
- 스프링 데이터 JPA는 두가지 방법으로 QueryDSL 지원
  - QueryDslPredicateExecutor
  - QueryDslRepositorySupport

## 12.10.1 QueryDslPredicateExecutor 사용
- 단순히 이거 상속받으면 된다
- 하지만 join, fetch 사용 불가

## 12.10.2 QueryDslRepositorySupport 사용
- 상속받고 사용자 정의 레포지토리에서 구현을 하면 된다

