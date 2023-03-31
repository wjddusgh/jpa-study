## 에러 처리하기
- 외부 서비스와 많은 커뮤니케이션 하는 애플리케이션 설계시 모든 종류의 예외 상황 처리해야함
- onError
  - 리액티브 스트림 스펙의 필수 요소 -> pub가 예외 처리 경로로 전파 가능
  - 최종 구독자가 onError 핸들러 정의 안하면 `UnsupportedOperationException` 발생
  - 스트림이 종료됐다고 정의 -> 시그널 받으면 시퀀스가 실행 중지
  - 다양한 전략
    - sub에서 핸들러 정의
    - onErrorReturn : 예외 발생 시 사전 정의된 정적 값 또는 예외로 계산된 값으로 대체
    - onErrorResume : 예외 catch하고 대체 워크플로 실행
    - onErrorMap : 예외 catch하고 상황을 더 잘 나타내는 다른 예외로 변환
    - retry : 오류 발생 시 다시 실행 시도
    - retryBackoff : 재시도 할 때마다 대기시간 증가
 
 - defaultIfEmpty, switchIfEmpty, timeout 연산자 등을 사용해 다양한 에러 처리 가능

```java
public Flux<String> recommendedBooks(String userId) {
        return Flux.defer(() -> {
            if (random.nextInt(10) < 7) {   //70% 확률로 실패
                return Flux.<String>error(new RuntimeException("Conn error"))
                    .delaySequence(Duration.ofMillis(100));   //시그널 지연
            } else {
                return Flux.just("Blue Mars", "The Expanse")
                    .delayElements(Duration.ofMillis(50));    //?
            }
        }).doOnSubscribe(s -> log.info("Request for {}", userId));
    }

public void handlingErrors() throws InterruptedException {
        Flux.just("user-1")
            .flatMap(user ->
                recommendedBooks(user)
                    .retryBackoff(5, Duration.ofMillis(100))      //실패시 100ms 지연시간으로 재시도(5회까지)
                    .timeout(Duration.ofSeconds(3))               //앞선 전략이 3초후에도 결과 안가져오면 타임아웃. 오류시그널
                    .onErrorResume(e -> Flux.just("The Martian")) //사전에 정의된 대체 워크플로 실행
            )
            .subscribe( //구독 필수인듯
                b -> log.info("onNext: {}", b),
                e -> log.warn("onError: {}", e.getMessage()),
                () -> log.info("onComplete")
            );
    }        
```
- 리액터 프로젝트는 예외적인 상황을 처리할 수 있게 해주고 결과적으로 응용 프로그램의 복원력을 향상시키는데 도움이 되는 다양한 도구 제공

## 배압 다루기
- 일부 컨슈머는 제한되지 않은 데이터에 순진하게 응답한 다음 생성된 부하를 처리하지 못한다
- 또는 수신 메시지 비율에 엄격한 제한을 가하기도 한다
- 배치 처리 기법이 도움이 됨(스트림 내의 원소 일괄 처리하기 절에서 봤다고 함)
- onBackPressure : 구독자가 정의, 기본적으로 제한되지 않은 요구를 pub한테 요청(많이보내나봄), 결과를 다운스트림으로 푸시
  - onBackPressureBuffer : 컨슈머 부하가 크면 큐를 이용해 버퍼링, 여러 옵션으로 동작 쉽게 조정 가능
  - onBackPressureDrop : 부하가 크면 일부 데이터 삭제, 사용자 정의 핸들러(파라미터에 넣어줌)를 사용해 삭제된 원소를 처리 가능
  - onBackPressureLast : Drop과 유사, 가장 최근에 수신된 원소를 기억하고, 요청 발생시 푸시. 오버플로우에서도 항상 최신 데이터를 수신가능
  - onBackPressureError : 컨슈머가 처리 유지 못하면 에러 발생
- limitRate(n) : 다운스트림 수요를 n(시간)보다 크지 않은 작은 규모로 나눔 ex. `.limitRate(1, Duration.ofSeconds(1))` : 1초간격엔 최대 1개
- limitRequest(n) : 다운스트림 총 요청 값을 제한, n개의 이벤트를 전송 후 pub는 스트림 닫음, 요청값 넘기면 onError 

