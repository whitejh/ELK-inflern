# 섹션2. 키바나(Kibana)를 활용한 시각화

## 1. Kibana 시각화 준비하기
## Kibana를 사용한 시각화
- kibana는 Elasticsearch와 함께 작동(버전 같아야 함)하도록 설계된 오픈 소스 분석 및 시각화 플랫폼
-	kibana를 사용하여 Elasticsearch 색인(index)에 저장된 데이터를 검색,보기 및 상호작용
-	고급 데이터 분석을 쉽게 수행하고 다양한 차트, 테이블 및 맵에서 데이터를 시각화 
-	브라우저 기반의 인터페이스를 통해 실시간으로 Elasticsearch 쿼리의 변경 사항을 표시하는 동적 대시보드를 신속하게 만들고 공유

## Window에서 키바나 설치
- zip 디렉토리 레이아웃 (윈도우는 zip 패키지를 사용하여 설치하면 됨)
```
home 
bin 
config : kibana.yml파일 존재 (우분투에서는 /etc/kibana/.yml 에 위치)
data 
optimize 
plugins : 머신러닝 플러그인 등(그러나 요즘엔 머신러닝을 기본으로 탑제)
```
## Accessing Kibana
- 5601 포트번호를 통해 접근가능.
- https를 사용하기 때문에 인증서 오류가 일어날 수도 있음
- checking kibana status (kibana 서버의 상태 페이지에 접근)
  - localhost:5601/status 치면 상태화면이 나옴. 플러그인 상태도 나오고 나옴
  - 상태 페이지는 서버의 자원 사용에 대한 정보를 표시 
  - 설치된 플러그인을 나열
## Connect Kibana with Elasticsearch
- index에 어떤 패턴을 넣어줘야 ES에 의존적이기 때문에 index가 존재해야된다 
  -> filebeat로 수집한 데이터는 filebeat-어쩌구 이렇게 존재한다, 마찬가지로 로그스테시도 똑같다. 인덱스를 원하는 대로 정의할 수 있다.
- 실행중인 ES인스턴스에 연결됨. - 기본적으로 kibana는 그렇게 실행됨
## 키바나 실습과정
1. 우분투에서 sudo service elasticsearch start와 sudo service service kibana start로 엘라스틱서치와 키바나를 킨다.

2. 127.0.0.1:9200/_cat/indices?v 로 index를 확인
   
