# Elastic STACK   
* 사용자가 서버나 데이터베이스로부터 원하는 데이터를 실시간으로 수집하고 검색, 분석하여 시각화 시키는 오픈소스 서비스   
* 기존의 ELK Solution (Elastic Search + Logstash + Kibana)에 Beats가 추가되면서 Elastic Stack이라는 이름으로 서비스를 제공

## Elastic Search
* Lucene 검색엔진 기반의 데이터베이스
* 고성능의 검색기능, 대규모 분산 시스템 기능 등 제공
* 표준 RESTful API와 JSON을 이용해 데이터 처리
* 정형, 비정형, 위치정보, 메트릭 등 원하는 방법으로 다양한 유형의 검색 수행하고 결합 가능
* 대량의 데이터를 신속하고 실시간으로 저장, 검색 및 분석 가능
  * Near Realtime   
    : 거의 실시간 검색 플랫폼
  * Cluster   
    : 전체 데이터를 보유하고 모든 노드에서 연합 인덱싱 및 검색 기능을 제공하는 하나 이상의 노드모음
  * Node   
    : 클러스터의 일부로 데이터를 저장하고 클러스터의 인덱싱 및 검색 기능에 참여하는 단일 서버
  * Index (RDB의 Database)   
    : 유사한 특성을 갖는 문서들의 집합
  * Type (RDB의 Table)   
    : Index내에서 하나 이상의 Type을 정의 가능
  * Document (RDB의 Row)   
    : Index를 생성할 수 있는 기본 정보 단위
    : JSON으로 표현
  * Shards   
    : shards를 이용하여 Index를 여러 조각으로 나눔 
    : 수평적으로 콘텐츠 볼륨을 split/scale 가능
    : 여러 노드에서 잠재적으로 분산을 통해 작업을 분산 및 병렬 처리를 할 수 있으므로 성능/처리량이 향상됨
  * Replication   
    : 장애가 발생할 경우 고가용성 제공
    : 복제본 샤드는 복사된 원본/기본 샤드와 동일한 노드에 할당되지 않음
    : 모든 복제본에서 검색을 병렬로 실행할 수 있기 때문에 검색 볼륨/처리량을 수평 확장 가능
## Logstash
* 데이터 수집 파이프라인 도구
* 각 데이터베이스의 데이터, 로우 데이터, 윈도우 이벤트 등으로부터 데이터 수집
* 데이터를 집계 및 보관, 서버 데이터 처리, 파이프라인으로 데이터를 수집하여 필터를 통해 변환 후 ElasticSearch로 전송
* 입출력 도구로 inpuut > filter > output 의 pipeline 구조로 구성되어 있다.
* input : Beats, CloudWatch, Eventlog 등의 다양한 입력을 지원하여 데이터를 수집
* filter : 형식이나 복잡성에 상관없이 설정을 통해 데이터를 동적으로 변환
* output : ElasticSearch, Email, ECS, Kafka 등 원하는 저장소에 데이터 전송

## Kibana
* 사용자 Application으로 Elastic Search에 저장된 정보들을 검색 및 분석하고 시각화하는 기능 제공

## Beats
* 경량 에이전트로 설치되어 데이터를 Logstash 또는 ElasticSearch로 전송
* Filebeat, Meticbeat, Packetbeat 등이 있음