## Hot 스트림과 Cold 스트림
- 콜드 퍼블리셔
  - 구독자가 나타날 때마다 해당 구독자에 대해 모든 시퀀스 데이터가 생성됨, 구독자 없이는 데이터 생성 안됨
  - 대표적으로 HTTP 요청이 이런 식으로 동작 
```java
 public void coldPublisher() {
        Flux<String> coldPublisher = Flux.defer(() -> {
            log.info("Generating new items");
            return Flux.just(UUID.randomUUID().toString());
        });

        log.info("No data was generated so far");
        coldPublisher.subscribe(e -> log.info("onNext: {}", e));
        coldPublisher.subscribe(e -> log.info("onNext: {}", e));
        log.info("Data was generated twice for two subscribers");
    }
```
- 핫 퍼블리셔
  - 데이터 생성이 구독자의 존재여부에 의존 X
  - 첫 구독자가 구독 시작하기 전에부터 원소 만들기 가능
  - 구독자가 나타났다고 이전 생성된 값을 주는것 뿐 아니라 새로운 값만 보낼수도 있음
  - 데이터 방송 시나리오가 이런 식으로 동작
  - 대부분의 핫 퍼블리셔는 Processor 인터페이스 상속
  - 하지만 팩토리 메서드 `just` 는 게시자가 빌드될 때 값 한번만 계산되고, 새 구독자 나타나면 다시 계산되지 않는 형태의 핫 퍼블리셔(?)
    - defer로 래핑시 콜드 퍼블리셔로 전환 가능
 
 ### 스트림 원소를 여러 곳으로 보내기
 - ConnectableFlux : 콜드를 핫으로 변환, 가장 수요가 많은 데이터 생성하고 다른 모든 가입자가 데이터 처리할 수 있도록 캐싱됨
  - 핫을 콜드로 변환하려면 refCount(), autoConnect() 등의 메서드를 활용해야함(쟤네가 리턴이 그냥 Flux, Mono임) 
 ```java
 public void connectExample() {
        Flux<Integer> source = Flux.range(0, 3)
            .doOnSubscribe(s ->
                log.info("new subscription for the cold publisher"));

        ConnectableFlux<Integer> conn = source.publish();       // source 라는 콜드 퍼블리셔를 핫 퍼블리셔로 전환(publish())

        conn.subscribe(e -> log.info("[Subscriber 1] onNext: {}", e));
        conn.subscribe(e -> log.info("[Subscriber 2] onNext: {}", e));

        log.info("all subscribers are ready, connecting");
        conn.connect();   // 이때부터 다운스트림으로 데이터 보냄
    }
 ```
### 스트림 내용 캐싱하기
 - 리액터에는 이벤트 캐싱을 이용하는 연산자로 `cache` 연산자가 이미 존재함
 - 근데 ConnectableFlux 사용하면 다양한 데이터 캐싱 전략 쉽게 구현 가능
 ```java
 public void cachingExample() throws InterruptedException {
        Flux<Integer> source = Flux.range(0, 2) // 콜드 퍼블리셔 생성
            .doOnSubscribe(s ->
                log.info("new subscription for the cold publisher"));

        Flux<Integer> cachedSource = source.cache(Duration.ofSeconds(1));   // 1초동안 cache 연산자와 함께 콜드 퍼블리셔 캐시

        cachedSource.subscribe(e -> log.info("[S 1] onNext: {}", e));   
        cachedSource.subscribe(e -> log.info("[S 2] onNext: {}", e)); // 구독자 생긴 직후 또 구독자 생김

        Thread.sleep(1200);   //캐시 만료 대기

        cachedSource.subscribe(e -> log.info("[S 3] onNext: {}", e)); //캐시 만료 후 구독자
    }
 ```
 - 두번째 구독자는 캐싱된 데이터를 공유함
 
