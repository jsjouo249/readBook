# item9. 타입 단언보다는 타입 선언을 사용하기
## 요약
### 타입 단언(as Type)보다는 타입 선언(: Type)을 사용해야 한다.
### 화살표 함수의 반환 타입을 명시하는 방법을 터득해야 한다.
### ts의 타입 추론보다 타입 정보를 더 잘 알고 있는 상황에서는 타입 단언문과 null 아님 단언문(!)을 사용한다.
타입 단언은 타엡 체커에게 오류를 무시하라는 뜻!
```
화살표 함수의 반환 타입
const people: Person[] = ['alice', 'bob', 'jan'].map(
  (name): Person => ({name})
);
```


# item10. 객체 래퍼 타입 피하기
## 요약
### 기본형 값에 메소드를 제공하기 위해서는 객체 래퍼 타입이 어떻게 쓰이는지, 이해해야 하고, 직접 사용하거나 인스턴스 생성은 피해야 한다.
### 객체 래퍼 타입은 지양하고, 기본형 타입을 사용해야 한다.
기본형 값들에 대한 일곱 가지 타입(string, number, boolean, null, undefined, symbol, bigint)이 있으며, 불변이며 메소드를 가지지 않음.
객체 레퍼 타입 => string - String, number - Number, boolean - Boolean, symbol - Symbol, bigint - BigInt


# item11. 잉여 속성 체크의 한계 인지하기
## 요약
### 객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때, 잉여 속성 체크가 수행된다.
### 잉여 속성 체크는 오류를 찾는 효과적인 방법이지만, ts 타입 체커가 수행하는 일반적인 구조적 할당 가능성 체크와 역할이 다르다.
### 할당의 개념을 정확히 알아야 잉여 속성 체크와 일반적인 구조적 할당 가능성 체크를 구분할 수 있다. 
### 잉여 속성 체크는 한계가 있는데, 임시 변수를 도입하면 잉여 속성 체크를 건너뛸 수 있다.
잉여 속성 체크가 할당 가능 검사와는 별도의 과정
```
임시 변수 도입 예시
const o: Options = { darkmode: true, title: 'Ski Free' }; -> 'Options' 형식에 darkmode가 없다.

const intermediate = { darkmode: true, title: 'Ski Free' };
const o: Options = intermediate; -> 정상
```

