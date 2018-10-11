---
layout: post
title: ElasticSearch Backup/Restore
excerpt: "ElasticSearch를 처음에 접하게 되면 막막한 부분이 있는데, 필자 역시 막막했던 것들 중 한 부분을 공유하고자 한다."
categories: [bigdata]
tags: [ElasticSearch, 색인, bigdata, 빅데이터]
comments: true
#image:
  #feature: es-logo.gif
---


![ElasticSearch]({{ site.url }}/img/es-logo.gif)

### 1. 개요

ElasticSearch를 처음에 접하게 되면 막막한 부분이 있는데, 필자 역시 막막했던 것들 중 한 부분을 공유하고자 한다. 

**ElasticSearch 백업/복구** : 어떤식으로 진행을 해야할지 간단하게 알아보자. '백업'과 '복구' 작업을 하기위해 먼저 선행되어야 할 작업이 있다. 아래 내용을 먼저 살펴보자. 

> 백업 : 백업경로설정 -> 설정파일에 경로추가 -> 서비스 재시작 -> 저장소 설정 -> 백업

> 복구 : 복구경로설정 -> 설정파일에 경로추가 -> 서비스 재시작 -> 저장소 설정 -> 복구파일 이동 -> 복구

선행되어야 할 작업은 백업과 복구 모두 동일하다. 따라서 백업하는 방법을 알면 자연스럽게 복구하는 법을 익힐 수 있을 것이라 본다.








#### 2. 저장소로 사용할 백업 경로 설정

**백업을 위해 스냅샷을 저장하고 복구하기 위해 스냅샷을 로드하는 경로**를 별도로 설정해 주어야 하는데, ElasticSearch에서는 이 경로를 저장소로 사용할 것이다. (!!! 만약 여러대의 서버 노드로 되어있다면, 서버 공유 디렉토리를 추천한다. !!!)
저장소를 설정 해 주기 전 ElasticSearch가 경로를 인식할 수 있도록  *'config/elasticsearch.yml'* 파일에 아래의 내용을 추가해 주고 ElasticSearch를 재시작 해주면 되는데 '노드가 여러개' 이거나 '대용량의 색인 데이터'의 경우 *'Rolling Restarts'* 가 필요할 수 있으니, 재시작을 하기 전에 3번에서 알아보자.

> path.repo:["{저장_경로}"]
> *- 경로는 절대경로*



#### 3. 재배치(Reallocation) 하지 않고 재시작하기, Rolling Restarts

ElasticSearch를 처음 접하다 보면 저게 또 무슨 말인가 할 수 있다. 이 [블로그][rollingRestartLink]를 훌터보자. 필자보다 훨씬 좋은 그림과 필력으로 잘 설명되어 있다. 핵심만 말하자면 ElasticSearch의 색인된 데이터는 N개의 노드에 임의로 분배하여 배치되어 있는데, 설정된 Shard(샤드) 수 만큼 분산저장되어 있다. 그리고 다수의 노드일 경우 한 노드가 내려가게 되면 그 노드에 저장되어있는 Shard를 검색 할 수 없기 때문에 N-1개의 노드로 재 배치를 수행하게 된다.

이러한 작업은 많은 리소스를 발생시키고 운영중이 서버에서는 위험부담이 있으므로 재 배치를 하지 않도록 먼저 해당 노드에서 아래와 같은 쿼리를 먼저 실행하자.

> ```json
> # 재배치 중지
> $ curl -XPUT 'localhost:9200/_cluster/settings?pretty=true' -d
> {
>     "transient" : {
>         "cluster.routing.allocation.enable" : "none"
>     }
> }
> ```



#### 4. 저장소 만들기

2번과 3번 과정을 잘 읽었다면 이제 ElasticSearch 서비스를 재시작 하자. 정상적으로 서비스가 실행되었다면 아래 쿼리를 실행하여 저장소를 만들어보자.

> ```json
> # 저장소 생성
> $ curl -XPUT 'localhost:9200/_snapshot/{저장소_이름}' -d
> {
>   	"type":"fs",
>   	"settings":{
>       	"compress":true,
>       	"location":"{저장소_경로}"
>   	}
> }
> ```

Rolling Restarts 위해 3번과정을 진행했다면 아래 쿼리를 실행하여 다시 배치된 샤드와 연결해 주도록 하자.

> ```json
> # 재배치 시작
> $ curl -XPUT 'localhost:9200/_cluster/settings?pretty=true' -d
> {
>     "transient" : {
>         "cluster.routing.allocation.enable" : "all"
>     }
> }
> ```
>
> 필자의 경우 재시작 후 ElasticSearch의 상태가 yellow에서 위 쿼리 실행 후 green으로 바뀌었다.

다음 단계로 가기전 저장소가 잘 만들어졌는지 아래 URL을 통해 알아보자

> ```json
> # 저장소 확인
> $ curl -XGET 'localhost:9200/_snapshot/{저장소_이름}'
> ```



#### 5. 스냅샷 만들기(백업하기)