### 스트림 내용 공유
 - `share` 연산자 사용시 콜드 -> 핫 변환 가능, 구독자가 각 신규 구독자에게 이벤트 전파하는 방식
  - 구독자 -> 신규 구독자라서 신규 구독자는 전파해주는 구독자가 이전에 받았던 데이터는 받을 수 없다

### 시간 다루기
- 리액티브 프로그래밍은 비동기적이므로 본질적으로 시간의 축이 있다고 가정한다
- `interval` 연산자로 주기적으로 이벤트 생성(0부터 시작하는 무한한 스트림), `delayElements` 연산자로 원소 지연, `delaySequence` 연산자로 모든 신호 지연
- `timestamp` : 데이터가 발생한 시간 정보 추가
- `elapsed` : 시간 간격 정보 추가
- 근데 자바는 예정된 이벤트를 `ScheduledExcutorService` 사용해서 정확하지는 않다고함. 주의

### 리액티브 스트림을 조합하고 변환하기
`transform`
- 두 스트림을 별도의 객체로 추출할 수 있어서 스트림 구조 자체를 변경 가능
- 스트림 라이프사이클 결합 단계에서 동작을 한번만 변경
- `composer` 는 구독자가 도착할 때마다 변환 작업 수행(분기 가능)
- 리액티브 응용 프로그램에서 코드를 재사용할 수 있는 강력한 도구

### Processor
- Pub 이면서 동시에 Sub 이기에 이 인스턴스는 구독이 가능하고, 시그널을 수동으로 보낼 수 있다
- 리액터 개발팀은 오류발생하기 쉬운 이거 사용안하는걸 권장 -> 연산자의 조합으로 대체할 수 있다, 외부 api 적용 시 팩토리 메서드 사용이 더 나을것
- 종류
  - Direct : 프로세서의 sink를 사용자가 직접 구현해 데이터 푸시만 가능
    - DirectProcessor : 배압을 처리 안하고, 다수의 구독자에게 이벤트 게시
    - UnicastProcessor : 내부적으로 큐를 이용해 배압 처리, 대부분경우 하나의 sub에게만 서비스 제공
  - Synchronous : pub 구독하거나 수동으로 데이터 푸시
    - EmitterProcessor : 여러 구독자에게 서비스 제공, 각 구독자의 수요 요청 중시, 동기식으로 단일 게시자의 데이터만 처리 가능
    - ReplayProcessor : 입력 데이터를 캐싱하기 위한 몇가지 전략 허용
  - Asynchronous : 여러 pub 처리하기 위해 RingBuffer 데이터 구조 사용, 옵션 수가 많아 전용 빌더 api 있음
    - TopicProcessor : 리액티브 스트림과 호환, sub마다 별도의 스레드 생성, sub 제한 존재
    - WorkQueueProcessor : 런타임에 사용하는 리소스 크기 줄이기 위해 리액티브 스트림 요구 사항중 일부를 완화해 사용

### 리액터 프로젝트 테스트 및 디버깅하기
`reactor-test` 라이브러리 존재
- 조립 단계에서 적용 가능한 디버깅 기능 제공 `Hooks.onOperatorDebug()`
- Flux, Mono는 log 메서드 제공

### 리액터 추가 기능
- `reactor-addons`
  - `reactor-adapter` : 리액티브 타입 및 스케줄러에 대한 어댑터 제공 ,Akka(액터 라고함) 와의 통합 제공  
  - `reactor-logback` : 고속의 비동기 로깅 제공
  - `reactor-extra` : 추가 유틸리티(TupleUtils 등)
- 이외에도 rabbitmq, netty등의 확장 모듈이 있다

# 리액터 프로젝트 심화학습
## 리액티브 스트림의 수명 주기
- 리액터에서 멀티스레딩이 작동하는 방법과내부 최적화가 구현되는 방법을 이해하자
### 조립 단계
- 스트림 수명 주기의 첫번째 부분
- 연쇄형 api 제공
  - (빌더랑 다름, 빌더는 가변적인 객체를 생성하며, 다른 객체를 생성하기 위해 build() 같은 최종 함수 호출)
  - 얘는 불변성 제공, 각각의 연산자가 새로운 객체를 생성, 이런 프로세스를 조립(어셈블링) 이라고함
  - concatWith -> concatWith 처럼 연쇄적인 시퀀스는 내부 구현으로 하나의 연결로 압축 가능 -> 성능 향상
