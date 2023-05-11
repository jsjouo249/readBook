# item4. 구조적 타이핑에 익숙해지기
## 요약
### js는 덕타이핑 기반이고, ts는 이를 모델링하기 위해 구조적 타이핑을 사용해야 한다. 어떤 인터페이스에 할당 가능한 값이라면, 타입 선언에 명시적으로 나열된 속성들을 가지고 있고, 타입은 봉인(선언된 속성 이외의 속성 추가 불가)되어 있지 않다
### 클래스 역시 구조적 타이핑 규칙을 따르기 때문에, 클래스의 인스턴스가 예상과 다를 수 있다.
### 구조적 타이핑을 사용하면 유닛 테스트를 손쉽게 할 수 있다.
덕타이핑은 객체가 어떤 타입에 부합하는 변수와 메소드를 가지면, 객체를 해당 타입에 속하는 것으로 간주하는 방식.
ts는 구조적 타이핑을 사용하는데, 매개 변수와 부합하는 객체를 잘못 전달하면,타입 체커도 오률 발견하지 못하는 경우가 발생할 수 있음. -> 따로 설정이 필요함.(nominal typing)
클래스 생성 시, 
```
class C {
  foo: string;
  constructor(foo: string) {
    this.foo = foo;
  }
}

const c = new C('instanf of C');
const d: C = { foo: 'object literal' };
```
구조적으로 필요한 속성(string타입의 foo)과 생성자가 존재하기 때문에, d도 정상
유닛 테스트 시,
```
interface DB {
  runQuery: (sql: string) => any[];
}
function getAuthors(database DB): Author[] {
  const authorRows = database.runQuery('...');
  return authorRows.Map(...);
}
```
구조적 타이핑 덕분에 db 인터페이스를 구현하는지 선언이 필요 없음. 추상화를 함으로써 로직과 테스트를 분리


# item5. any 타입 지향하기
## 요약
### any타입은 타입 안정성이 없다.
### any타입은 함수 시그니처(약속된 타입의 출력 반환)을 무시한다.
### any타입은 언어 서비스가 적용되지 않는다.
### any타입은 코드 리팩토링때 버그를 감춘다.
### any타입은 타입 설계를 감춘다.
### any타입은 타입시스템의 신뢰도를 떨어트린다.
