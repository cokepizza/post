# About Prism

프리즘이 무엇인지, 어떻게 동작하는지, 실제 사례를 살펴봅니다.

----

프리즘이란?
- OpenAPI v2/v3를 기반으로 실제 API 서버처럼 동작하여 test가 가능한 응답을 던져주는 Mock Server를 생성
- Prism은 Mock을 생성하기 때문에 웹이나 앱에서 API 별로 유닛 테스트가 가능함
- jest등과 같이 programatic 한 unit test tool이 아닌 endpoint를 가지고 있는 서버로서 동작
    - 서버에서 mock을 생성하는 장점
    