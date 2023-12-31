# 이벤트 기반 아키텍처 스타일

## 서론

- 확장성이 뛰어난 고성능 애플리케이션 개발에 널리 쓰이는 **비동기 분산 아키텍처 스타일**
- 다른 아키텍처 스타일에 내장될 수도 있다.

![image](https://github.com/leehjhjhj/reading-books/assets/102458609/366f515b-dc6b-41f7-9f9e-64343883aeec)

- 대부분 **요청 기반 모델**을 따른다.
    - 이 모델은 어떤 액션을 수행하도록 시스템에 요청하면 요청 오케스트레이터가 접수한다.
    - 요청 오케스트레이터는 보통 유저 인터페이스지만, API 레이어나 엔트프라이즈 서비스로도 구현 가능
    - 다양한 요청 프로세서에 확장적으로, 동기적으로 요청을 전달
    - 요청을 받아 DB 조회 등의 작업을 수행

## 토폴로지

- 중재자 토폴로지: 이벤트 처리 워크플로를 제어할 경우
- 브로커 토폴로지: 신속한 응답과 동적인 이벤트 처리 제어가 필요할 때

## 브로커 토폴로지
![image](https://github.com/leehjhjhj/reading-books/assets/102458609/60b16ace-64c5-49b6-a211-229d7ced22d1)

- 중앙에 이벤트 중재자가 없다는 점이 중재자와 다르다
- 메시지는 경량 메시지 브로커를 통해 브로드 캐스팅, 이벤트 프로세서 컴포넌트에 분산되어 흘러감
- 이벤트 처리 흐림이 단순하고, 중앙에서 제어할 필요가 없을 때 유용
- 시작 이벤트, 이벤트 브로커, 이벤트 프로세서, 처리 이벤트로 구성
- 시작 이벤트는 전체 이벤트 흐름을 개시하는 이벤트, 이벤트 브로커의 이벤트 채널로 전송되어 처리
- 단일 이벤트 프로세서는 이벤트 브로커에서 시작 이벤트를 받자마자 관련된 처리 작업을 마치고, 처리 이벤트를 생성
- 그 후에 시스템의 나머지 부분에 자신이 할 일을 비동기로 알림
- 처리 이벤트는 필요시 부가적인 처리를 위해 이벤트 브로커에 비동기 전송되고, 다른 이벤트 프로세서는 처리 이벤트를 리스닝하다가 이벤트가 들어오면 처리 한 뒤 새로운 처리 이벤트를 발행하여 자신이 한 일을 **모두에게** 알림
- 토픽은 일반적으로 발행-구독 메시징 모델을 사용
- 다른 이벤트 프로세서의 관심 여부와 무관하게 각 이벤트 프로세서가 한 일을 모두에게 알리는게 바람직 → 확장성을 위해
- 전체 워크 플로를 제어할 수 없다는 큰 단점이 존재
    - 트랜잭션이 언제 끝나는지 모르고 에러 처리도 어렵다.

## 중재자 토폴로지
![image](https://github.com/leehjhjhj/reading-books/assets/102458609/c5329c99-d734-44b7-84c5-c28712c46b72)

- 브로커 토폴로지의 단점을 일부 보완
- 이벤트 중재자가 핵심
- 시작이벤트, 이벤트 큐, 이벤트 중재자, 이벤트 채널, 이벤트 프로세서 컴포넌트로 구성
- 이벤트 중재자는 이벤트 처리에 관한 단걔 정보만 갖고 있어 점대점 메시징으로 각각의 이벤트 채널로 전달되는 처리 이벤트 생성
- 그러면 이벤트 프로세서는 처리 한 뒤에 작업을 완료했다고 중재자에게 응답
- 특정 도메인, 이벤트 그룹과 관련된 중재자가 여럿 존재하기 때문에 단일 장애점을 줄이고 전체 처리량과 성능을 높임
    - 전체 고객은 고객 중재자가, 주문 관련은 주문 중재자가 처리
- 중재자는 전체 워크플로를 잘 알고 통제 가능, 에러처리, 복구, 재시작 가능
- 트레이드 오프도 존재
    - 복잡한 이벤트 흐름 내에서 발생하는 동적인 처리를 선언적으로 모델링하기가 매우 어렵다
    - 쉽게 확장할 수 있지만, 중재자도 확장해야하기 때문에 전체 이벤트 처리 흐름에 병목 지점이 생기기 쉬움
    - 이벤트 처리를 중재자가 제어하기 때문에 이벤트 프로세서가 커플링, 성능은 브로커보다 좋지 않다.

## 비동기 통신
![image](https://github.com/leehjhjhj/reading-books/assets/102458609/a12651af-7277-449a-ac4c-941676423aea)

- 요청, 응답 처리 뿐만 아니라 파이어 엔드 포켓 처리도 모두 비동기 통신만 사용한다.
- **유저가 굳이 어떤 정보를 돌려받을 필요가 없으면 기다리게 할 이유 또한 없다.**

```sql
응답성과 성능의 차이점
- 응답성은 어떤 액션이 접수되어 곧 처리될 거라는 사실을 유저에게 알리는 것
- 성능은 종단간 프로세스가 더 빨리 수행되게끔 만드는 것
```

- **비동기 통신에서는 에러 처리가 가장 큰 문제이다.** 응답성은 엄청나게 개선되지만 에러를 제대로 처리하기가 쉽지 않기 때문에 이벤트 기반 시스템의 복잡도가 가중된다. (댓글 욕설 예시)

## 에러 처리
![image](https://github.com/leehjhjhj/reading-books/assets/102458609/e816af76-f1de-41ea-bd5c-3c59b581458c)

- 리액티브 아키텍처의 **워크플로 이벤트 패턴** = 비동기 워크플로의 에러처리 방법중 하나
- 탄력성과 응답을 둘다 잡음
- 대리자를 통해 위임, 봉쇄, 수리작업을 한다
- 과정
    - 이벤트 프로듀서는 메시지 채널을 통해 데이터를 이벤트 컨슈머에게 비동기 전송
    - 컨슈머가 데이터 처리 도중 에러가 발생하면 해당 에러를 워크플로 프로세서에게 위임
    - 그리고 이벤트 큐에 있는 다음 메시지로 넘어감 → 응답성에 영향을 안 받음
    - 워크플로 프로세서는 해당 에러를 정정하고 다시 큐로 돌려보낸다.

![image](https://github.com/leehjhjhj/reading-books/assets/102458609/29b0648b-7458-4cc9-af74-bbbb7d5a4cbd)

- 이 패턴의 주의점은 에러가 발생한 메시지의 처리 순서가 바뀐다는 것

## 데이터 소실 방지
![image](https://github.com/leehjhjhj/reading-books/assets/102458609/eb0ddbed-e61b-4616-806c-926c872ec1a8)


- 이벤트 기반 아키텍처는 데이터가 소실 될 만한 곳이 많다.
- 소실이 일어나는 경우
    1. 이벤트 프로세서 A에서 메시지 큐로 전달이 안 되는 경우
    2. 이벤트 프로세서 B가 큐에서 메시지를 꺼내, 처리하기 전에 장애
    3. 데이터 에러로 인해 이벤트 프로세서 B가 DB에 메시지를 저장 할 수 없음
- 해결법
    1. 동기 전송과 퍼시스턴스 메시지 큐를 이용
        1. 퍼시스턴스 메시지 큐는 전달 보장 지원. 메시지 브로커가 메시지를 수신하면 메모리에 저장하는 동시에 물리적 DB에도 메세지를 저장
    2. 클라이어니트 확인 응답 모드를 사용
        1. 메시지를 큐에 보관 한 채 다른 컨슈머가 메시지를 읽을 수 없게 클라이언트 ID를 부착.
        2. 이벤트 프로세서 B가 잘못돼도 메시지는 큐에 계속 남아 있다.
    3. ACID 트랜잭션의 커밋으로 해결
        1. 저장이 되면 큐에서 메시지가 삭제되도록

## 브로드 캐스팅
![image](https://github.com/leehjhjhj/reading-books/assets/102458609/f01e13aa-5494-4b5f-859b-72ad57f73bb2)

- 이벤트 기반 아키텍처는 메시지를 받은 컨슈머가 있다면 그 메시지로 무엇을 하든 상관 없이 이벤트를 브로드캐스트 할 수 있다.
- 누가 받는지 상관 안하기 때문에 높은 수준의 디커플링, 최종 일관성, 복잡한 이벤트 처리 가능한 필수 기능

## 요청 응답
![image](https://github.com/leehjhjhj/reading-books/assets/102458609/8c950099-f61c-46ce-b78b-c03b9120c000)

- 동기 통신이 필요할 경우도 있다.
- 이벤트 기반 아키텍처는 동기 통신을 **요청-응답 메시징** 방식으로 수행
    - 메시징 내부의 이벤트 채널은 요청 큐, 응답 큐로 구성
    - 처음 정보를 요청하면 요청 큐에 비동기 전송 후, 메시지 프로듀서에 제어권이 반환
    - 메시지 프로듀서는 응답 큐에 응답이 도착하길 바라며 차단 대기 상태
    - 메시지 컨슈머가 메시지를 받아 처리한 후에 응답 큐에 응답을 보내면 이벤트 프로듀서는 메시지를 수신
- 구현하는 기술
    - 메시지에 헤더에 상관 ID 사용
    - 응답 큐에 임시 큐를 두는 방법: 임시 큐는 지정된 요청에만 사용, 요청이 들어오면 생성되고 종료되면 삭제.
    - 후자는 대용량 메시지 처리시 속도 저하. 그래서 상관 ID를 방법을 권장

## 요청 기반이냐, 이벤트 기반이냐
![image](https://github.com/leehjhjhj/reading-books/assets/102458609/7cecc172-88cd-4774-a03a-0c2ac6f08dd9)

## 하이브리드 이벤트 기반 아키텍처

- 이벤트 기반 아키텍처와 다른 아키텍처 스타일을 함께 사용
- 마이크로서비스, 공간 기반 아키텍처가 대표적
- 이벤트 기반 아키텍처를 추가하면 병목 지점을 없애고 유저 응답성을 보장
- 마이크로, 공간 기반은 데이터 펌프에 메시징 활용, 다른 프로세서에 데이터를 비동기 전송하여 DB 업데이트
- 또, 서비스 간의 통신도 이벤트 기반 아키텍처를 활용
