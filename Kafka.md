# Kafka
* Apache Kafka : 분산 스트리밍 플랫폼이며 데이터 파이프라인을 만들 때 주로 사용되는 오픈소스 솔루션
* 대용량의 실시간 로그처리에 특화되어 있는 솔루션
* 데이터를 유실없이 안전하게 전달하는 것이 주목적인 메세지 시스템에서 fault-tolerant한 안정적인 아키텍처와 빠른 퍼포먼스로 데이터 처리 가능
  
## Kafka 특징
* Publisher Subscriber 모델
  * Publisher Subscriber 모델은 데이터 큐를 중간에 두고 서로 간 독립적으로 데이터를 생산하고 소비함
  * Publisher나 Subscriber가 죽을 때 서로 의존성이 없으므로 안정적으로 데이터 처리 가능
* 고가용성 및 확장성
  * 카프카는 클러스터로서 작동하므로 fault-tolerant한 고가용성 서비스를 제공할 수 있고 분산 처리를 통해 빠른 데이터 처리를 가능하게 함
  * 서버를 수평적으로 늘려 안정성 및 성능 향상시키는 scale-out 가능
* 디스크 순차 저장 및 처리
  * 메세지를 디스크에 순차적으로 저장
    * 서버에 장애가 나도 메세지가 디스크에 저장되어있어 유실걱정 없음
    * 디스크에 순차적으로 저장되어 있어 디스크 I/O가 줄어들어 성능 향상
* 분산 처리
  * 카프카는 파티션 개념을 도입하여 여러개의 파티션을 서버에 분산시켜 나누어 처리 가능하여 메세지를 상황에 맞추어 빠르게 처리

## Kafka 구조
* 카프카 클러스터를 중심으로 producer와 consumer가 데이터를 push하고 pull하는 구조
#### Producer  
* 데이터를 발생시키고 카프카 클러스터에 적재하는 프로세스
* topic에 해당하는 메시지 생성
* 특정 topic으로 데이터를 publish
* 전송 실패할경우 재시도
* 카프카 클라이언트인 consumer와 producer를 사용하기 위해서는 아파치 카프카 라이브러리 추가
  * 브로커 버전과 클라이언트 버전의 호환성 확인해야함
    ```java
    public class Producer{
        public static void main(String[] args) throws IOException{
            Properties configs = new Properties(); // java properties 객체를 통해 프로듀서의 설정 정의
            configs.put("bootstrap.servers", "localhost:9092"); // 부트스트랩 서버 설정을 로컬 호스트의 카프카를 바라보도록 설정
            configs.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer"); // key :  메세지 보내면 토픽의 파티션이 지정될 때 쓰임
            configs.put("value.serializer", "org.apache.kafka.comon.serialization.StringSerializer");
            KafkaProducer<String, String> producer = new KafkaProducer<String, String> (configs); // 설정한 property로 카프카 프로듀서 인스턴스 생성
            ProducerRecord record = new ProducerRecord<String, String>("click_log", "login"); // ProducerRecord 객체를 생성할 때 어느 토픽에 넣을 것인지, 어떤 value와 key를 담을지 선언 가능
            // 여기서는 click_log라는 토픽에 login이라는 value
            producer.send(record); // 전송
            producer.close(); // 프로듀서 종료
        }
    }
    ```
#### Consumer Group
* consumer의 집합을 구성하는 단위
* 카프카 클러스터에서 데이터를 가져올때 consumer group 단위로 가져오며, consumer group 안의 consumer 만큼 파티션의 데이터를 분산처리함
#### Consumer
* Topic의 partition으로부터 데이터 polling
  * polling loop : poll() 메서드가 포함된 무한루프
* Partition offset 위치 기록 (commit)
  * offset : 토픽별로, 파티션별로 별개로 지정됨, consumer가 데이터를 어느지점까지 읽었는지 확인하는 용도
    * consumer가 데이터 읽기 시작하면 offset을 commit하게 됨
    * 이렇게 가져간 정보는 카프카의 __consumer_offset 토픽에 저장됨
    * consumer가 멈추더라도 재실행하면 중지되었던 시점을 알고있으므로 시작위치 복구 가능 -> 고가용성