스냅샷을 만들기 전 현재 어떤 색인 데이터가 있는지 아래 URL을 통해 리스트를 확인해 보자.

> ```json
> # 색인 데이터 리스트 보기
> $ curl -XGET 'localhost:9200/_cat/indices'
> ```

스냅샷을 만들 대상이 리스트에서 확인이 되었다면 아래 쿼리를 실행하여 스냅샷을 만들어보자.

> ```Json
> # 스냅샷 만들기(백업하기)
> $ curl -XPUT 'localhost:9200/_snapshot/{저장소_이름}/{스냅샷_이름}?wait_for_completion=true' -d
> {
>   	"indices": "${대상_색인_이름}",
>   	"ignore_unavailable": true,
>   	"include_global_state": true
> }
> ```

**!!! 여기서 만약 여러개의 노드를 사용하는 경우 !!!**

위의 명령어가 정상적으로 실행되지 않고 에러결과를 리턴받을 수 있다. 필자의 경우는 **서버간에 공유디렉토리가 저장소로 설정되어있지 않아서 발생하는 에러**였다. 노드가 여러개라면 각 노드별로 설정된 Shard 수 만큼 임의로 배치가 되어있을 것이다. 스냅샷 데이터를 저장하는데 각각의 노드 데이터를 한 저장소에 한꺼번에 저장하는 것이 default로 설정되어 있는 것 같다.
**방법**은 두 가지이다. 하나는 **각 노드 서버의 공유 디렉토리를 생성**하는 것이고, 다른 하나는 **아래 명령어를 통해 샤드를 각자 노드의 설정된 디렉토리에 저장**할 수 있다.

> ```json
> # 저장소 확인하지않기(공유 디렉토리 무시하기)
> $ curl -XPUT 'localhost:9200/_snapshot/{저장소_이름}?verify=false
> {
>   	"type":"fs",
>   	"settings":{
>     	"compress":true,
>       	"location":"{저장소_경로}"
>   	}
> }
> ```

위의 request는 서버간 공유 디렉토리가 아니더라도 스냅샷을 만들도록 해주는 것 같다.
(공유 디렉토리를 이용하여 백업하는 것이 아니기 때문에 데이터가 유실 될 수 있다.)

정상 수행이 되었다면 다시 스냅샷을 생성하는 쿼리를 수행해보자. 정상적으로 스냅샷이 생성되었다면, 노드(또는 각 노드별) 저장소에 샤드가 생성되어 있을 것이다. 만약 여러개의 노드 환경에서 각 서버별 디렉토리로 스냅샷을 생성하도록 했다면, scp명령어를 통해 한 디렉토리에 모아두자.

자 이제 디렉토리를 tar나 zip형태로 압축하여 보관하자.



#### 6. 스냅샷 확인 및 복원

앞서 1번 개요에서 말했듯 백업과 복구를 시작하기전 선행되는 단계가 동일하다. 따라서 다른 과정은 생략하고 바로 복구 대상 파일인 스냅샷을 확인하고 복원하는 방법을 알아보자.

> ```json
> # 스냅샷 확인하기
> $ curl -XGET 'localhost:9200/_snapshot/{저장소_이름}/{스냅샷_이름}/_status'
> ```

위의 request로 스냅샷을 확인하기 위해선 저장소에 스냅샷 파일이 있어야 한다.

정상적으로 response를 받았다면 아래 쿼리를 수행하여 복원을 진행하도록 하자.

> ```json
> # 스냅샷으로 복구하기
> $ curl -XPOST 'localhost:9200/_snapshot/{저장소_이름}/{스냅샷_이름}/_restore' -d
> {
>   	"indices":"{기존_인덱스_이름}",
>   	"index_settings":{
>       	"index.number_of_replicas":0
>   	},
>   	"ignore_index_settings":[
>       	"index.refresh_interval"
>     ]
> }
> ```

복원에 대한 response가 정상이라면 실제 색인 데이터를 확인한다.



#### 7. 마무리

이번에 공유하게 된 'ElasticSearch Backup/Restore' 방법은 필자와 같이 막막한 분들에게 조금이라도 도움이 되길바라는 마음에 작성하게 되었다. 또한 위의 방법대로 해도 되지 않을 수 있다는 것을 염두해 두길 바란다. 단지 경험을 공유하기 위한 글이다.



끝.





**엘라스틱서치 덤프관련 사이트**

http://www.kwangsiklee.com/ko/2017/02/%EC%97%98%EB%9D%BC%EC%8A%A4%ED%8B%B1-%EC%84%9C%EC%B9%98-%EC%8A%A4%EB%83%85%EC%83%B7-%EC%A0%80%EC%9E%A5-%EB%B3%B5%EC%9B%90%ED%95%98%EA%B8%B0/

http://asuraiv.blogspot.kr/2015/04/elasticsearch-rolling-restarts.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-snapshots.html#_shared_file_system_repository

https://www.elastic.co/guide/en/elasticsearch/guide/current/_rolling_restarts.html

[rollingRestartLink]: http://asuraiv.blogspot.kr/2015/04/elasticsearch-rolling-restarts.html

