# 검색 구현
- 총 3가지 방법이 있을거라 생각
  - RDBMS(PostgreSql) Full-text search
  - NoSQL(MongoDB) Full-text search
  - Elastic Search

## RDBMS
- 무지성으로 rdbms에서 검색을 구현하면 `like` 이용하는데 이거 엄청 느리다고 함
- Mysql, Pg 모두 다양한 인덱싱 지원하는데 그중 하나가 full-text search
- Full-text search 란?
  - 단어 단위 검색이 아님
  - 텍스트 전체를 대상으로 인덱싱해놓고 검색 -> 단일 단어 뿐 아니라 여러 단어, 문장, 문단, 문서 이런 큰단위 검색도 가능
  - 또한 정확한 단어가 아니라 유사 단어라도 검색이 나올수 있음 -> 보다 넓은 범위의 검색 결과 제공
- RDBMS에서의 full-text search 특징
  - 각 레코드를 하나의 문서로 취급함 -> "title", "content" 등의 특정 컬럼을 검색 대상 지정 가능
  - SQL 쿼리 사용 -> 복잡한 검색조건 설정 가능
  - 검색 결과를 다른 테이블과 쉽게 조인하여 aggregate 가능

## NoSQL(MongoDB)
- text index 라는 텍스트 검색용 인덱싱 지원
- 컬렉션 내의 도큐먼트 전체를 검색 대상 지정
- 일부 필드만 지정한 검색도 가능(PG, Mongo 둘 다 복합 키 인덱스에 대한 full-text search 가능)

## Elastic Search
- 실시간 분산 검색 및 분석 엔진
- 검색에 최적화되어 가장 좋은 성능 가짐
- 데이터베이스로 사용하기 별로임
  - 트랜잭션 지원 X
  - 동기화 지연 존재
  - 쓰기 지연이 있어 터지면 다날라가므로 어차피 따로 저장할 데이터베이스 필요  
- 결국 어차피 todo list (history 등으로) 저장할 데이터베이스 또 만들거면 여기에 full-text search 를 쓰는게 좋아보임


## 개인적 결론
- Elastic Search 는 학습비용도 필요하고, 큰 데이터에나 장점이 있어보이고, 어차피 쓴다 하더라도 따로 DB가 필요하니 별로임
- 결국 RDBMS, NoSQL 중에 골라야 할 것 같음
- 우리 프로젝트의 비즈니스상 특징
  - todo list 는 다른 테이블과의 조인이 필요한지 여부
  - todo list 데이터가 자주 변경이 되는가의 여부
  - todo list 데이터 양
- 내생각에 todo list 는 잦은 update가 없고 crd만 있음, 다른 테이블과의 조인도 적어보임. NoSQL 사용이 가장 좋아보임

# AWS
- DB 기준으로 알아보겠음
- AWS 인프라 상에서 DB 사용방법
  - EC2 인스턴스 위에서 DB 설치 후 구성 -> 모든 책임이 개발자에게 있다
  - (MongoDB 기준)Atlas -> Cloud Native 한 데이터베이스, AWS 뿐 아니라 GCP, Azure 에도 연동 가능. 데이터베이스 구축 자동화가 잘 되어있다고 함
  - (RDBMS) RDS -> 아마존이 직접 만들어준 RDBMS. 다양한 RDBMS 지원
  - Aurora -> RDS 보다도 더욱 Mysql, PostgreSQL 지원 특화, 다양한 자동화 지원
  - Amazon Document DB -> 아마존이 직접 만들어준 도큐먼트 기반 NoSQL, 그렇다보니 AWS 다른 기능과 통합이 쉽다고 함
-  
## 개인적 결론
- EC2 인스턴스 위에서 띄우는건 커스텀 할 일 있는거 아니면 지양하는게 좋아보임
- Aurora + Document DB 두 데이터베이스를 사용하는게 좋아보임
- 그런데 데이터베이스 비용이 월 100% 사용 기준 한달 거의 최소 10만원인거같음 -> 비용부담이 클 것 같으면 하나의 데이터베이스에 스키마로만 분리해야 할 것 같음
