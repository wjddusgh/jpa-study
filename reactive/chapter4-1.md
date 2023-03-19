# 4 리액터 프로젝트 - 리액티브 앱의 기초

## 리액터 프로젝트의 간략한 역사
- 리액티브 스트림 스펙은 API 및 규칙만 정의하고, 구현된 라이브러리가 아님(JPA 표준 같은거인듯)
- `리액터 라이브러리` 는 예술의 경지에 다다른 라이브러리 라고함

### 리액터 프로젝트 버전 1.x
- 처음부터 비동기 논블로킹 처리를 지원하도록 설계됨
- 본질적으로 리액터 패턴, 함수형 프로그래밍 및 리액티브 프로그래밍과 같은 메시지 처리에 대한 모범 사례 통합
  - 모든 이벤트가 큐에 추가되고 이벤트는 나중에 별도의 스레드에 의해 처리됨
- Streams 클래스는 현재 권장되지 않는 옛날 클래스
- 라이브러리에 배압 조절 기능이 없다(프로듀서 스레드를 차단하거나 이벤트를 생략하는것 정도)

### 리액터 프로젝트 버전 2.x
- 이벤트버스 및 스트림 기능을 별도의 모듈로 추출함
- 새로운 리액터 api는 자바 컬렉션 api와 더욱 쉽게 통합 가능
- RxJava api와 훨씬 비슷해짐
- 스트림 생성,소비에 더불어 배압 관리, 스레드 처리, 복원력 지원등을 위한 다양한 기능 추가됨
```java
stream
  .retry()                                    // 재시도 연산자를 통해 플로우에 복원력을 부여, 오류 발생시 업스트립 작업 다시 실행
  .onOverflowBuffer()                         // 푸시 모델만 지원할 때 버퍼 및 드롭을 사용해 배압 관리
  .onOverflowDrop()
  .dispatchOn(new RingBufferDispatcher("test")) // 새로운 Dispatcher를 이용해 해당 리액티브 스트림에서 동시에 작업함에 따라 메시지 비동기적 처리
```
- Reactor 객체가 EventBus 로 이름 바꿈 (리액티브 스트림 사양 지원하도록 다시 설계됨)

## 리액터 프로젝트 필수 요소
- 목적 : 비동기 파이프라인 구축 시 콜백 지옥과 깊게 중첩된 코드를 생략시키기
- 리액터 api는 연산자를 연결해서 사용하는 것을 권장
- 동시성에 좌우되지 않도록 설계됐으므로 동시성 모델을 적용하지 않음(근데 배제한게 아니라 최소화 시킨거라고 함)

## 리액티브 타입 - Flux와 Mono
- 리액터 프로젝트에는  Publisher<T>의 구현체로 `Flux<T>, Mono<T>` 가 있다
  
### Flux  
- Flux 스트림을 연산자를 통해 다른 Flux 스트림으로 변환
- 0, 1, N 개의 요소를 생성할 수 있는 일반적인 리액티브 스트림
- 무한 스트림 예시 `Flux.range(1, 5).repeat()`
  - repeat()은 앞선 스트림이 끝나면 onComplete 신호 수신 후 다시 동작 반복시킴
  
  
### Mono
- 최대 하나의 요소를 생성할 수 있는 스트림
- 0 또는 1이기에 버퍼 중복, 값비싼 동기화 생략 가능
- 클라이언트에게 작업이 완료됐음을 알릴 때 사용 가능(Mono<void> 반환 후 처리 완료시 onComplete(), 실패시 onError(), 데이터는 안보내는거)
- Flux와 변환 가능
  - `Flux <T>.collectList()` --> `Mono<List<T>>` 반환
  - `Mono<t>.flux()` --> `Flux<T>` 반환
  - Mono -> Flux -> Mono 변환 같은경우 스마트 최적화 해줌