* Consumer group을 통해 병렬 처리
  * consumer 개수는 partition의 개수보다 작거나 같아야함
  * 다른 consumer group에 속한 consumer들은 다른 consumer group에 영향 미치지 않음
    * 한 그룹이 각 파티션의 특정 offset을 읽고 있어도 다른 그룹이 읽는 데 영향 미치지 않음
    * __consumer_offset 토픽에는 consumer group별로, 토픽별로 offset을 나누어 저장하기 때문
    *  -> 하나의 토픽으로 들어온 데이터는 다양한 역할을 하는 consumer들이 각자 원하는 데이터로 처리 가능
#### Kafka Cluster 
* 카프카 서버로 이루어진 클러스터
* **Broker**
  * 카프카가 설치되어있는 서버 단위
  * 3개 이상의 브로커로 구성하여 사용하는것을 권장 
* **Zookeeper**
  * 분산 코디네이션 시스템으로 카프카 브로커를 하나의 클러스터로 코디네이팅
* **Topic**
  * 카프카 클러스터에 데이터를 관리할 때 그 기준이 되는 개념
  * 카프카 클러스터에서 여러개를 만들 수 있으며 하나의 토픽은 1개 이상의 파티션으로 구성되어 있음
* **Partition**
  * 각 토픽 당 데이터를 분산 처리하는 단위
  * 토픽 안에 파티션을 나누어 그 수대로 데이터를 분산 처리
  * 파티션들은 운영 중 수를 늘릴 수 있지만 줄일 수 없음
  * 데이터가 들어갈 때 키가 null이고 기본 파티셔너를 사용하면 round robin으로 할당되고, 키가 있고 기본 파티셔너를 사용하면 키의 hash값을 구해 특정 파티션에 할당됨
    * 카프카는 key를 특정한 hash값으로 변경시켜 파티션과 1대1 매칭 시킴
    * key값으로 전송하다가 파티션을 추가하면 키와 파티션의 매칭이 깨져 일관성이 보장되지 않음
      * 키를 사용할 경우 파티션을 생성하지 않는것을 추천
  * 파티션의 레코드가 저장되는 최대 시간과 크기 지정 가능
    * log.retention.ms : 최대 record 보존 시간
    * log.retention.byte : 최대 record 보존 크기(byte)
  * 카프카 옵션에서 지정한 replica의 수 만큼 파티션이 각 서버들에게 복제됨
    * 브로커 개수에 따라 replication의 개수가 제한
    * replication이 많아지면 그만큼 브로커의 리소스 사용량 늘어남
* **ISR(In Sync Replica)**
  * Leader partition : 각 파티션당 복제된 파티션 중에서 하나의 리더가 선출됨, 읽기 쓰기 연산 담당
    * producer의 ack를 통해 고가용성 유지
      * 0 : 프로듀서는 leader partition에 데이터 전송하고 응답값 받지 않음, 속도는 빠르지만 데이터 유실 가능
      * 1 : 프로듀서는 leader partition에 데이터 전송하고 응답값 받음, 나머지 파티션에 복사되었는지는 알 수 없음
      * all : 1 옵션에 추가로 follower partition에 복제가 잘 이루어졌는지 응답값 받음, 데이터 유실 없음
  * Folloer partition : 리더의 데이터를 복사함
  * 브로커 1개가 죽더라도 follower partition이 존재하면 복제본으로 복구 가능, leader partition 역할 승계

#### Lag
* **Kafka consumer lag**
  * 프로듀서가 최근에 넣은 데이터의 offset, 컨슈머가 가져간 데이터의 offset의 차이
  * lag의 숫자를 통해 현재 해당 토픽에 대해 파이프라인으로 연계되어있는 프로듀서와 컨슈머의 상태에 대해 유추 가능
  * 토픽에 여러 파티션 존재할 경우 lag은 여러개가 존재할 수 있음
  * 한 개의 토픽과 컨슈머 그룹에 대한 lag이 여러개 존재할 수 있을 때 그 중 높은 숫자의 lag을 records-lag-max라고 부름
* Burrow
  * 카프카의 consumer lag을 효과적으로 모니터링할 수 있도록 해줌
  * golang으로 작성된 오픈소스
  * 멀티 카프카 클러스터 지원
  * Sliding window를 통한 consumer의 status 확인
  * Http API 제공