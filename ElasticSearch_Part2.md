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
* `curl -XPOST http://localhost:9200/classes/class/1/ -d '{ "title" : "Algorithm" , "professor":"John" }'`
  * classes : index명
  * class : type명
  * 1 : id
#### Create Index, Type, Document from file
* `curl -XPOST http://localhost:9200/classes/class/1/ -d @oneclass.json`
  * oneclass.json라는 파일을 사용해서 document 생성
  
## Update
#### Add one more field
* `curl -XPOST http://localhost:9200/classes/class/1/_update -d '{ "doc" : {"unit":1} }'`
#### Update one field
* `curl -XPOST http://localhost:9200/classes/class/1/_update -d '{ "doc" : {"unit":2} }'`
  * 추가할때와 같음
#### Update one field with Script
* `curl -XPOST http://localhost:9200/classes/class/1/_update -d '{ "script" : "ctx._source.unit += 5" }'`
  * unit에 5를 더해줌

## Bulk Post
* 여러개의 document를 한 번에 Elasticsearch에 삽입
* `curl -XPOST http://localhost:9200/_bulk?pretty --data-binary @classes.json`
  * classes.json : bulk로 document가 들어있음
  * bulk는 두 개의 라인으로 구성되어있음
    * meta information : index, type, id
    * document

## Mapping
* Mapping은 관계형 데이터 베이스에서의 schema와 동일
* mapping없이 elasticsearch에 데이터를 넣을 수 있지만 위험
* 처음 index를 생성하면 mapping 없음
* `curl -XPUT 'http://localhost:9200/classes/class/_mapping' -d @classesRating_mapping.json`
  * classesRating_mapping.json
  ```
  {
    "class":{
          "properties":{
                    "title":{"type":"string"},
                    "professor":{"type":"string"},
                    "major":{"type":"string"},
                    "student_count":{"type":"integer"},
                    "submit_date":{"type":"date","format":"yyyy-MM-dd"},
                    "school_location":{"type":"geo_point"}
            }
    }
  }
  ```

## Search
#### search
* `curl -XGET localhost:9200/basketball/record/_search?pretty`
  * 모든 document가 나옴
#### search - URI옵션
* `curl -XGET 'localhost:9200/basketball/record/_search?q=points:30&pretty'`
  * query는 points가 30인것만 search 하라는 것
#### search - request body 사용
* `curl -XGET 'localhost:9200/basketball/record/_search -d '{"query":{"term":{"points":30}}}'`
* request body는 여러가지 옵션 있음

## Aggregation(Metric)
* Aggregation은 Elasticsearch안에 있는 document안에서 조합을 통해서 어떠한 값을 도출할때 쓰이는 방법
* 그 중 Metric Aggregation은 산술할 때 쓰임
  * ex) 평균, 최대, 최소 구할때
* `curl -XGET localhost:9200/_search?pretty --data-binary @avg_points_aggs.json`
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
* `curl -XGET localhost:9200/_search?pretty --data-binary @terms_aggs.json`
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