![1](https://user-images.githubusercontent.com/56130599/89035212-8721e000-d375-11ea-9ea0-747f5edd3d04.png)

index를 확인해보면 sample_data를 받은 것과 실습에서 진행한 데이터들이 존재함을 확인할 수 있다


3. kibana의 status와 많은 plugin들을 확인
   
![2](https://user-images.githubusercontent.com/56130599/89035963-ce5ca080-d376-11ea-9499-d6e5d109ac1b.png)

4. kibana에서 엘라스틱 서치의 127.0.0.1:9200/_cat/indices 처럼 index가 나오는 것을 확인
   
<img width="629" alt="3" src="https://user-images.githubusercontent.com/56130599/89036122-14196900-d377-11ea-9098-eee08d119cec.png">

5. kibana의 index pattern에서 확인후 엘라스틱 서치에 없는 pattern을 추가하는 실습
   
<img width="622" alt="4" src="https://user-images.githubusercontent.com/56130599/89036341-7f633b00-d377-11ea-8fbd-9d7767ce65dc.png">

create index pattern 후 1step에서는 어떤 index를 pattern으로 추가할 지 정할 수 있다

<img width="588" alt="1" src="https://user-images.githubusercontent.com/56130599/89097961-83966380-d41e-11ea-9e9f-c6c61d4f871b.png">

2step

<img width="576" alt="2" src="https://user-images.githubusercontent.com/56130599/89097994-c0625a80-d41e-11ea-87aa-62a3a1ce8250.png">

타임 시리즈 : 시계열    
데이터 분석 -> 시계열 데이터    
시간에 따라 분석을 할 수 있는 데이터 : 주식, 시세, 로그(몰리는 시간이 존재 등)   
tourcompany는 타임필터가 없기 때문에 세팅 안해도 된다.

<img width="540" alt="3" src="https://user-images.githubusercontent.com/56130599/89098024-facbf780-d41e-11ea-9fd2-039f2be15509.png">

.keyword는 문자열로 바꾼것 - 집계에서 사용 못하기 때문에 keyword로 바꿔야 된다.

<img width="565" alt="4" src="https://user-images.githubusercontent.com/56130599/89098060-336bd100-d41f-11ea-9e22-60f15cf10b32.png">

왼쪽 상단에 있는 나침반 모양을 클릭해 원하는 데이터 검색
위의 화면은 검색할 인덱스를 고르는 것

## 지금까지 무엇을 했는가? 
-> 데이터를 직접 만들어 낸 것을 index pattern에 정의한 것   
우리가 만드는 것들은 자동으로 index pattern에 등록 되지 않는다.   
앞으로 로그를 수집해서 엘라스틱 서치에 데이터가 올라오면 가장 먼저 해야할 일은    
index pattern을 정의하고 그 다음을 잘 조회되는지 확인한다.   

## 2. 시각화할 데이터 매핑하기
- 엘라스틱 서치에 샘플 데이터 세트 로드
- 인덱스 패턴 정의
- Discover를 사용하여 샘플 데이터 검색
- 샘플 데이터의 시각화 설정
- 시각화를 대시 보드로 어셈블

PUT을 통해 매핑한다.
매핑하기 위해선 JSON 데이터를 넘겨줘야한다.

## Loading Sample Data
- 쉐익스피어 데이터 작품 - sharkspeare.json
- 무작위로 생성된 데이터로 구성된 가상 계정 집합 - accounts.json
- 임의로 생성된 로그 파일 세트

데이터를 로드하기 전에 매핑을 먼저 수행해야된다 -> 데이터 형을 지정해야됨   
데이터형을 지정해야하는 이유   
: 지도의 자표 등은 모양새가 float형이랑 비슷   
mapping 없이 진행하면 float형으로 진행되어서 나중에 지도 표현할 때 geopoint를 찾지 못한다.   

- 벌크데이터를 사용하여 엘라스틱 서치에 데이터 세트를 로드   
예시 :    
![5](https://user-images.githubusercontent.com/56130599/89098284-133d1180-d421-11ea-9c9c-55d174da7e71.PNG)

## 쉐익스피어 데이터 실습, 매핑
https://www.elastic.co/guide/en/kibana/current/tutorial-build-dashboard.html를 참고하여 실습진행

![6](https://user-images.githubusercontent.com/56130599/89098297-3b2c7500-d421-11ea-9418-0035a96e68ce.png)

status가 201인지 확인해야한다. 만약 400이 나오면 문제가 있는 것이다.

![7](https://user-images.githubusercontent.com/56130599/89098324-74fd7b80-d421-11ea-9af4-2b330e4b28f7.png)

키바나의 index management에서 logstash 개와 shakespeare가 다 올라온 것을 볼 수 있다. 
yellow는 쉐딩이 아직 안 끝난 상태. 끝나면 green으로 변한다.

![8](https://user-images.githubusercontent.com/56130599/89098371-e63d2e80-d421-11ea-8411-dd45b26bbcf4.png)

logstash index pattern을 만들 때 timefield를 지정할 때 설명
시계열 데이터는 @로 표현
@timestamp : logstash가 수집한 시간
utc_time : log가 생긴 시간 -> 좀 더 서버를 신뢰할 수 있는 데이터

일반적으로 timestamp(logstash가 수집한 시간)와 utc_time(로그가 생성된 시간)은 일치 
만약 es가 장시간 죽었다가 다시 살아나면?
-> 그동안 올라가지 못했던 로그들이 한꺼번에 올라가면서 @timestamp가 그 시간대로 찍힌다. 많은 시간들이 찍힌 것처럼 표현됨 

<img width="457" alt="9" src="https://user-images.githubusercontent.com/56130599/89098424-74b1b000-d422-11ea-80e9-db621d551121.png">

geo_cooridinates에 geo_point 데이터가 반드시 있어야한다. 없으면 mapping이 잘못된 것이고 coodinate map을 그릴 수 없다.

## 3. kibana를 활용한 시각화
New Visualization 메뉴
- metric : 그림을 그리기 전에 숫자를 계산하는 것
- pie : 통계 쪽에서 가장 많이 쓰이는 기본적인 것

![image](https://user-images.githubusercontent.com/56130599/89098497-3668c080-d423-11ea-8a6a-b38dad833cb5.png)

![image](https://user-images.githubusercontent.com/56130599/89098509-4aacbd80-d423-11ea-82ec-81c061bf9c70.png)

로그 발생량할 때 Metric으로 횟수 count 가능

![image](https://user-images.githubusercontent.com/56130599/89098521-67e18c00-d423-11ea-83ff-10c77c4f99c8.png)

생성된 Metric

![image](https://user-images.githubusercontent.com/56130599/89098525-75971180-d423-11ea-85cd-27370d464712.png)

dashboard에 그 차트들을 추가할 수 있다. 한 번에 관찰 가능!

![image](https://user-images.githubusercontent.com/56130599/89098536-91021c80-d423-11ea-85e6-945a88257c9c.png)

통계 쪽에서 유용한 pie 차트. split slices를 통해 원을 나눌 수 있다.  

![image](https://user-images.githubusercontent.com/56130599/89098550-a8410a00-d423-11ea-87be-36f6f71ce4e5.png)

계좌의 잔액으로 split한 결과

![image](https://user-images.githubusercontent.com/56130599/89098558-b68f2600-d423-11ea-93e6-0f6330c1534e.png)

state 기준으로 두 번 나눌 수 있다. 어느 주에 있는 사람들이 돈이 많은지 분석 

![image](https://user-images.githubusercontent.com/56130599/89098579-d6bee500-d423-11ea-9ab2-371eb787c546.png)

x축 y축 등 bar를 실습

![image](https://user-images.githubusercontent.com/56130599/89098592-f7873a80-d423-11ea-9e12-2554ab637fc7.png)

logstash 로그를 불러와서 coordinate정보를 좌표로 찍는다. discover에서 데이터가 최근 15분꺼까지만 보게 설정되어있어서 바꿔줘야한다.   
2015년 데이터로 적용시키고 확인하면 좌표가 생긴다.

- 최종 dashboard 생성
![10](https://user-images.githubusercontent.com/56130599/89098492-25b84a80-d423-11ea-8d4e-003b0055513d.png)

만들어진 시각화도표로 dashboard를 마음대로 꾸밀 수 있다.
