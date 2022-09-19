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
- spdl
