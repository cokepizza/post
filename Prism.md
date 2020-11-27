# About Prism

프리즘이 무엇인지, 어떻게 동작하는지, 실제 사례를 살펴봅니다.

----

프리즘이란?

- OpenAPI v2/v3를 기반으로 실제 API 서버처럼 동작하며, 테스트가 가능한 응답을 던져주는 Mock Server를 생성

프리즘의 장점?

- Prism을 통한 병렬 개발
  - FE, BE가 동시에 작업해야 하는 경우 Open API Generator(ex. Swagger)로 중간에 인터페이스를 맞춰두고, FE 쪽에서는 Prism을 통해 API 관련 Flow 구현 작업이 가능
- Prism은 Mock을 생성하기 때문에 웹이나 앱에서 API 별로 유닛 테스트가 가능함
  - Jest가 programatic 한 유닛 테스트 툴이라면, Prism은 API 레벨에서 endpoint를 가지고 있는 서버로서 유닛 테스트가 가능
- 서버에서 mock을 생성하는 장점

프리즘의 실제 사례

- 프리즘 설치는 다양한 옵션이 있음
  - npm or yarn을 global 설치
  - curl 을 통한 바이너리 다운 및 설치
  - Docker Image를 이용한 설치

- package.json script를 통한 webpack-dev-server, prism server 실행
![package.json script](https://user-images.githubusercontent.com/56418546/100434051-46e22c80-30df-11eb-8adc-7f8a5ad5cb0f.png)

- curl 을 통한 바이너리 다운 및 설치
![prism 실행 쉘 스크립트](https://user-images.githubusercontent.com/56418546/100434061-49448680-30df-11eb-844a-8ec0a2b80270.png)

- webpack-dev-server를 프록시 서버로 하여 prism api server로부터 요청을 받아오는 방식
  - origin server와 api server가 동일 출처에 있기 때문에 api server에서 별도 cors 설정이 필요 없음

- webpack proxy 설정, mock의 경우에는 로컬의 4010 prism 서버에게 요청. 서버간에는 cors 제약 없음
![webpack proxy 설정](https://user-images.githubusercontent.com/56418546/100433479-7f353b00-30de-11eb-950b-dc730b1b06a1.png)