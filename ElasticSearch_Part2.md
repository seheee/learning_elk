# ElasticSearch Part2

## Create, Read, Delete
#### Verify Index
* `curl -XGET http://localhost:9200/classes`
  * classes가 index name, classes라는 인덱스가 있는지 확인함
  * "status":404 인 경우 현재 classes라는 인덱스가 elasticsearch에 존재하지 않는것
* `curl -XGET http://localhost:9200/classes?pretty`
  * ?pretty를 파라미터로 넣어주면 결과를 깔끔하게 볼 수 있음
#### Create Index
* `curl -XPUT http://localhost:9200/classes`
  * classes라는 인덱스를 생성
#### Delete Index
* `curl -XDELETE http://localhost:9200/classes`
  * classes라는 인덱스를 삭제
#### Create Document
* index가 있을때 만들 수 있고, index가 없을 때도 index명과 type명을 명시하면 바로 document 생성 가능
* `curl -XPOST http://localhost:9200/classes/class/1/ -H 'Content-Type: application/json' -d '{ "title" : "Algorithm" , "professor":"John" }'`
  * classes : index명
  * class : type명
  * 1 : id
  * -d 옵션 : HTTP Post data
  * 헤더에 타입 없으면 error -> `-H 'Content-Type: application/json'` 추가
#### Create Index, Type, Document from file
* `curl -XPOST http://localhost:9200/classes/class/1/ -d @oneclass.json -H 'Content-Type: application/json'`
  * oneclass.json라는 파일을 사용해서 document 생성
  
## Update
* 입력된 document를 수정하기 위해서는 기존 document의 URL에 변경될 내용을 다시 PUT하는 것으로 대치 가능   
  하지만 필드가 여럿 있을때 하나의 필드만 바꾸기 위해 전체 내용을 다시 입력하는것은 번거로움   
  이 때 POST를 이용해 원하는 필드의 내용만 업데이트 가능
* 업데이트 할 내용에 "doc"이라는 지정자 사용
* _update API를 사용해서 단일 필드만 수정하는 경우에 실제로 내부에서는 document 전체 내용을 가져와서 doc에서 지정한 내용을 변경한 새 document를 만든 뒤 전체 내용을 다시 PUT으로 입력하는 작업 진행 -> 업데이트 후 버전이 증가한 것 확인 가능
#### Add one more field
* `curl -XPOST http://localhost:9200/classes/class/1/_update -d '{ "doc" : {"unit":1} }' -H 'Content-Type: application/json'`
#### Update one field
* `curl -XPOST http://localhost:9200/classes/class/1/_update -d '{ "doc" : {"unit":2} }' -H 'Content-Type: application/json'`
  * 추가할때와 같음
#### Update one field with Script
* `curl -XPOST http://localhost:9200/classes/class/1/_update -d '{ "script" : "ctx._source.unit += 5" }' -H 'Content-Type: application/json'`
  * unit에 5를 더해줌

## Bulk Post
* 여러개의 document를 한 번에 Elasticsearch에 삽입
* bulk는 두 개의 라인으로 구성되어있음
  * meta information : index, type, id
  * document
* bulk 동작은 따로따로 수행하는 것 보다 속도가 훨씬 빠름
* 특히 대량의 데이터를 입력 할 때는 bulk API를 사용해야 불필요한 오버헤드가 없음
* `curl -XPOST http://localhost:9200/_bulk?pretty --data-binary @classes.json -H 'Content-Type: application/json'`
  * classes.json : bulk로 document가 들어있음

   

## Mapping
* Mapping은 관계형 데이터 베이스에서의 schema와 동일
* mapping없이 elasticsearch에 데이터를 넣을 수 있지만 위험
* 처음 index를 생성하면 mapping 없음
* mapping 타입 중 string을 삭제하고 text로 변경됨
* 
* `curl -XPUT 'http://localhost:9200/classes/class/_mapping?include_type_name=true' -d @classesRating_mapping.json -H 'Content-Type:application/json'`
* ex) classesRating_mapping.json
  ```
  {
    "class":{
          "properties":{
                    "title":{"type":"text"},
                    "professor":{"type":"text"},
                    "major":{"type":"text"},
                    "student_count":{"type":"integer"},
                    "submit_date":{"type":"date","format":"yyyy-MM-dd"},
                    "school_location":{"type":"geo_point"}
            }
    }
  }
  ```

