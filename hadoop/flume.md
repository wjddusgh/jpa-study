# 14. 플룸(Flume)
### 아파치 플룸
- 로그 또는 이벤트 데이터를 중앙 위치로 전송하는데 사용되는 데이터 수집 도구
- fluentd, filebeat, logstash 등이 대표적인 데이터 수집 도구
- 아파치 플룸은 주로 Hadoop 및 HDFS 생태계와 통합
### 이름의 유래
- 공식문서에 명시되어있지는 않음
- 놀이기구중 후룸라이드 처럼 flume 에 물같은 액체를 전달하는 인공적인 홈 이라는 의미(잘 어울림)

### 플룸의 구조
![flumeImg](https://github.com/wjddusgh/jpa-study/assets/69251780/a0988513-c16e-467f-8505-29563c816b53)
- 구조
  - Agent: 소스, 채널, 그리고 싱크로 구성되며, Flume의 기본 실행 단위, 이벤트는 소스에서 수집되어 채널을 거쳐 싱크로 전송됨
  - Event: Flume을 통해 전송되는 데이터의 기본 단위
  - Source: 데이터를 수집하고 이벤트로 변환하는 Flume의 구성 요소, 로그 파일이나 메시징 큐에서 데이터를 읽어옴
  - Channel: 소스에서 받은 이벤트를 일시적으로 저장하고, 싱크로 전송될 때까지 보존하는 중간 저장소 역할, 메모리나 파일시스템 사용
  - Sink: 채널에서 이벤트를 소비하고 외부 저장소나 서비스로 전송하는 구성 요소, `HDFS, HBase, Solr` 등이 있음
- 플룸 사용의 핵심: 개별 요소가 함께 유기적으로 동작할 수 있도록 설정하는것

### 예제
```
a1.sources = source1
a1.sinks = sink1
a1.channels = c1

a1.sources.source1.channels = c1
a1.sinks.sink1.channel = c1

a1.sources.source1.type = spooldir
a1.sources.source1.spoolDir = /opt/homebrew/Cellar/flume/1.11.0/libexec/conf/spooldir

a1.sinks.sink1.type = logger

a1.channels.c1.type = file
```
- Spooling Directory: Flume 에서 제공하는 소스 유형중 하나, 특정 디렉토리에 저장된 파일들을 주시하며, 새로 생성되는 파일을 자동으로 읽어들임
  - 파일 업데이트는 감지 X
- `flume-ng agent -c $FLUME_HOME/conf -f flume.conf -n a1 -Dflume.root.logger=INFO,console`
  - -f: --conf-file
  - -n: --name
  - -c: --conf
- name 옵션을 통해 하나의 conf-file 에 여러 agent 만들어놓고, -n 으로 구분하여 사용 가능

### 트랜잭션과 신뢰성
- 플룸은 `소스 -> 채널`, `채널 -> 싱크` 를 각각 트랜잭션으로 분리
- `소스 -> 채널`
  - 성공시 파일에 `.COMPLETED` 표시
- `채널 -> 싱크`
  - 오류 발생시 트랜잭션 롤백, 해당 이벤트는 나중에 다시 전송할 수 있도록 채널에 남게 됨
- 채널
  - 파일 채널 : 파일 시스템 사용, 처리속도는 메모리 채널에 비해 느리지만 에이전트가 재시작 하더라도 이벤트는 유실되지 않음
  - 메모리 채널 : 메모리 사용, 처리속도는 파일 채널에 비해 빠르지만 에이전트 재시작시 이벤트 유실
- At Least Once(파일 채널 사용시)
  - 적어도 한번 이상 도착 -> 중복 가능성 존재
  - 2PC(2단계 커밋, 준비 -> 커밋 or 롤백) 고비용
- 효율성을 위해 플룸은 각 트랜잭션에서 이벤트를 한 건씩 처리하기보다는 배치 사용
### 파티셔닝과 인터셉터
- HDFS에서 플룸 이벤트 데이터는 주로 시간을 기준으로 파티셔닝 됨
- 데이터 저장시 바이너리 포맷이 텍스트 파일보다 크기가 작아짐
- 사용하는 소스에 따라 설정이 달라짐

### 분기
- 분기: 하나의 소스에서 발생하는 이벤트를 여러 개의 채널로 전송하는 것
- `a1.sources.source1.channels = c1 c2` 처럼 여러 채널을 소스에 설정 가능
  ![Fan-out-Flow-1](https://github.com/wjddusgh/jpa-study/assets/69251780/a7f0818e-1401-41c2-9d51-cad768322210)
- 분기된 각 채널마다 각각 다른 트랜잭션 사용
  - `source.selector.optional` 사용 시 일부 채널은 트랜잭션이 실패해도 재전송 안하게끔 설정 가능
- 전체 분기가 아닌 선택적 분기는 다중화 선택기 사용시 가능, 자세한 설정은 책에 없음

### 분배: 에이전트 계층
- 플룸은 소스, 채널, 싱크 라는 독립적인 모듈의 집합이므로, 한 플룸 에이전트의 싱크는 또다른 플룸의 소스가 될 수 있다. (에이브로, 쓰리프트 등의 RPC를 사용)

![consolidation](https://github.com/wjddusgh/jpa-study/assets/69251780/ae86e508-332f-4215-96d8-5288318467fc)
- 깊은 계층의 에이전트 장애는 심각하다

### 싱크 그룹
- `agent1.sinkgroups` 통해 싱크그룹 설정
- 위에서의 에이전트 장애를 복구, 부하를 분산 시키기 위해 사용

### 애플리케이션 내에서 플룸 사용
- `Embedded Agent` 사용 [참고](https://flume.apache.org/releases/content/1.5.0.1/apidocs/org/apache/flume/agent/embedded/EmbeddedAgent.html)


 

