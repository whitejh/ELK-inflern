# Logstash를 활용한 로그 통합
## 1. 로그스태시 파일비트 개요

![image](https://user-images.githubusercontent.com/56130599/89098680-a0359a00-d424-11ea-87a7-ce36a166c5b5.png)   

로그스태시는 엘라스틱서치와 가장 좋은 호환을 보인다

```txt
dats Source => input>Filters>OUTPUTS => ES
               --LOGSTASH Pipeline--
```
문법과 플러그인을 잘아야 잘 사용 가능!

## 로그 스태시의 입력

- input : filebeat, rabbit 등 수집 프로그램으로 수집 경로 지정하고 포트만 지정해주면 그곳에서 데이터를 가져옴.
- filter 종류 : GROK, GEO IP, FINGERPRINT, DATA 등 각 필터를 통해 Field를 구문을 분석 및 변환해줌.
- OUTPUT: ES,메일 등 데이터 베이스로 저장이 가능! 유연성을 확보할 수 있다.

## 파이프라인 이미지

![image](https://user-images.githubusercontent.com/56130599/89098732-176b2e00-d425-11ea-9f75-b7b26fe230fb.png)

## 플러그인들 집합

![image](https://user-images.githubusercontent.com/56130599/89098754-3ec1fb00-d425-11ea-9230-51ce313c0f3f.png)   

각 부분에 대한 수많은 필터들

## 각부분에 대한 설명

- input: 로그스태시로 데이터를 가져옴
  - file: 파일시스템의 파일에서 읽음
  - syslog: syslog 메세지 및 구문 분석을 위해 포트 514를 수신
  - redis: 데이터베이스
  - beat: Filebaet에서 보낸 이벤트를 처리
  
- Filter: 로그스태시 파이프 라인의 중간 처리
  - grok: 구조화 되지 않은 로그 데이터를 구문 분석
  - mutate: 일반적인 변환 수행 - 이벤트의 데이터 수정 및 제거
  - drop : 이벤트 삭제
  - clone : 이벤트 복사
  - geoip : IP 주소의 지리적 위치에 대한 정보를 추가(키바나의 지도 차트에서 사용됨)
  
- OUTPUT: 최종 단계 - 여러출력 사용 가능
  - ES: ES에 데이터 전송
  - file: 이벤트 데이터를 디스크 파일로 저장
  - graphite: 이벤트데이터를 Graphite에 전송
  - statsd: statsd에 이벤트 데이터를 전송
  
- Codec: 스트림필터
  - json
  - multilne

## input: file

![image](https://user-images.githubusercontent.com/56130599/89098793-9e200b00-d425-11ea-8ad0-2a12936206b4.png)  

path에서 부터 데이터를 읽어온다.
beginning은 처음부터, end는 끝에서부터 읽는다.

## input: beats

![image](https://user-images.githubusercontent.com/56130599/89098805-b3953500-d425-11ea-8f56-128b60697f54.png)

여러개의 filebeat를 하나의 logstash로 수집 5044포트를 열고 대기함

**파일 비트가 설치된 컴퓨터는 로그스테시에 전달**하는 그런 일 (이거 우리가 계획할 것)

## filter: grok
![image](https://user-images.githubusercontent.com/56130599/89098852-07a01980-d426-11ea-8407-ab6509bfb618.png)  

원하는 형식으로 바꿔줄 수 있음

## filter: geoip
![image](https://user-images.githubusercontent.com/56130599/89098873-2e5e5000-d426-11ea-8078-0e7ee90edf93.png)  
ip를 가져와서 source를 가져와서 (좌표가 나오고 저장)

## output:elasticsearch
![image](https://user-images.githubusercontent.com/56130599/89098878-3b7b3f00-d426-11ea-9b9f-e0fe28bb8103.png)   
HTTP프로토콜로 알아서 전송   
hosts를 분산형태이기 때문에 여러곳으로 보낼 수 있음.   
template제공 - 기본적인 매핑을 지원하지만 원하지 않으면 false로 셋팅하기 셋팅되면 로그의 인덱스가 logstash-%{YYYY.MM.DD}로 저장이 됨   
원하는 index이름을 지정

## filebeat
굉장히 쉬운 모듈 경량화 하여 만든 수집기임.   
filebeat는 중앙 집중화된 작업을 간편하게 유지해 줌.   
대시보드도 이미 구현된 것들이 있음.

## 2. 로그스태시, 파일비트 설치와 실행
- 설치파일은 usr/share에, 설정 파일은 /etc에 있음!

![image](https://user-images.githubusercontent.com/56130599/89098911-85fcbb80-d426-11ea-809f-5d57d88ff1ba.png)

![image](https://user-images.githubusercontent.com/56130599/89098917-8eed8d00-d426-11ea-806f-a10139644c2a.png)   

직접 -e 옵션으로 인풋 아웃풋 설정 가능!
  
![image](https://user-images.githubusercontent.com/56130599/89098932-a0cf3000-d426-11ea-8b90-4449e8ba6bb2.png)   

logstash로 원하는 데이터를 보내고 출력하는 방식으로 사용

123123123 넣은 데이터가 메시지로 그대로 출력된다.

![image](https://user-images.githubusercontent.com/56130599/89098942-b5132d00-d426-11ea-8720-07a2eabc1ae3.png)  

enabled 이 false면 동작을 안함 true로 바꿔줘야됨 

path에 설정되어있는 로그들을 가져옴

filebeat의 인풋 설정 type두번째를 넣고싶으면 -로 시작되는 type을 넣어주면 됨.  

![image](https://user-images.githubusercontent.com/56130599/89098959-d6741900-d426-11ea-9e30-6bc232299a97.png)   

#로 주석 처리되어있던 output.logstash를 주석해제하고 활성화 시킨다.

![image](https://user-images.githubusercontent.com/56130599/89098974-04595d80-d427-11ea-9571-02d1d512ca10.png)    

키바나에 설정된 주소값 - 만약 원격이면 localhost말고 원격 주소를 적어야된다. 

키바나는 filebeat에 내장되어 대쉬보드를 꾸미고 싶은 경우 localhost의 정보를 통해 가능하다.

- conf.d에 설정파일들을 옮겨주고 모든 설정파일을 지정함. 따라서 주어진 예시의 설정파일을 옮겨주는 과정
  
![image](https://user-images.githubusercontent.com/56130599/89098984-22bf5900-d427-11ea-8034-38fe1e615707.png)  

pipelines/yml 파일을 cat을 통해 볼 수 있다. conf.d를 참조하도록 되어있다. conf.d 파일에는 지금 아무것도 없다.      
설정파일이 여러 개 있을 수도 있어서 여러 개를 수행하기 위해서 다수의 conf 파일을 가져올 수 있도록 해놓았다.    
이런 것도 나중에 다 버전관리 하게 되어있다.    
sample 코드를 하나 복사해서 conf.d 파일(first-pipeline.conf)에 넣는다.   

![image](https://user-images.githubusercontent.com/56130599/89099018-57331500-d427-11ea-8fb6-ccc948c34b74.png) 

first-pipeline.conf 파일을 열면 json형태로 구현된 내용들을 볼 수 있다.   
output의 이름으로 세팅되도록 index가 조정되어있다.    

![image](https://user-images.githubusercontent.com/56130599/89099026-69ad4e80-d427-11ea-81be-299c42c3412b.png)   

![image](https://user-images.githubusercontent.com/56130599/89099032-7336b680-d427-11ea-9792-2f95a009c238.png) 
  
![image](https://user-images.githubusercontent.com/56130599/89099042-80ec3c00-d427-11ea-8505-2c946b727316.png)

![image](https://user-images.githubusercontent.com/56130599/89099047-8b0e3a80-d427-11ea-8f66-78f1e56569ba.png)

filebeat를 통해서 log들이 올라오는 것을 확인할 수 있다.