## Search
#### search
* 쿼리를 통한 검색 기능
* 검색은 인덱스 단위로 이루어짐
* 쿼리를 입력하지 않으면 전체 도큐먼트를 찾는 match_all검색을 함
* `curl -XGET localhost:9200/basketball/record/_search?pretty`
  * 모든 document가 나옴
#### search - URI검색
* _search뒤에 q파라미터를 사용해서 검색어 입력
* 요청주소에 검색어를 넣어 검색하는 방식
* `curl -XGET 'localhost:9200/basketball/record/_search?q=points:30&pretty'`
  * query는 points가 30인것만 search 하라는 것
#### search - request body 사용
* 검색 쿼리를 데이터 본문으로 입력하는 방식
* `curl -XGET 'localhost:9200/basketball/record/_search -d '{"query":{"term":{"points":30}}}' -H 'Content-Type:application/json'`
* request body는 여러가지 옵션 있음
* Term 쿼리
  * 입력한 검색어는 애널라이저를 적용하지 않고 입력된 검색어 그대로 일치하는 텀을 찾음
  * Full Text Search
    * Elasticsearch는 데이터를 실제로 검색에 사용되는 term으로 분석 과정을 거쳐 저장하기 때문에 검색 시 대소문자, 단수나 복수, 원형 여부와 상관없이 검색 가능함
   
 
## Aggregation(Metric)
* Aggregation은 Elasticsearch안에 있는 document안에서 조합을 통해서 어떠한 값을 도출할때 쓰이는 방법
* 그 중 Metric Aggregation은 산술할 때 쓰임
  * ex) 평균, 최대, 최소 구할때
* `curl -XGET localhost:9200/_search?pretty --data-binary @avg_points_aggs.json -H 'Content-Type:application/json'`
#### Aggregations structure
```
"aggregations" : {
    "<aggregation_name>" : {
      "<aggregation_type>" : {
        <aggregation_body>
    }
    [,"meta" : { [<meta_data_body>]} ]?
    [,"aggregations" : { [<sub_aggregation>]+ } ]?
  }
  [,"<aggregation_name_2>" : { ... } ]*
}
```
ex)
```
{
  "size" : 0,
  "aggs" : {
      "avg_score" : {
          "avg" : {
              "field" : "points"
          }
      }
  }
}
```
* aggs = aggregations
* 원하는 aggregation 정보만 보기 위해 size를 0으로 함
* field값 중에서 points를 사용해서 평균을 구함
* avg 이외에도 max, min, sum, stats 등 가능
  * stats를 사용하면 max, min, avg, sum 모두 출력 가능

## Aggregation(Bucket)
* Bucket Aggregation은 group by로 볼 수 있음
* `curl -XGET localhost:9200/_search?pretty --data-binary @terms_aggs.json -H 'Content-Type:application/json'`
#### term aggregations
ex)
```
{
  "size" : 0,
  "aggs" : {
      "players" : {
          "terms" : {
              "field" : "team"
          }
      }
  }
}
```
* team별로 document를 묶기 위해 field를 team으로 정의
* bucket aggregation은 sub aggregation을 포함할 수 있음   

ex)
```
{
  "size" : 0,
  "aggs" : {
      "team_stats" : {
          "terms" : {
              "field" : "team"
          },
          "aggs" : {
              "stats_score" : {
                "stats" : {
                    "field" : "points"
                }
              }
          }
      }
  }
}
```
* term aggregation을 사용해서 team별로 document를 묶음
* team별로 묶인 document들에서 stats (metric aggregation)
