# 19. 스파크

아파치 스파크 : 대용량 데이터 처리를 위한 클러스터 컴퓨팅 프레임워크
### 맵 리듀스와의 차이점
- 데이터 처리 모델
  - 맵리듀스 : Map 과 Reduce 두 단계로 데이터를 처리함
  - 스파크 : DAG(Directed Acyclic Graph) 기반 연산모델
- 데이터 보관
  - 맵리듀스 : 맵, 리듀스 사이를 디스크에 보관하고, 작업이 끝날 때 마다 중간 결과를 디스크에 저장
  - 스파크 : 중간 결과를 메모리에 저장
- 프로그래밍
  - 맵리듀스 : 자바 기반, 맵 단계와 리듀스 단계를 명확히 구현해야함
  - 스파크 : 자바, 스칼라, 파이썬 의 여러 언어 지원하고, RDD, DataFrame, Dataset 등 더 고수준의 API를 제공
- 장애허용(fault tolerance)
  - 맵리듀스 : 데이터를 HDFS 에 저장하여 복구
  - 스파크 : 데이터를 미리 계산하는게 아닌 RDD(Resilient Distributed Dataset) 으로 추상화하여, 노드가 실패하면 RDD 를 재실행하여 복구
- 스트리밍 여부
  - 맵리듀스 : 기본적으로 배치 처리에 최적화
  - 스파크 : Spark Streaming 을 통해 실시간 데이터 처리 지원
### 맵 리듀스와 비교해서 장점
- 메모리 내 처리를 통해 빠른 데이터 처리 가능
- 다양한 데이터 소스와 연산을 제공
- 배치 처리 뿐 아니라, 실시간 처리, 머신러닝, 그래프 처리도 가능

### 스파크 컨텍스트(Spark Context)
- 스파크 애플리케이션의 진입점, Spark 와 클러스터 간의 연결을 관리하고, 클러스터 상의 노드들에 작업을 분배
- RDD API 직접적으로 사용할 때 필요함
- 스파크 대화형(스칼라 쉘, pyspark) 실행시 쉘이 자동으로 sc 라는 이름으로 스파크 컨텍스트 변수 생성
- 이외의 경우 `SparkConf conf = new SparkConf().setAppName("MyApp")`, `conf = SparkConf().setAppName("MyApp")`같이 생성

### 스파크 세션(Spark Session)
- Spark 2.x 버전부터 도입됨
- SparkContext, SQLContext, 그리고 HiveContext를 하나의 진입점으로 통합한 것 
- 데이터의 구조적 처리, SQL 쿼리, Hive 연산 등을 더 쉽고 일관적으로 수행할 수 있음
- 여러 형식(JSON, CSV, JDBC)의 데이터를 DataFrame 으로 변경
- `session.sparkContext` 같은 방식으로 스파크 컨텍스트가 내부에 존재

### 주요 API
- RDD API : Spark 의 가장 기본적인 분산 컬렉션 구조
  - textFile(): 텍스트 파일을 로드하여 RDD를 생성
  - map(): 각 요소에 함수를 적용한 새 RDD를 반환
  - filter(): 조건에 맞는 요소만을 가진 새 RDD를 반환
  - reduce(): 요소들을 하나의 값으로 리듀스
  - collect(): RDD의 모든 요소를 배열로 반환
- DataFrame API : 테이블 형식의 데이터 다루는 api
  - read: 다양한 포맷의 데이터를 읽어 DataFrame을 생성
  - select(): 특정 컬럼을 선택
  - groupBy(): 특정 컬럼에 따라 그룹화
  - filter(): 조건에 따라 데이터를 필터링
- MLlib API: 머신러닝 알고리즘과 유틸리티를 제공
  - classification: 분류 알고리즘을 제공
  - regression: 회귀 알고리즘을 제공
  - clustering: 군집화 알고리즘을 제공

### 스파크 잡
- 맵리듀스는 맵, 리듀스 두 단계로만 구성됨
- 스파크는 하나의 액션 이 하나의 잡으로 구성됨, 잡은 여러 스테이지로 구성될 수 있고, 각 스테이지는 여러 태스크로 구성될 수 있음
- 액션 예시: `count`, `collec`t, `saveAsTextFile`
- 잡은 항상 RDD 및 공유변수를 제공하는 애플리케이션(SparkContext 인스턴스로 표현되는)의 콘텍스트 내에서 실행됨

### 스파크 스테이지
- DAG(Directed Acyclic Graph) 로 표현되며, 연산의 의존성을 보여줌
- 주로 shuffle 연산(groupBy, reduceByKey, join 등)을 경계로 나눠짐
  - 파티션 재분배나 RDD 를 메모리에 저장하는 경우에도 스테이지가 나뉠 수 있음 
- 스테이지 내의 연산은 병렬로 이루어지며, 메모리 내에서 처리 가능

### 스파크 태스크
- 스테이지의 실행 단위
- 클러스터 내의 개별 노드에서 실행됨
- RDD의 하나의 파티션에서 실행되는 하나의 연산 단위

## 탄력적인 분산 데이터셋 RDD
### RDD 생성
1. 객체의 인메모리 컬렉션으로 생성
- 적은 양의 입력 데이터를 병렬로 처리하는 CPU 부하 중심의 계산에 유용
```scala
val params = sc.parallelize(1 to 10)
val result = params.map(performExpensiveComputation)  // 병렬 실행
```
2. 외부 데이터셋에 대한 참조 생성
```scala
val text: RDD[String] = sc.textFile(inputPath)
```
- 경로에는 로컬 파일시스템 혹은 HDFS 경로 지정 가능
- 파일 스플릿 방식은 맵리듀스와 동일해서 HDFS 블록당 스파크 파티션 하나지만, 파라미터로 두번째에 스플릿 수 지정 가능
- 다수의 텍스트 파일도 한번에 전체 파일로 처리 가능(모두 메모리에 저장되므로 메모리가 크거나, 작은 파일들이어야 적합)
- 텍스트 파일 외에도 다양한 파일 타입 지원

## 트랜스포메이션과 액션
- 트랜스포메이션 : 기존 RDD 에서 새로운 RDD 로 변환하는 연산, 메모리에 캐싱 가능
- 액션 : 특정 RDD 를 계산하여 어떠한 결과를 만드는 것
- 연산자 구분법: 반환 타입이 RDD 면 트랜스포메이션, 아니면 액션
### Narrow Transformation
- 각 입력 파티션이 단 하나의 출력 파티션에만 영향을 줌
- 연산은 한 노드에서 자체적으로 완료될 수 있고, 데이터의 이동이 적어 성능이 뛰어남
- map(), filter(), union()

### Wide Transformation
- 여러 입력 파티션이 여러 출력 파티션에 영향을 미칠 수 있음
- 연산이 클러스터 전체에서 수행되어야 하므로, 데이터의 이동(shuffle)이 발생하여 성능에 영향을 줄 수 있음 
- groupByKey(), reduceByKey(), join()
### Lazy Evaluation
- Transformation 코드를 실행할 때 실제 연산이 일어나지 않음
- 대신 Spark는 연산 그래프를 만들고, 액션(Action)이 호출될 때 실제 연산을 수행
- 이렇게 해서 Spark는 전체 데이터 플로우를 최적화

### Chaining:
Narrow Transformation은 하나의 스테이지에서 여러 개를 연결(chain) 가능









