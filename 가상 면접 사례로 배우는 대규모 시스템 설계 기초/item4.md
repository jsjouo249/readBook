# item4. 처리율 제한 장치의 설계
api 처리율 제한 장치
- DoS(Denial of Service) 공격에 의한 자원고갈을 방지
- 비용 절감
- 서버 과부하 방지

## 1단계 문제 이해 및 설계 범위 확정
여러 가지 알고리즘을 사용할 수 있으니, 어떤 제한 장치를 구현해야 하는지 분명히 해야 함
- 클라이언트 측 제한 or 서버 측 제한
- 어떤 기준으로 api 제어
- 시스템 규모
- 분산 환경
- 독립된 서비스 or 애플리케이션 코드
- 사용자에게 알림 기능

## 2단계 개략적 설계안 제시 및 동의 구하기
- 일반적으로 클라이언트는 처리율 제한을 안정적으로 걸 수 있는 장소가 아님
  - 서버 측에 구현
    - api 서버에 구현
    - api 서버 앞 미들웨어를 만들어 구현
  - 보통 api 게이트웨이라 불리는 컴포넌트에 구현
  - api 게이트웨이는 처리율 제한, SSL 종단, 사용자 인증, IP 허용 목록 관리
  - 고려할 점
    1. 기술 스택을 점검하여, 서버 측 구현을 지원하기 충분한 정도로 효율이 높은지 확인
    2. 사업에 필요한 처리율 제한 알고리즘을 찾되, 서드파티를 사용하면, 선택지는 제한됨
    3. 마이크로서비스에 기반하고 있고, api 게이트웨이를 이미 설계에 포함 했다면, 처리율 제한 기능 또한 게이트웨이에 포함 될 수 있음
    4. 처리율 제한 장치를 구현하기 어려우면, 상용 api 게이트 웨이도 방법
- 처리율 제한 알고리즘
  - 토큰 버킷 알고리즘
    - 지정된 용량을 갖는 컨테이너로, 사전 설정된 양의 토큰이 주기적으로 채워지며, 각 요청은 처리될 때, 하나의 토큰을 사용
    - 파라미터
      - 버킷 크기: 버킷에 담을 수 있는 토큰의 최대 개수
      - 토큰 공급률: 초당 몇개의 토큰이 버킷에 공급되는가
  - 누출 버킷 알고리즘
    - 토큰 버킷과 비슷하지만, 큐로 구현되며, 요청 처리율이 고정되어 있다
    - 파라미터
      - 버킷 크기: 큐 사이즈
      - 처리율: 지정된 시간당 몇 개의 항목을 처리할지
  - 고정 윈도 카운터 알고리즘
    - 타임라인을 고정된 간격의 윈도로 나누고, 각 윈도마다 카운터를 붙여서, 카운터의 값이 사전에 설정된 임계치에 도달하면 새로운 요청은 새로운 윈도가 열릴 때 까지 버려짐
    - 문제점: 윈도의 경계 부근에 순간적으로 많은 트래픽이 몰릴경우, 윈도에 할당된 양보다 더 많은 요청이 처리될 수 있음
  - 이동 윈도 카운터 알고리즘
    - 고정 윈도 카운터 알고리즘과 이동 윈도 로깅 알고리즘 결합
- 카운터는 보통 메모리상에서 동작하는 캐캐시가 바람직하며, 레디스에 자주 사용

## 3단계 상세 설계
- 처리율 제한 규칙
- 처리가 제한된 요청들 처리 방법
### 처리율 한도 초과 트래픽의 처리
429응답을 클라이언트에게 보내며, 한도 제한에 걸린 메시지를 나중에 처리하기 위해 큐에 보관하기도 함  
http 응답 헤더에 값을 추가  
예시)
 - X-Ratelimit-Remaining: 윈도 내에 남은 처리 가능 요청의 수
 - X-Ratelimit-Limit: 매 윈도마다 클라이언트가 전송할 수 있는 요청의 수
 - X-Ratelimit-Retry-After: 클라이언트가 다음 요청을 보낼 수 있는 시간

## 분산 환경에서의 처리율 제한 장치의 구현
- 경쟁 조건
- 동기화
문제를 해결해야 한다.

### 경쟁 조건
널리 알려진 해결책은 락 이지만, 시스템의 성능을 상당히 떨어뜨린다는 문제.
1. 루아 스크립트 - https://stripe.com/blog/rate-limiters
2. 레디스 자료구조인 정렬 집합 - https://engineering.classdojo.com/blog/2015/02/06/rolling-rate-limiter/

### 동기화 이슈
- 고정 세션을 활용할 수도 있지만, 확장에 용이하지 않고, 유연하지 않음
- 레디스와 같은 중앙 집중형 데이터 저장소 사용

## 성능 최적화
- 데이터센터가 멀면 지연시간이 증가하니, 에지 서버를 생성
- 일관성 모델을 사용

## 모니터링 방법
- 채택된 처리율 제한 알고리즘이 효과적이다
- 정의한 처리율 제한 규칙이 효과적이다
를 확인해야 함


### 4. 마무리
- 알고리즘
  - 토큰 버킷
  - 누출 버킷
  - 고정 윈도 카운터
  - 이동 윈도 로그
  - 이동 윈도 카운터
- 아키텍처
- 분산환경에서의 처리율 제한 장치
- 성능 최적화와 모니터링 등
- 경성 또는 연성 처리율 제한
  - 경성: 요청 개수는 임계치를 절대 넘어설 수 없음
  - 연성: 요청 개수는 잠시동안은 임계치 넘어설 수 있음
- 다양한 계청에서의 처리율 제한
  - 애플리케이션 계층도 되지만,
  - IP주소인 3계층에 처리율 제한 적용도 가능

