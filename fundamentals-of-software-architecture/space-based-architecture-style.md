# 공간 기반 아키텍처 스타일

## 서론

- 기존의 웹 기반 비지니스 애플리케이션은 브라우저의 요청 → 웹 서버 → 애플리케이션 서버 → 데이터베애스 서버로 도달하는게 일반적
- 하지만 이러한 패턴은 유저 수가 많아지면 병목 현상이 발생한다.
- 해결책으로는 웹 서버의 확장, 그러나 이는 다음 레이어의 병목점을 이동 시킬 뿐 근본적인 해결책은 아니다.
- 따라서 이러한 문제를 해결할 수 있는 아키텍처는 `공간 기반 아키텍처 스타일` 로, 높은 확장성과 탄력성, 동시성을 보장한다.

## 토폴로지

![image](https://github.com/leehjhjhj/reading-books/assets/102458609/82623407-c0e6-47d8-a720-183ef8b445a8)


- 이 아키텍처는 튜플 공간에서 유래되었고, 튜플 공간은 공유 메모리를 통해 통신하는 **다중 병렬 프로세서**
- 동기 제약 조건인 중앙 데이터베이스를 없애는 대신 `인메모리 데이터 그리드`를 활용
- 구성요소
    - 처리장치: 구현된 코드들
    - 가상 미들웨어: 처리 장치를 관리, 조정
    - 데이터 펌프: 업데이트된 데이터를 db에 비동기 전송
    - 데이터 라이터: 펌프에서 데이터를 받아 업데이트 수행
    - 데이터 리더: db의 데이터를 읽어 전달

### 처리장치

- 애플리케이션 로직을 갖고 있다.
- 단일 처리 장치도 가능하나, 대규모 애플리케이션은 기능별로 처리장치를 나눈다.

### 가상 미들웨어
![image](https://github.com/leehjhjhj/reading-books/assets/102458609/84efeb74-f511-4650-8084-27d07dcd38a9)
- 아키텍처 내부에서 데이터 동기화 및 요청 처리의 다양한 부분을 제어하는 인프라를 담당한다.
- 메시징 그리드, 데이터 그리드, 처리 그리드, 배포 관리자로 구성
    - 메시징 그리드: 입력 요청과 세션 상태를 관리. 부하 분산 웹서버로 구현(엔진엑스)
    - 데이터 그리드: 처리 장치 안의 복제 캐시로서 구현. 하지만 분산 캐시로 쓰일 때는 가상 미들웨어도 있다. 모든 데이터 동기화는 **비동기로 구현**
    - 처리 그리드: 다수의 처리장치가 단일 비지니스 요청을 처리할 때 요청 처리를 오케스트레이트
    - 배포 관리자: 부하 조건에 따라 처리 인스턴스를 동적으로 시작, 종료

### 데이터 펌프

![image](https://github.com/leehjhjhj/reading-books/assets/102458609/267b4e1d-2bbb-45f6-a285-bd3f4c05825a)


- 데이터를 다른 프로세서에 보내 데이터베이스를 업데이트
- 항상 비동기로 작동, 메모리 캐시와 데이터베이스의 최종 일관성 유지
- 대게 메시징 기법으로 구현.(FIFO, 비동기)

### 데이터 라이터

- 데이터 펌프에서 메시지를 받아 그에 맞게 db를 업데이트
- 하나의 데이터 라이터를 두는 방법과 각 처리 장치마다 데이터 라이터를 두는 방법이 있다.

### 데이터 리더

- 데이터 베이스에서 데이터를 읽어 리버스 데이터 펌프를 통해 처리장치로 나른다.
- 작동하는 경우
    1. 동일한 이름의 캐시를 가진 모든 처리 장치 인스턴스가 실패하는 경우.
    2. 동일한 이름의 캐시 안에서 모든 처리 장치를 재배포하는 경우.
    3. 복제 캐시에 들어있지 않은 아카이브 데이터를 조회하는 경우.
- 데이터 라이터와 데이터 리더는 본질적으로 추상 **레이어를 형성**
- 처리 장치마다 복제 캐시 스키나는 하부 데이터베이스와 구조가 다를 수 있다.

## 데이터 충돌

- **데이터 충돌은 한 캐시(캐시 A) 인스턴스에서 데이터가 업데이트되어 다른 캐시 인스턴스에 복제하는 도중에 동일한 데이터가 해당 캐시(캐시 B)에서 업데이트 되는 현상을 말한다.**

## 클라우드 대 온프레미스 구현

![image](https://github.com/leehjhjhj/reading-books/assets/102458609/36b8e83a-af63-4da7-9c64-e69c7bc23789)


- 물리 데이터베이스는 온프레미스 환경에, 처리장치, 가상 미들웨어를 클라우드에 배포할 수 있다.

## 복제 캐시 대 분산 캐시

![image](https://github.com/leehjhjhj/reading-books/assets/102458609/fe72348b-a880-4bd5-9690-a5576acd4744)


- 복제 캐시는 공간 기반 아키텍처의 표본
- 그러나 데이터량이 많거나 캐시 데이터가 빈번하게 업데이트 되는 경우 사용 불가능 → 데이터 일관성 문제

![image](https://github.com/leehjhjhj/reading-books/assets/102458609/dcec400a-d3d5-433d-a015-a72188a01daf)


- 그런 경우 분산 캐시를 사용.
- 처리장치에 내부 메모리에 저장하는게 아니고 동기 전용 프로토콜로 중앙 캐시 서버에 데이터를 액세스.
- 높은 일관성을 제공하나 성능이 낮고 시스템 전체 레이턴시 증가
- 각 트레이드 오프가 있기 때문에 둘다 섞어서 사용. 예를 들어서 재고 관리는 분산캐시, 고객 프로필 유지는 복제 캐시

## 니어캐시

![image](https://github.com/leehjhjhj/reading-books/assets/102458609/030bbf98-e532-464b-a652-67841e0987af)


- 풀 백킹 캐시, 프런트 캐시가 있다.
- 프런트 캐시는 방출 정책을 따라 옛 항목을 삭제
- 그러나 프런트 캐시끼리는 통신을 하지 않고 풀 백킹 캐시와 동기화함.
- 때문에 일관성 문제가 생겨서 이 아키텍처에서는 권장하지 않는다.

## 구현 예시

- 콘서트 티켓 판매 시스템
- 온라인 경매 시스템
