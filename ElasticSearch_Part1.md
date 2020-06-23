# ElasticSearch Part1

## Elastic Stack Tutorial
### Tutorial 준비
* Ubuntu18.04, RAM 8GB이상의 계정 이름이 ec2-user인 시스템
* 튜토리얼 파일 설치
  * git clone https://github.com/yoonje/elastic-stack-tutorial.git
  * CentOS기준으로 작성되어 있어 Ubuntu에 맞게 수정 필요
  * shell script 수정
    * yum을 apt-get으로 모두 변경
    * npm error로 인해 head는 chrome 확장 프로그램으로 대체
      * head관련 코드 모두 삭제(npm포함)
    * java 설치 방법 변경
      * sudo apt-get install openjdk-8-jdk
	    * sudo sh -c "echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64' >> /etc/profile"
	    * sudo sh -c "echo 'export PATH=\$JAVA_HOME/bin:\$PATH"
      * sudo sh -c "echo 'export Class_PATH=\$JAVA_HOME/lib:\$CLASS_PATH"
### Tutorial 1, 2
* sh tuto 1 : Elasticsearch, Kibana, Filebeat 설정
* sh tuto 2 : Elasticsearch, Kibana, Filebeat 실행   
* 결과 확인
  * curl localhost:9200으로 ElasticSearch의 반응 확인
  * HEAD 반응 확인, filebeat 인덱스 생성 여부 확인
  * http://localhost:5601로 kibana 확인
### Tutorial 3 - Logstash 이용
* sh tuto 3, Hello Sehee 텍스트 입력
* `packages/logstash/bin/logstash -e 'input{stdin{}} output{stdout{}}'`을 통해 logstash가 stdin을 stdout으로 출력
### Tutorial 4 - Logstash 이용 2
* sh tuto 4, Hello Sehee 텍스트 입력
* `packages/logstash/bin/logstash -f logstash_conf/simple.conf를 통해 grok filter를 활용`
```
filter {
  grok {
    match => { "message" => "Hello %{WORD:name}" }
  }
}
```
* Hello 뒤에 나오는 이름에 name key를 매칭
* 결과 : "name" => "Sehee"
### Tutorial 5 - Elasticsearch에 데이터 저장
* 단일 인덱싱을 통해 데이터를 ES에 인덱싱
  * `curl -H 'Content-Type: application/json' -XPOST localhost:9200/firstindex/_doc -d '{ "mykey": "myvalue" }'`
* 벌크 인덱싱을 통해 데이터를 ES에 인덱싱
  * `curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/_bulk?pretty' --data-binary @logs.jsonl`
### Tutorial 6 - Kibana 활용   

## ElasticSearch VS Relational DB
* ElasticSearch
  * 키워드가 어떤 document에 있는지 저장
* RDB
  * 각 document에 대한 내용을 전부 따로 저장
* -> search시 ElasticSearch를 사용할 경우 더 빠름      

ElasticSearch -> Relational DB       
Index : Database   
Type : Table   
Document : Row   
Field : Column   
Mapping : Schema   
GET : Select   
PUT : Update   
POST : Insert   
DELETE : Delete   


