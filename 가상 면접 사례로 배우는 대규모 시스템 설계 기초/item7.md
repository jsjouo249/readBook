# item7. 분산 시스템을 위한 유일 ID 생성기 설계

auto_increment 속성이 설정된 관계형 db 사용하면 안되나??  
분산 환경은 안되고, 여러 db를 쓰는 경우엔 지연 시간을 낮추기 힘듬

## 1. 문제 이해 및 설계 범위 확정
예시)
- ID는 유일해야 한다
- ID는 숫자로만 구성되어야 한다.
- ID는 64비트로 표현될 수 있는 값이어야 한다.
- ID는 발급 날짜에 따라 정렬 가능해야 한다.
- 초당 10,000개의 ID를 만들 수 있어야 한다.

## 2. 개략적 설계안 제시 및 동의 구하기
유일성이 보장되는 ID를 만드는 방법
- 다중 마스터 복제(multi-master replication)
- UUID(Universally Unique Identifier)
- 티켓 서버(ticket server)
- 트위터 스노플레이크(twitter snowflake) 접근법

### 다중 마스터 복제
데이터베이스의 auto_increment 기능을 활용하는 것.
- 다만 1만큼 증가가 아닌, 사용 중 인 데이터베이스 의 서버 수인 k만큼 증가 시킨다.

단점
- 여러 데이터 센터에 걸쳐 규모를 늘리기 어렵다.
- ID의 유일성은 보장되겠지만, 그 값이 시간 흐름에 맞추어 커지도록 보장할 수 없다.
- 서버를 추가하거나 삭제할 때도 잘 동작하도록 만들기 어렵다

### UUID
컴퓨터 시스템에 저장되는 정보를 유일하게 식별하기 위한 128비트짜리 수

장점
- 만드는 것은 단순하다
- 각 서버가 만들기 때문에, 규모 확장도 쉬움

단점
- 128비트로 길다
- 시간순으로 정렬이 불가능 하다
- 숫자가 아닌 값이 포함될 수 있다

### 티켓 서버
ID를 발급하는 서버를 별도로 두는 방법
핵심은 auto_increment 기능을 갖춘 db서버를 중앙 집중형으로 사용

장점
- 유일성이 보장되는 숫자로만 구성된 id 쉽게 생성 가능
- 구현하기 쉽고, 중소 규모 애플리케이션에 적합

단점
- 티켓 서버가 SPOF가 됨
  - SPOF를 피하기 위해, 다중 티켓 서버를 두면, 데이터 동기화 문제가 생김

### 트위터 스노플레이크
독창적인 ID 생성 기법
- 각개 격파 전략
64비트 ID 구조
- 사인 비트 : 1비트, 양수와 음수를 구분
- 타임스탬프 : 41비트, 41비트로 표현할 수 있는 시간의 범위는 69년
- 데이터센터 ID : 5비트, 32개의 데이터 센터를 구분
- 서버 ID : 5비트, 하나의 데이터 센터 당 32개의 서버를 구분
- 일렬번호 : 12비트, 1밀리초가 경과할 때 마다 0으로 초기화하며, 각 서버는 id를 생성할 때 마다 1 증가


## 3. 상세 설계
64비트 ID 구조에서
데이터센터 ID 와 서버 ID는 시스템이 시작할 때 결정되며, 시스템 운영 중 바뀌지 않음.

### 타임스탬프
시간의 흐름에 따라 점점 큰 값을 갖게 되며, 시간 순정렬 가능  
다만 41비트는 69년이기 때문에, 69년이 지나면 기원 시각을 바꾸거나, ID 체계를 다른 것으로 이전

## 4. 마무리
- 시계 동기화: 생성 서버들이 전부 같은 시계를 사용한다고 가정했지만, 여러 코어에서 실행될 경우 유효하지 않음.
  - NTP(Network Time Protocol)를 사용하여 시계 동기화
- 각 64비트 ID의 구조 길이 최적화
- 고가용성