- 스트림 구성을 조작하고 리액티브 시스템을 구축하는데 필수적인 디버깅 최적화나 모니터링, 더 나은 스트림 전달을 위한 다양한 기술을 적용할 수 있는 단계

### 구독 단계
- 특정 pub 구독할 때 발생
- 실행 플로우 만들기 위해 내부적으로 pub를 다른 pub에게 전달 -> pub 체인(sub과 역피라미드 형태)
- 조립 단계와 동일한 최적화 수행 가능
- 멀티스레딩 지원하는 일부 연산자는 구독이 발생하는 작업자를 변경 가능

### 런타임 단계
- 스트림 실행의 마지막 단계
- pub sub 간의 실제 신호 교환
  - 교환하는 첫 두 신호는 `onSubscribe`, `request`
- 런타임 중에 데이터는 각 sub 체인을 거쳐 단계마다 다른 기능 실행
- 신호 교환량을 줄이기 위한 최적화 적용 가능
  - Subscription.request() 는 요청 수요를 저장하는 volatile 필드에 쓰기가 발생해서 비용이 큼, 이걸 줄여야함

## 리액터에서 스레드 스케줄링 모델
- 멀티스레딩 실행을 위해 제공하는 연산자

### publishOn 연산자
- 런타임 실행의 일부를 지정된 워커로 이동할 수 있게 해줌
- 내부적으로 전용 워커가 메시지를 하나씩 처리할 수 있도록 새로운 원소를 제공하는 큐를 가지고 있음
  - 그래서 직렬성이 있어 원소의 순서는 항상 예측 가능함
### publishOn 연산자를 이용한 병렬 처리
- 원소의 처리 시간을 유지하되, 처리 단계 사이에 비동기 영역(publishOn) 추가하면 좌우간의 기다림이 필요없어져서 빨라짐

### subscribeOn 연산자
- 구독 체인에서 워커의 작업 위치를 변경 가능
- 솔직히 책보고도 이해가안됨

### parallel 연산자
- 하위 스트림에 대한 플로 분할, 분할된 플로 간 균형 조정 역할
- ParallelFlux 라는 다른 유형의 Flux를 동작시킴 -> 다수의Flux 추상화한 것으로 Flux 간에 데이터 크기가 균형을 이룸

### Scheduler
- schedule, createWorker 라는 두가지 핵심 메서드를 가진 인터페이스
- `schedule`
  - Runnable 작업 예약 가능
  - 워커 풀을 나타냄
- `createWorker`
  - Worker 인터페이스를 통해 Runnable 예약 가능
  - Thread 또는 리소스를 추상화한 것
- 구현체
  - `SingleScheduler` : 모든 작업을 한 개의 전용 워커에 예약 가능, 시간에 의존적, `Scheduler.single()` 호출로 생성
  - `ParallelScheduler` : 고정된 크기의 작업자 풀에서 작동, CPU 제한적인 작업에 적합, `Scheduler.parallel()` 호출로 생성
  - `ElasticScheduler` : 동적으로 작업자 만들고 스레드 풀 캐시, 최대 개수 제한없으므로 I/O작업에 적합, `Schdeuler.elastic()`

### 리액터 컨텍스트
- 스트림을 따라 전달되는 인터페이스
- 나중에 런타임 단계에서 필요한 컨텍스트 정보에 액세스 할 수 있도록 하는 것
- 제일 중요한건 ThreadLocal 대체(비동기에선 이거 못씀)
  - 비동기식에선 하나의 스레드에서 여러 요청을 처리할 수 있기 때문에 다른 스레드의 변수 공유할 수도 있다. 이경우 곤란
  - Reactor Context는 불변객체라 괜찮나봄, subscriberContext 