### RxJava 2의 리액티브 타입
- Observable : non-nullm, 배압 x, 낡은 타입
- Flowable : Flux 같은거, Publisher 구현
- Single : Publisher 상속 x, Mono보단 CompletableFuture 의미 잘 표현, 구독시에만 처리 시작
- Maybe : Mono 느낌
- Completable : onNext() 못보내고, 완료,실패 신호만 보내기 가능

  ### Flux와 Mono 시퀀스 만들기
  - 팩토리 메서드 패턴 제공 
  ```java
  Flux<String> s1 = Flux.just("hello", "world");
  Flux<String> s2 = Flux.fromArray(new Integer[]{1,2,3});
  ```
  - Mono는 주로 nullable, Optional 타입과 함께 사용
    - HTTP 요청이나 DB 쿼리 같은거 래핑해서 사용하면 좋다
  - empty() 메소드로 빈 인스턴스 생성 가능

  ### 리액티브 스트림 구독하기
  - subscribe() 메서드 사용가능
  - Disposable 인터페이스의 인스턴스 반환(구독 취소하는데 사용가능 `disposable.dispose()`)
  ```java
  Flux.just("a","b","c")
      .subscribe(
        data -> log.info("onNext: {}", data),
        err -> {},
        () -> log.info("fin"));
  //////////
  // onNext: a
  // onNext: b
  // onNext: c
  // fin
  ```
  - 구독쪽에서 직접 수요, 요청을 제어도 가능(subscription 사용)
  
  ### 사용자 정의 Subscriber 구현하기
  - 직접 커스텀 구현도 가능
  
  ### 연산자를 이용해 리액티브 시퀀스 변환하기
  - 도구 분류
    - 기존 시퀀스 변환
    - 시퀀스 처리 과정을 살펴보는 ㅔ서드
    - Flux 시퀀스를 분할 또는 결합
    - 시간을 다루는 작업
    - 데이터를 동기적으로 반환
  
  ### 리액티브 시퀀스의 원소 매핑하기
  - map 연산자 제공
  ### 리액티브 시퀀스 필터링하기
  - filter 제공
  - ignoreElements : Mono<T> 반환하고 어떤 원소도 다 필터링해버림. 겨로가 시퀀스는 원본 시퀀스가 종료된 후에 종료(스트림의 완료가 궁금한경우 쓰는듯)
  - take() 연산자로 유입되는 원소 개수 제한 가능
  - takeLast는 스트림 마지막 원소만 반환
  - takeUntil() : 조건 만족시까지 전달
  - elementAt() : n번째 원소 가져옴
  - single : 단일 항목 내보내고, 빈 소스면 익셉션 발생, 복수의 반환갯수도 익셉션 발생
  - skip() : 양, 시간 기준해서 원소 가져오거나 무시할수 있음
  - takeUntilOther() 로 특정 스트림에서 메시지가 도착할 때까지 필터링
  
  ### 리액티브 시퀀스 수집하기
  - collectList(), collectSortedList() 등
  - Collection 타입으로도 변환 가능함
  
  ### 스트림의 원소 줄이기
  - hasElement :특정 원소의 존재 유무
  - 근데 필터링도 줄이는거 아닌가?
  
  ### 스트림 조합
  - concat, merge(합쳐진 소스는 별개로 동시에 구독됨), zip, combineLatest
  
  ### 스트림 내의 원소 일괄 처리하기
  - List 컨테이너를 이용한 버퍼링(반환 타입은 Flux<List<T>>)
  - 윈도우잉(Flux<Flux<T>> 형태, 이 경우 스트림 내부원소는 값이 아니라 다른 스트림이 되므로 별도의 추가 처리 가능)
  - 그룹화(GroupedFlux<K, T>)
  - 버퍼링 및 윈도우 처리하는 경우
    - 처리된 원소의 수에 기반. N개의 원소를 처리할 때마다 신호를 보내야 할 때
    - 시간 기반, N분마다 신호를 보내야 할 때
    - 특정 로직에 기반, 짝수마다, 5의 배수마다 등
    - 실행을 제어하는 다른 Flux에서 전달된 이벤트에 기반
  
  ### flatMap, concatMap, flatMapSequential 연산자
  - 함수형 프로그래밍의 핵심 변환인 flatMap
  
  ### 샘플링
  - 처리량이 많은 경우 일부 이벤트만 처리하게끔 할 수 있음
  - sample() 메서드사용 시 정해진 규칙에 따라 일부만 처리함
  
  ### 리액티브 시퀀스를 블로킹 구조로 전환하기
  - 리액티브 앱에서 블로킹 처리를 해서는 안되지만, 상위 api에서 필요한 경우가 있다
  - 스트림 차단 후 결과를 동기적으로 생성하는 옵션
    - toIterable() : Flux -> Iterable
    - toStream() : Flux -> Blocking Stream API(리액터 3.2부턴 내부적으로 toIterable() 사용한다고 함)
    - blockFirst() : 업스트림이 첫번째 값을 보내거나 완료될 떄까지 현재 스레드를 차단함
    - blockLast() : 마지막 값 보내거나 완료 까지 스레드 차단, onError 신호의 경우 차단된 스레드에 예외가 발생
  - 클라이언트가 블로킹을 차단하는 것보다 빠리 도착한 이벤트를 큐에 저장할 수 있음
  
  ### 시퀀스를 처리하는 동안 처리 내역 살펴보기
  - doOnNext() : 각 원소에 액션 수행하게 해줌
  - 그 외doOn머시기들
  
  ### 데이터와 시그널 변환하기
  - materialize, dematerialize
